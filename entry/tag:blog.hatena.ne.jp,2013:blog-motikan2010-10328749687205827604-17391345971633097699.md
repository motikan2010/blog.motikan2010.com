[f:id:motikan2010:20180407203434p:plain]  
　まず、MySQLからRedshiftへのコピーなんて「Amazon Data Pipeline」を利用しろよと思われますが、そんなお金はない（※調べてみるとクソ安かった）。

　なので、Embulkを利用して MySQLに格納されているデータをRedshiftへコピーしてみました。  
そしてタイトル通り『<span style="color: #ff0000">文字行文字（\r\n）・タブ文字(\t)が消える</span>』のような現象が起きました・・・。

##### 各バージョン
- embulk (0.9.4 java)
- embulk-output-redshift (0.8.0)

まずはその事象を実際に確認してみます。



<!-- more -->



## 事象の確認

### 確認用テーブル
　単純な３つのカラムを持つ構造です。  
"description"列に改行を含む値を入れていきます。
```sql
create table product(
    id int primary key auto_increment,
    name varchar(255),
    description varchar(255)
);

insert into product(name, description) values 
('商品１', 'これは商品１です。\r\nこれは商品１です。');
```

まずはMySQLで改行が格納されていることを確認。  
　想定通り"description"列のレコードに改行が入っています。
```sql
mysql> select * from product\G
*************************** 1. row ***************************
         id: 1
       name: 商品１
description: これは商品１です。
これは商品１です。
```

### EmbulkでRedshiftへコピー
　下記の設定ファイルを利用します。  
ファイル名: mysql_to_redshift.yml
```yaml
in:
  type: mysql
  user: root
  password: XXXXX
  database: sampledb
  host: 127.0.0.1
  query: |
    select * from product

out:
  type: redshift
  host: xxxxx.yyyyy.ap-northeast-1.redshift.amazonaws.com
  user: sample_user
  password: XXXXX
  database: sampledb
  table: product
  aws_access_key_id: XXXXXXXXXX
  aws_secret_access_key: XXXXXXXXXXXXXXX
  iam_user_name: sample_user
  s3_bucket: sample-bucket
  s3_key_prefix: temp/redshift
  delete_s3_temp_file: false
  mode: replace
```
　そしていつも通りに実行。
```
$ embulk run mysql_to_redshift.yml
```

### Redshift側で改行の確認
##### psqlコマンドで確認
```
sampledb=# select * from product;
 id |  name  |              description
----+--------+----------------------------------------
  1 | 商品１ | これは商品１です。rnこれは商品１です。
```
「改行(\r\n)」が「rn」になっている・・・。

##### SQLWorkbenchで確認
　やはり改行を表現できていない。  
[f:id:motikan2010:20180407194119p:plain]

## 原因
　なぜこうなってしまうのかをRedshiftへ挿入されるまでの流れを追って確認してみる。

### Redshiftへの挿入の流れ
1. MySQLのレコードをTSV形式で出力
1. 出力されたTSVファイルをS3へ配置
1. S3に配置されたTSVファイルをRedshiftへCOPY

　簡単な流れだがどこかで意図しない処理が実行されていると思われる。

### S3に配置されたファイル
 EmbulkがS3に配置するTSVファイルを見てみることにする。
##### COPYコマンドによって参照されるファイル
　GZIPで圧縮されているため、内容を確認するには解凍する必要があります。
 ファイルの中身は、各値はタブ文字で区切りであるTSVファイルの形式となっている。
 TSVファイルに置き換えた上でRedshiftにcopyする仕組みとなっているらしい。
```
$ gzip -d b5b538cf-de9e-4bdf-947b-1a97d3975a5a.gz
$ cat b5b538cf-de9e-4bdf-947b-1a97d3975a5a
1	商品１	これは商品１です。\r\nこれは商品１です。
```

##### COPY時に指定されるオプション
下記のオプションが指定されてRedshiftにcopyされます。
```
GZIP DELIMITER '\t' NULL '\\N' ESCAPE TRUNCATECOLUMNS ACCEPTINVCHARS STATUPDATE OFF COMPUPDATE OFF
```
https://github.com/embulk/embulk-output-jdbc/blob/master/embulk-output-redshift/src/main/java/org/embulk/output/redshift/RedshiftCopyBatchInsert.java#L62

　まず、RedshiftにCOPYする際の改行方法は「\r\n」を記述するだけでよいのだろうか？
その答えは下のスライドに書かれていた。
<iframe src="//www.slideshare.net/slideshow/embed_code/key/qBzkNrKsz802vQ?startSlide=35" width="340" height="290" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/mokemokechicken/logs3redshift" title="Logをs3とredshiftに格納する仕組み" target="_blank">Logをs3とredshiftに格納する仕組み</a> </strong> from <strong><a href="//www.slideshare.net/mokemokechicken" target="_blank">Ken Morishita</a></strong> </div>

RedshiftへのCOPYを使っての改行の表現は<span style="color: #ff0000"><b>「\r\n」ではなく「\\\r\n」と記述する</b></span>必要があるらしい。

#### なぜ「\r\n」は挿入後「rn」となっていのか
それははCOPYの際に指定された"<b>ESCAPEオプション</b>"の仕業

[https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/copy-parameters-data-conversion.html#copy-escape:title]

## 解決
　「embulk-output-jdbc」には改行についてのオプションは存在していないので、ソースコードを修正して対応する。
#### 「org.embulk.output.postgresql.AbstractPostgreSQLCopyBatchInsert」クラスを修正する
[https://github.com/embulk/embulk-output-jdbc/blob/master/embulk-output-postgresql/src/main/java/org/embulk/output/postgresql/AbstractPostgreSQLCopyBatchInsert.java#L217:title]

改行文字・タブ文字のエスケープも修正している。
##### 修正前
```java
private void setEscapedString(String v) throws IOException{
    for (char c : v.toCharArray()) {
        String s;
        switch (c) {
        case '\\':
            s = "\\\\";
            break;
        case '\n':
            s = "\\n";
            break;
        case '\t':
            s = "\\t";
            break;
        case '\r':
            s = "\\r";
            break;
        case 0:
            s = "";
            break;
        default:
            s = String.valueOf(c);
        }
        writer.write(s);
    }
}
```

##### 修正後
```java
private void setEscapedString(String v) throws IOException{
    for (char c : v.toCharArray()) {
        String s;
        switch (c) {
        case '\\':
            s = "\\\\";
            break;
        case '\n':
            s = "\\\n";  // 修正 "\r"は削除
            break;
        case '\t':
            s = "\\\t";  // 修正
            break;
        case 0:
            s = "";
            break;
        default:
            s = String.valueOf(c);
        }
        writer.write(s);
    }
}
```
後はビルドをして、プラグインとして読み込ませる。  
　Embulkのプラグイン自作・反映の記事は他にたくさんあるのでここでは特にふれない。

## 改行が正常に挿入されているかを確認
[f:id:motikan2010:20180407202220p:plain]  
改行が正常に反映されている。✌('ω')✌

## まとめ
　RedshiftへCOPY時のエスケープに関するスライドありがたや〜。
「embulk-output-jdbc」のgemにも反映されるといいですね（OSSに詳しい人お願いします。）