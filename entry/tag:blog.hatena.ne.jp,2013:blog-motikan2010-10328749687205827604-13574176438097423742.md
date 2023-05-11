<div style="text-align:center;">[f:id:motikan2010:20220531225101p:plain]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

### 「Merry Maker」 とは

　「Merry Maker」は、Webサイトに仕込まれた<span style="color: #ff0000">Webスキマー（クレジットカード情報や認証情報を盗む悪意あるスクリプト）を検出するツール</span>です。

　米国の大手百貨店Target社のセキュリティエンジニアによって開発されており、GitHub上で公開されています。  
（WikiによるとTarget社は2013年11月に7,000万人規模の顧客情報の流出を経験しているようです）
[https://tech.target.com/blog/meet-merry-maker:embed:cite]

　ソースコードはこちらです。  
<span><a href="https://github.com/target/mmk-ui-api" target="_blank">target/mmk-ui-api: UI, API, and Scanner (Rules Engine) services for Merry Maker</a></span>


### ざっくりと説明

- 監視対象の<span style="color: #ff0000">WebサイトをクローリングすることでWebスキマーを検出</span>
- クローリングは <span style="color: #ff0000">Puppeteer (Node.js) </span>で記述 （クローリングが柔軟。でも少し知識が必要）
  - ログイン処理を記述することでログイン後画面も監視可能
- 検出条件は  <span style="color: #ff0000">yaraルール</span>で記述
  - yaraルール：https://github.com/target/mmk-ui-api/blob/main/scanner/src/rules/skimmer.yara

## 環境構築

### Merry Maker を起動


　Dockerファイルが提供されており、簡単に試すことができます。  

　執筆時点の最新版( 2022/04/22更新 コミットID `ef57040e9cfabcb1ece2eaa118152ea57c632637` )を利用しています。
<div class="md-code" style="width:100%">
```
$ docker compose -f docker-compose.all.yml up
```
</div>

　Merry Maker の起動後「`http://127.0.0.1:8080/`」にアクセスするとダッシュボードが表示されます。  
初期アクセス時には認証情報を設定する必要があります。  

　ログイン後に監視サイトの追加などの設定ができる画面が表示されます。  
サイドバーから各種設定ができる画面に遷移できるようになっています。

[f:id:motikan2010:20220530224034p:plain:w700]

### 監視対象(EC-CUBE)を構築

　次に本記事で利用する監視対象サイトを構築します。

　EC向けCMSである EC-CUBE を監視対象にします。

　EC-CUBE は以下のリポジトリから取得できます。  バージョンは特に指定ありませんが 4.0.5 を使っています。  
<span><a href="https://github.com/EC-CUBE/ec-cube/tree/4.0.5" target="_blank">EC-CUBE/ec-cube at 4.0.5</a></span>

　こちらもDocker で起動します。
<div class="md-code" style="width:100%">
```
$ docker-compose up
```
</div>

　起動後、DBの接続設定やサイトの情報を入力を終えると、EC-CUBEのトップページが表示されます。


[f:id:motikan2010:20220530225113p:plain:w500]

管理画面から会員登録をします。後々ログイン後画面を監視するのに利用します。

## Webスキマーの検知設定

　ではサイトを監視するための設定作業を行なってきます。

　本記事では初めに「トップページを監視」の設定を行い、その次にログイン処理などの設定が必要な「ログイン後ページ(購入確認) を監視」の設定を行なっていきます。

### 監視の設定入門

　Merry Maker は監視対象サイトに対してクローリングを実施し、HTMLやJavaScriptファイルにWebスキマーが埋め込まれていないかを検査していきます。

　クローリングは Puppeteer (Headless Chrome Node.js API) が動作するようになっており、 Node.js で画面の遷移方法やログイン処理などを記述していきます。  

　Webスキマーはyaraルールに従って検知されるようになっています。  
<figure class="figure-image figure-image-fotolife" title="yaraルール抜粋">[f:id:motikan2010:20220531234106p:plain:w400]<figcaption>yaraルール抜粋</figcaption></figure>


### ケース１：トップページを監視

　手始めにトップページだけを監視する設定をしてみます。

#### "Sources" で遷移内容を設定

　"Sources"画面ではWebサイトのクローリング内容を設定します。  

　Name項目には Source の識別子を適当に記述します。今回は `shop 1` とします。  

　Source項目には Puppeteer の書き方でクローリング内容を記述します。  
今回はトップ画面だけを監視するため、トップ画面にアクセスする一行だけの記述で動作します。  

[f:id:motikan2010:20220530224038p:plain:w700]

#### "Sites"で監視間隔を設定

　"Sites"画面ではクローリング間隔を設定することができます。

　今回は検証なので最短の5分間隔でクローリングを行うように設定します。

　この画面で"Source"(クローリング内容)も設定する必要があるため、先ほど作成した `shop 1` を設定します。  
[f:id:motikan2010:20220530224042p:plain:w700]

#### 監視結果を確認 (検知なし)

　"Alerts"に何も表示されていない。
[f:id:motikan2010:20220530224045p:plain:w700]

#### Webスキマー(マルウェア)を埋め込む

　検知の検証には本物のWebスキマーを利用せずに、yaraルールに引っかかる文字列(検知文字列)をWebサイトに埋め込んで検証をしていきます。  

　`digital_skimmer_slowaes` ルールの検知文字列をWebサイトに埋め込みます。
[f:id:motikan2010:20220530232958p:plain:w500]

　EC-CUBEの管理画面からJavaScriptを編集して、検知文字列が出力されるようにします。
[f:id:motikan2010:20220530224058p:plain:w700]

　編集後、`customize.js`に検知文字列が記述されていることが確認できます。
[f:id:motikan2010:20220530232806p:plain:w700]

#### 監視結果を確認 (検知あり)

　検知文字列の設定後、クローリングが完了するとアラートが表示されます。

　Merry Maker のダッシュボードから確認することができます。

[f:id:motikan2010:20220530224050p:plain:w700]

　"Alerts"画面からWebスキマーが検知されたJavaScriptファイル名を確認することができます。
[f:id:motikan2010:20220530224054p:plain:w700]


### ケース２：ログイン後ページ(購入確認) を監視

　次は「ご注文内容のご確認」画面にWebスキマーを埋め込んで検知されることを確認していきます。  

　この画面は注文完了の直前の画面であり、遷移するにはログインする必要があります。

[f:id:motikan2010:20220531000707p:plain:w700]

　クローリングを簡潔にするため、カートに商品を入れた状態にしておきます。以下の画面が監視対象です。  
[f:id:motikan2010:20220530224101p:plain:w700]

#### Webスキマー(マルウェア)を埋め込む

　`digital_skimmer_gibberish` ルールの検知文字列をWebサイトに埋め込みます。

　以下の条件に該当する文字列をサイトに埋め込みます。
[f:id:motikan2010:20220531001111p:plain:w500]

　「ご注文内容のご確認」のHTMLに検知文字列が表示されていることが確認できました。  
[f:id:motikan2010:20220530224105p:plain:w700]

#### "Sources" で遷移内容を設定

　遷移内容を記述します。

　ログインとリンク遷移の処理を記述する必要があります。  

[f:id:motikan2010:20220530224111p:plain:w700]

　全スクリプトの内容は以下のようになります。  
**HTMLを監視するためには`htmlSnapshot`関数を利用する必要があります。**  
[f:id:motikan2010:20220530224115p:plain:w700]

#### 監視結果を確認 (検知あり)

　Alertsログに「`digital_skimmer_gibberish hit`」があり、無事検知することができました。
[f:id:motikan2010:20220530224109p:plain:w700]

　文字化けしていますが、画面キャプチャを取得することも可能です。
[f:id:motikan2010:20220530224119p:plain:w700]

## まとめ

　2022年2月に公開されたソフトウェアあり、まだ実績などの効果は不明ですが、ECサイトを運営しておりWebスキマーを検出する仕組みを取り入れていない方は使ってみてはいかがでしょうか。　

## 参考

- <span><a href="https://github.com/target/mmk-ui-api" target="_blank">target/mmk-ui-api: UI, API, and Scanner (Rules Engine) services for Merry Maker</a></span>
- <span><a href="https://tech.target.com/blog/behind-the-scenes-of-merry-maker" target="_blank">Behind the Scenes of Merry Maker</a></span>
- <span><a href="https://tech.target.com/blog/meet-merry-maker" target="_blank">Meet Merry Maker: How Target Protects Against Digital Skimming</a></span>
