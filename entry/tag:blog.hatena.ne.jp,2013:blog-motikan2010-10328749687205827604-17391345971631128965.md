<div style="text-align:center;">[f:id:motikan2010:20180401005113j:plain]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　最近、Embulkを使い始めましたが、想像していたよりも便利でした。

　データ量によってはエラーも発生するみたいでしたので、メモ程度に書き留めておく。

[https://github.com/embulk/embulk:embed:cite]
 
　Redshift上のデータをCSVファイルとして保存したり、その逆で、CSVファイルに記載されてるデータをRedshift上に保存したり(CREATE TABLE文も自動生成)可能。

　その便利さ故に、業務でも利用することになりましたが、**Redshift上のデータ量**が多いことが原因で発生したエラーがありましたので記載してきます。

　下記の設定で作成されたRedshiftインスタンスを使用した場合に発生したエラーの解決策を記述していきます。  

 　結論から書きますと、どちらのエラーも**Multi Node(ノード数を複数)** でインスタンスを作成すれば解決できました。（貧乏人には辛い・・・）
 
## 検証環境

### Redshift

| | |
|-|-|
|クラスタータイプ|Single Node（ノード数：1）|
|ノードタイプ|dc2.large|

### Embulk

- Embulk (0.9.4 java)
- embulk-input-redshift (0.9.1)

[https://github.com/embulk/embulk-input-jdbc/tree/master/embulk-input-redshift:embed:cite]

## 検証

### ケース① FETCHの最大サイズを超過

#### エラー内容

```
org.embulk.exec.PartialExecutionException: java.lang.RuntimeException: org.postgresql.util.PSQLException: ERROR: Fetch size 10000 exceeds the limit of 1000 for a single node configuration. Reduce the client fetch/cache size or upgrade to a multi node installation.
```

[https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/fetch.html:title]

#### 解決策

　embulk-input-redshift にはfetchサイズを設定する**fetch_rowsオプション**が存在しているので、「1000」を指定すればエラーは発生しなくなります。

```yaml=
in:
  type: redshift
  host: XXXXX.YYYYY.ap-northeast-1.redshift.amazonaws.com
  user: root
  password: "XXXXXX"
  database: sampledb
  fetch_rows: 1000
  table: users
  select: "*"
```

### ケース② DECLAREの上限

　こちらは取得対象であるテーブルに格納されているデータ容量が大きい場合に発生します。
（dc2.large - ノード数 1 の場合は8,000MB）

#### エラー内容

```
org.embulk.exec.PartialExecutionException: java.lang.RuntimeException: org.postgresql.util.PSQLException: ERROR: exceeded the maximum size allowed for the total set of cursor data: 8000MB.
```

[https://docs.aws.amazon.com/ja_jp/redshift/latest/dg/declare.html#declare-constraints:title]

#### 解決策

- 今のところノード数を増やす方法しかなさそう・・・。  
上記リンクの情報通り、ノード数を2にしたらエラーは発生しなくなった。