<div style="text-align:center;">[f:id:motikan2010:20201117002745p:plain:w500]</div>

<div class="contents-box"><p>[:contents]</p></div>  

## はじめに

　WordPress プグラインを開発するにあたって、脆弱性を作らないためにはどのような点に気を付ける必要があるのかを紹介していきます。  

### 読者対象

主に以下の方を対象に本記事を書いています。  

- WordPress プラグイン開発者
- WordPress プラグインの脆弱性を見つけたい方
- 脆弱なWordPress プラグインを開発したい方

### プラグイン開発で気を付ける脆弱性

本記事では以下の6つの脆弱性について紹介しています。  

1. PHPファイルへの直接アクセス
2. サードパーティライブラリ
3. 権限の検証に不備があるAPIの呼び出し
4. CSRF（Cross-site Request Forgery）
5. XSS（Cross-site scripting）
6. SQL インジェクション

　また、WPScan から「WordPress Plugin Security Testing Cheat Sheet」というセキュリティテストのチートシートが提供されております。  
 このチートシート内の説明は少し物足りないように見受けられますが、網羅的に脆弱性が紹介されていますので目を通しておくことをオススメします。  
[https://github.com/wpscanteam/wpscan/wiki/WordPress-Plugin-Security-Testing-Cheat-Sheet:embed:cite]  

## 脆弱性種類

　WordPress の特徴が強い脆弱性の順で紹介していきます。

### ① PHPファイルへの直接アクセス

　まず <span style="color: #ff0000">WordPress のドキュメントルート以下には誰でもアクセスすることが可能</span>です。

　プラグインのインストール先である 「`/wp-content/plugins`」配下も例外ではないため、プラグインに含まれるPHPファイルに WordPress 経由でなくとも直接アクセスすることが可能です。  

　そのため WordPress 経由でのコード実行を想定している場合に、直接アクセスされることで<span style="color: #ff0000">意図していない処理が行われる恐れ</span>があります。    

#### 対策

　<span style="color: #ff0000">直接PHPファイルが呼び出された場合にはエラーを返すようにし</span>、WordPress コード内から呼び出された場合にのみ処理を始めるようにします。  

具体的にはPHPファイルの先頭に以下のコードを記述します。  

<div class="md-code" style="width:100%">

```php
<?php

if ( !defined('ABSPATH') ) {
    die( 'Forbidden' );
}
```
</div>

　定数「`ABSPATH`」は `wp-config.php` で定義されることになっているので、WordPress 経由でないと「if より後続の処理が実行されない」ということになります。  
直接PHPファイルにアクセスされても処理が行われないようになります。  

#### 機密ファイルへの直接アクセス

　似たような特徴を持った脆弱性を1つ紹介します。  

　前述しましたがプラグインのディレクトリ配下は誰もがアクセスすることが可能です。

　そのディレクトリにアクセスログや管理操作ログなどの<span style="color: #ff0000">機密情報を保存するプラグインの場合</span>、それらのファイルにアクセスされ、内容が閲覧されてしまう可能性があります。実際にそのような脆弱性が報告されているプラグインも存在しています。    

　そこで機密情報を含んだファイルを扱っているプラグインは以下のような実装をすることで内容が閲覧されないように対策されていました。  

1. ファイル名に推測されないランダム文字列の利用  
→ ファイルにアクセスすることが難しくなる
2. ファイル内容の先頭に`<?php exit; ?>` と記述し、ファイル名を `*.php` で保存  
→ ファイルにアクセスされても内容が表示されない
<figure class="figure-image figure-image-fotolife" title="このようにすると秘密の情報が読み取られません">[f:id:motikan2010:20201117211421p:plain:w500]<figcaption>このようにすると秘密の情報が読み取られません</figcaption></figure>

　上の対策方法の片方を行えば問題ないと考えられますが、両方やっていた方がより安全かなと思っていたりします。  
何かの拍子に 「ディレクトリリスティングが有効状態」や「PHPが実行されずPHPファイルがダウンロードされる状態」になった場合のことを考えて。  

### ② サードパーティライブラリ

　サードパーティから提供されているプログラムを用いて WordPress プラグインを開発した場合に発生する可能性がある脆弱性です。  

#### 事例

　2つ事例を紹介します。

##### 事例① elFinder

　「File Manager」には `/wp-content/plugins/wp-file-manager/lib/php/connector.minimal.php` に特定のリクエストを送信することで任意のファイルをアップロードすることができるという脆弱性がありました。  

　これはライブラリ（elFinder）が <span style="color: #ff0000">WordPress を前提で開発されているものではなく、それを考慮せずにライブラリをディレクトリ配下に配置したのが原因で脆弱性が生じてしまった</span>と考えられます。  

[https://wpscan.com/vulnerability/10389:title]

　サードパーティライブラリを導入することで脆弱性が生じてしまうのは、ライブラリの導入方法が同様であれば脆弱性も同様に発生する点が特徴です。  

　実際に「augmented-reality（脆弱性が修正されず非公開状態）」という WordPress プラグインにも全く同じ脆弱性が発見されています。  
攻撃者は、このライブラリを用いているプラグインが他に存在するか確認していることでしょう。  

[https://wpscan.com/vulnerability/10457:title]

##### 事例② Epsilon Framework

　こちらの事例はプラグインではなくテーマの脆弱性です。 
 
しかし脆弱性が生じた原因は事例1と大きく変わらないです。

　Epsilon Framework を利用している15種のテーマから脆弱性が発見されたというものです。  

[https://blog.nintechnet.com/unauthenticated-function-injection-vulnerability-fixed-in-15-wordpress-themes/:embed:cite]  

　このことからサードパーティライブラリを利用してWordPressプラグインを開発する場合は、ライブラリ導入時に<span style="color: #ff0000">直接アクセスされたくないファイルは「除外」または「WordPress経由以外ではコードを実行させないように修正」</span>したほうが良いです。  
（例：デバッグ用のコードが含まれていそうなテストコード `tests/` 配下を削除）  

　しかしながらサードパーティーライブラリの中身を見てファイルの必要有無の判断やコードの修正するのは難しいです。  

日々のプラグイン脆弱性情報から開発で利用しているライブラリ経由で発生している脆弱性がないかを確認するようにし、該当するようであればパッチを当てる運用でも問題ないと思われます。

#### 対策

　開発プラグイン内で利用しているライブラリに脆弱性が存在していないかを検出する仕組みを導入します。  
また、脆弱性が存在している場合は「セキュリティパッチが適用されているライブラリに更新」または「開発プラグインには影響がないことを確認」する必要があります。  

### ③ 権限の検証に不備があるAPI

　WordPress プラグインは `register_rest_route` 関数 を用いることで、容易に WordPress サイトに REST API を実装することが可能です。  

　その API に<span style="color: #ff0000">権限の検証に不備</span>があったり、そもそも<span style="color: #ff0000">権限の検証をしていない</span>といったことが脆弱性になります。  

　脆弱性が悪用されることでプラグインの機能が意図していない第三者によって利用される恐れがあります。（記事の投稿・ユーザアカウントの作成 など）

#### APIが列挙できる REST API ルートエンドポイント

　サイトの REST API の定義内容は、REST API ルートエンドポイント（`?rest_route=/`）にアクセスすることで確認できるようになっています。  

　この機能は<span style="color: #ff0000">デフォルトで有効になっています</span>。無効にするには、WordPressのコードを修正するか、この機能を無効化するプラグインを導入する必要があります。  

　この定義情報は攻撃者にとっては有意義な情報であり無効状態が推奨されますが、普段使う機能でないためにデフォルト（有効）状態なっているサイトが多く見られます。  

<figure class="figure-image figure-image-fotolife" title="ルートエンドポイントにアクセスすると定義されているAPI一覧が表示される">[f:id:motikan2010:20201116204043p:plain:w500]<figcaption>ルートエンドポイントにアクセスすると定義されているAPI一覧が表示されます</figcaption></figure>

#### 認可処理がないAPI

　REST API のルートエンドポイントから API の定義情報からインストールされているプラグインが確認できることから、プラグインの REST API が攻撃の対象となってしまうことがあります。  

　そのため <span style="color: #ff0000">REST API にアクセスされた場合に権限の検証をすることは必須事項</span>です。  

　WordPress では権限を検証するために「`current_user_can`」関数が用意されており、簡単に権限の検証を行えるようになっています。  

　`current_user_can` 関数の詳しい使い方は以下を参照下さい。  
[https://elearn.jp/wpman/function/current_user_can.html:title]

　むやみに`current_user_can` 関数を使えば安全というわけではなく、<span style="color: #ff0000">引数として渡す権限のレベルにも気を付けましょう</span>。
寄稿者アカウントで管理者アカウント用の操作ができるようであれば安全とは言えません。  

　権限のレベルについては以下を参照下さい。  
[https://elearn.jp/wpman/column/c20110414_01.html:title]

#### 事例

　権限検証の不備で脆弱性が発生してしますが、「`current_user_can`」関数を用いることで脆弱性の修正が行われています。  

[https://blog.nintechnet.com/multiple-vulnerabilities-fixed-in-security-malware-scan-by-cleantalk-plugin/:embed:cite]

<figure class="figure-image figure-image-fotolife" title="権限の不備が起因となった脆弱性の修正内容">[f:id:motikan2010:20201116212145p:plain:w500]<figcaption>権限の不備が起因となった脆弱性の修正内容</figcaption></figure>

#### 対策

　前述した「`current_user_can`」関数などの現行ユーザの権限を取得し、各APIで正常に処理を続行してよいかを検証します。

### ④ CSRF（Cross-site Request Forgery）

　特定のWebアプリケーションフレームワークを用いて開発している場合は自動的にCSRFトークンの検証が行われますが、<span style="color: #ff0000">WordPress の場合は自動的にはCSRFの検証は行われません</span>。  

　CSRFトークンの検証はプラグイン開発者自身で実装する必要があります。  
実装といってもトークンの検証する `wp_verify_nonce` 関数が用意されてますので、簡単に実装することができます。

　`wp_verify_nonce` 関数を用いてトークンの検証する場合に気を付けることがあります。  

#### 事例

　以下の記事は「`wp_verify_nonce`」関数を用いているにも関わらず 25種のテーマで CSRF の脆弱性が発見されたというものです。  

[https://blog.nintechnet.com/25-wordpress-plugins-vulnerable-to-csrf-attacks/:embed:cite]  

　1つのパターンを引用し説明します。  
とある脆弱なプラグインでは以下のコードでCSRFの対策が行われていました。  

<div class="md-code" style="width:100%">

```php
if ( isset( $_POST['some-nonce'] ) && ! wp_verify_nonce( $_POST['some-nonce'], 'some-nonce' ) ) {
   exit( 'Potential CSRF attack detected.' );
}
```

</div>

　`some-nonce`パラメータが CSRF トークンであり、`wp_verify_nonce`関数を用いてCSRFの検証を行い、検証結果が `false` の場合に処理を中断するので CSRF 対策ができていそうです。  

　しかしこのコードでは `some-nonce` パラメータが送信されない場合が考慮されていないようです。  

`some-nonce` パラメータが送信されていない場合は `isset` 部分で `false` となりトークンの検証が行われずに後続処理が行われてしまいます。  

　トークンが送信されない場合はエラーとしたいので最初の条件は `! isset( $_REQUEST['some-nonce'] )` にする必要があります。  

　細かい点ではありますが、このような実装は実際のテーマで発見されているので、このようなパターンがあることを頭に置いていた方が良いでしょう。  

　このような小さなミスで脆弱性が発生するので、脆弱性を見つけてみたいという方には上の記事は面白いと思います。  

#### 対策

　誤ったCSRFトークンだけでなく、**CSRFトークンが送信されなかった場合**にもにエラーになるように実装する。

### ⑤ XSS（Cross-site scripting）

　ユーザの入力値を返すようなプラグインに発生する可能性がある脆弱性です。  

　Webアプリケーションフレームワークを用いて開発した場合、ブラウザへの出力する際に自動的にエスケープ（サニタイジング）されるのがほとんどですが、<span style="color: #ff0000">WordPress プラグイン開発では自動的にエスケープされません</span>。  

　<span style="color: #ff0000">WordPress ライブラリ開発ではエスケープ漏れの部分があると XSS の脆弱性が発生する可能性があります。</span>  

　PHPでは`htmlspecialchars`関数を用いてエスケープすることができますが、WordPress には表示箇所に応じたエスケープ処理をラップした関数が用意されていますので、それらを用いて開発した方が無難です。  

| 関数名 | 表示箇所 |
| - | - |
| esc_html | タグ要素内 |
| esc_attr  | タグ属性内 |
| esc_url | href属性内（利用できるスキームに制限が掛かるようになります） |
| esc_js  | script 内 |

#### 対策

　ブラウザに表示する値は「`htmlspecialchars`」関数などを用いてエスケープ処理します。  
（昨今のWebフレームワークではそのあたりは自動的に対策されますが、WordPressプラグイン開発の場合は開発者自身で対策します。）

### ⑥ SQLインジェクション

　読み込み、書き込み問わずデータベースにアクセスする必要があるプラグインに発生する可能性がある脆弱性です。  

　<span style="color: #ff0000">WordPress にはデフォルトで O/Rマッパー が用意されていないので</span>、SQLインジェクションの脆弱性が発見されることがたびたびあります。  

#### 対策

対策方法を2つ紹介します。

##### prepare 関数を用いる

　WordPress はデフォルトで O/Rマッパー を用意されておらず、データベースにアクセスするためには 素のSQL文 をコード内に記述する必要があります。  

　セキュリティを意識せずにSQL文を組み立てることで、ユーザが入力した任意の値がSQL文の中に挿入されてしまいます。  

　<span style="color: #ff0000">ユーザから入力値をエスケープするために `prepare`関数 を用いるようにしましょう。</span>  

`prepare`関数 の使い方は以下のページが参考になります。  
[https://wpdocs.osdn.jp/%E9%96%A2%E6%95%B0%E3%83%AA%E3%83%95%E3%82%A1%E3%83%AC%E3%83%B3%E3%82%B9/wpdb_Class#Protect_Queries_Against_SQL_Injection_Attacks:title]

　以下の差分はとあるプラグインの SQLインジェクション の修正コミットです。  

<figure class="figure-image figure-image-fotolife" title="SQLインジェクションの修正部分">[f:id:motikan2010:20201103214654p:plain:alt=SQLインジェクションの修正部分]<figcaption>SQLインジェクションの修正部分</figcaption></figure>

##### バリデーション を行う

　POST（投稿） ID のような数値が想定されている値については、ユーザからの入力値を int型にキャストする処理をします。  
また 数値であることの検証（バリデーション）するようにします。  

　実際、`prepare`関数 を使うような脆弱性の修正ではなく、脆弱性があるパラメータを int型にキャストする 修正がいくつかありました。  

## まとめ

　本記事で紹介したようにセキュアなWordPressプラグインを開発する場合は、脆弱性を生み出さないことを考えながら開発（コーディング）を行っていく必要があります。  
Webアプリケーションフレームワークのように自動的に対策されるような脆弱性は皆無と言ってもよいでしょう。
（その点はレガシーと感じてしまう部分となっていますが・・・。）
そのため日々、様々なプラグインで脆弱性が報告されている状態であり、シェアの大きいプラグインであっても、深刻な脆弱性が見つかることも珍しくありません。

　このことからWordPressプラグイン開発者は常にプラグインが攻撃の対象になることを意識しながら開発する必要があります。  
たとえリリースしたプラグインの導入数が少なくても脆弱性が発見・報告されるということは当然のようにありますので、利用者が少ないから安全というわけではなりません。  

　また、何でもいいから脆弱性を発見したいという方は、WordPress プラグインは脆弱性が生まれやすいかつOSSなので、オススメの領域だと思います。      
PHPコードの静的解析で脆弱性を検出する<span><a href="https://github.com/AsaiKen/phpscan" target="_blank">ツール</a></span>も存在しているようです。（試してはいません。）

　今回は WordPress プラグインの特色が強い脆弱性を紹介しましたが、本記事で紹介した脆弱性以外にもプラグインに潜む脆弱性は多々あります。  
（例：アップロードファイルの検証が行われずPHPファイルなどのファイルがアップロードが可能になっている 等）  

## 参考

- <span><a href="https://www.slideshare.net/owaspnagoya/owasp-wordpress-wordpress" target="_blank">OWASP WordPressセキュリティ実装ガイドライン （セキュアなWordPressの構築）</a></span>
- <span><a href="https://wordpress.org/support/article/roles-and-capabilities/" target="_blank">Roles and Capabilities | WordPress.org</a></span>
- <span><a href="https://elearn.jp/wpman/" target="_blank">WordPress私的マニュアル – WordPressを使ったサイト作成資料として</a></span>

## 更新履歴

- 2020年11月17日 新規作成
- 2020年11月24日 「WordPress Plugin Security Testing Cheat Sheet」のリンク追加
