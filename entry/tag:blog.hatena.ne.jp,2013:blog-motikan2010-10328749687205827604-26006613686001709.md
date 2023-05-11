<div style="text-align:center;">[f:id:motikan2010:20210202005126p:plain:w500]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　この記事は先日書いた「GitHubのCode Scanningを使ってみる」の続きです。  
Code Scanning の導入は下の記事を参照ください。
[https://blog.motikan2010.com/entry/2020/10/11/%E3%80%90%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E3%80%91GitHub%E3%81%AECode_Scanning%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B:embed:cite]  

　前回は GitHub Code Scanning によって「SQL Injection」、「Insecure Deserialization（安全でないデシリアライゼーション）」、「Cross-Site Scripting（XSS）」が検出されるのかを検証しました。  

　本記事では前回に引き続き下記3つの脆弱性が検出されるのかを確認していきます。

- RCE (Remote Code Execution)
- Directory Traversal
- XML External Entity 攻撃

　さらに今回は脆弱性の検出有無の確認だけでなく、<span style="color: #ff0000">Code Scanning の検出レポートを参考に脆弱性の修正し、脆弱性が検出されなくなったことも確認します</span>。

## スキャン実行

　前回と同様に脆弱性を含んだ Java(Spring Boot) のアプリケーションで確認していきます。  

下のリポジトリが脆弱性を含んだアプリケーションです。
[https://github.com/motikan2010/GitHub-code-scanning-Test/pulls:embed:cite]

　スキャン結果の内容については前回の記事で説明しているため流す程度にします。  

### RCE (Remote Code Execution)

　最初は RCE (Remote Code Execution) の確認です。  

　Code Scanning はプルリク作成時に実行されます。 
<span><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/6" target="_blank">RCE を含んだコードのプルリク</a></span>を作成したので、想定通りであれば脆弱性が検出されるはずです。  

　スキャン結果は下画像の通り脆弱性が検出されました。  

[f:id:motikan2010:20210201223220p:plain]  

　検出された脆弱性の詳細も確認することもできます。  

[f:id:motikan2010:20210201230111p:plain]  

### Directory Traversal

　次は <span><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/8" target="_blank">Directory Traversal の脆弱性</a></span> です。  

先ほどと同様に脆弱性が検出されました。

[f:id:motikan2010:20210201223224p:plain]  

　脆弱性の詳細は下画像の通りです。  

[f:id:motikan2010:20210201230115p:plain]  

### XML External Entity 攻撃

　最後は <span><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/7" target="_blank">XML External Entity 攻撃</a></span> です。  

この脆弱性も検出されました。

[f:id:motikan2010:20210201223228p:plain]  

　脆弱性の詳細も先ほとど同様に下の画像の通り表示されました。  

この詳細には脆弱性の修正方法も記載されているので、次はこの記載に従ってソースコードを修正し、脆弱性が検出されなくなったことを確認していきます。  

[f:id:motikan2010:20210201230119p:plain]  

## 脆弱性を修正

　ここで修正していく脆弱性は「XML External Entity 攻撃」です。  

　修正前のアプリケーションに対しては攻撃が成功できることが確認できます。  
<figure class="figure-image figure-image-fotolife" title="脆弱性の修正前">[f:id:motikan2010:20210202002512p:plain]<figcaption>脆弱性の修正前</figcaption></figure>

　検出された脆弱性の詳細の下部に脆弱性の修正方法が記載されています。   
XML External Entity 攻撃 の場合は下画像のように表示されています。  
[f:id:motikan2010:20210201230127p:plain]

修正方法として

> The best way to prevent XXE attacks is to disable the parsing of any Document Type Declarations (DTDs) in untrusted data. 

と記載されていますので、 DTD(Document Type Definition) を無効にすれば良さそうです。  

そして無効化するためのサンプルコードも記載されています。下記のコードを追加すればよさそうです。  

<div class="md-code" style="width:100%">
```java
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
```
</div>

　先ほどの脆弱性が検出されたコードに<span><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/9/commits/cf12532279d2b4d3f9b604d78e9fbce6f8c93284" target="_blank">この修正を加えてみて</a></span>、再度スキャンを実行してみます。  

　スキャン結果：脆弱性が検出されなくなりました。  
[f:id:motikan2010:20210202001914p:plain]  

　修正後のアプリケーションに対して攻撃をしてみます。結果はエラーとなり攻撃は成功せず脆弱性は修正されたことが確認できます。  
<figure class="figure-image figure-image-fotolife" title="脆弱性の修正後">[f:id:motikan2010:20210202002416p:plain]<figcaption>脆弱性の修正後</figcaption></figure>

　アプリケーション側のエラーメッセージからXML解析に制限が掛かっていることが確認できます。  
<figure class="figure-image figure-image-fotolife" title="XML解析に制限が掛かっている">[f:id:motikan2010:20210202002310p:plain]<figcaption>XML解析に制限が掛かっている</figcaption></figure>

## まとめ

　今回は3種類の脆弱性を Code Scanning で検出できるのかを確認してみましたが、問題なく検出することができました。  
そして検出された脆弱性に関しては、検出レポート通りにアプリケーションを修正することで脆弱性が検出されなくなることも確認できました。  

　Code Scanning は脆弱性を検出するためのツールですが、検出レポートの説明が詳細に記載されているためセキュリティの学習のツールとしても活躍しそうです。  
（学習者が事前に脆弱性の含んだアプリケーションを修正するなど...）

## 参考

- <span><a href="https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing" target="_blank">XML External Entity (XXE) Processing | OWASP</a></span>
- <span><a href="https://www.mbsd.jp/blog/20171130.html" target="_blank">XXE攻撃 基本編 | MBSD Blog</a></span> 

## 更新履歴

- 2021年2月1日 新規作成
