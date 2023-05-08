<div style="text-align:center;">[f:id:motikan2010:20180211221846p:plain]</div>  

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　Webアプリケーションのセキュリティを診断するツールとして、Burp Suiteが代表の１つとしてあり、拡張プラグインに頼らなくても標準の機能で多種類の診断を行うことができるほど多機能です。    

　しかし、Webアプリケーションによっては遷移方法が特殊な場合がありは、拡張プラグインに頼らなくてはいけない場合もあります。  (値が更新されるCSRFトークンが存在する等)

　そこで、Burp Siteでは**「BApp Store」に多くの人が開発した、拡張プラグインが公開されています。**  

#### 拡張プラグインのストア「BApp Store」

　「BApp Store」で公開されている拡張プラグインで事足りるのであれば、独自に拡張プラグインを開発する必要はないと思っています。    

[https://portswigger.net/bappstore:embed:cite]

　しかし、公開されている拡張プラグインで対応できない診断対象に対応するためにも、独自に拡張プラグインを作成し、**より妥協のないセキュリティ診断を行うため**にも拡張プラグインの開発方法を学習していきます。

<!-- more -->

　ユニークな拡張プラグインを開発を開発できたら、BApp Storeに公開するのもありだと思います。  
　良い拡張プラグインを開発する際に気を付ける点は下記のブログにまとめられています。  

[http://blog.portswigger.net/2018/01/your-recipe-for-bapp-store-success.html:embed:cite]

　今回は手始めに「Hello, world!」拡張プラグインを開発していきます。  
拡張ということもあり、Burp Suiteにメインの処理は任せるようにし、送信するリクエストの加工や受信したレスポンスの検査等を拡張プラグインで行うようになっていくと考えています。

## 開発環境

　拡張プラグインは <span style="color: #ff0000">**Java・Ruby・Python で開発できます**</span>が、今回はJavaを用いて開発を行なっていきます。  

今回作成するプロジェクトのリポジトリは下記になります。

[https://github.com/motikan2010/Burp-Suite-Learning-Chapter01:title]

　私の開発環境は以下のようになっています。  

| | |
|-|-|
| OS | macOS 10.12 Sierra |
| プログラム言語 | Java8 |
| ビルドツール | Maven |
| IDE | IntelliJ IDEA |

## 実装

### 1. ディレクトリ構造

　最終的なディレクトリ構造は下記のようになります。

<div class="md-code" style="width:100%">
```
.
├── pom.xml
└── src
    └── main
        └── java
            └── burp
                └── BurpExtender.java
```
</div>

### 2. Burp拡張ライブラリ(Burp Extender API)の取得

[https://mvnrepository.com/artifact/net.portswigger.burp.extender/burp-extender-api:title]

　Burp Extender API は "Maven Repository" に登録されており、`pom.xml`ファイルを編集することで導入することができます。

#### pom.xml の編集

<div class="md-code" style="width:100%">
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.motikan2010</groupId>
  <artifactId>burplearning</artifactId>
  <version>1.0-SNAPSHOT</version>

  <dependencies>
    <!-- https://mvnrepository.com/artifact/net.portswigger.burp.extender/burp-extender-api -->
    <dependency>
      <groupId>net.portswigger.burp.extender</groupId>
      <artifactId>burp-extender-api</artifactId>
      <version>1.7.22</version>
    </dependency>
  </dependencies>
</project>
```
</div>

### 3. Javaファイルの作成

　最初はライブラリ読込み時に実行されるコードを書いていきます。  

　Burp拡張プラグインは、Javaでよく見られる mainメソッド から実行ではありません。  

　下記の条件に合致するクラスを作成します。  

- パッケージが <span style="color: #ff0000">burp</span> に属している
- クラス名が <span style="color: #ff0000">BurpExtender</span>
- インターフェース <span style="color: #ff0000">IBurpExtender</span> を実装

　実行コードは、<b>registerExtenderCallbacksメソッド</b>内に書いていきます。

- BurpExtender.java  
<div class="md-code" style="width:100%">
```java
package burp;

import java.io.PrintWriter;

public class BurpExtender implements IBurpExtender {

    public void registerExtenderCallbacks(IBurpExtenderCallbacks iBurpExtenderCallbacks) {
        // ここにコードを書いていく
    }
}
```
</div>

　この記事では下記を学んでいきます。  

- プラグイン名の設定と表示
- 「Output」タブに標準メッセージの表示
- 「Error」タブにエラーメッセージの表示、例外メッセージの表示
- 「アラート」タブにメッセージの表示

### 4. プラグインに名前を付ける

[f:id:motikan2010:20180211204005p:plain]  

　`Name列`の値を指定できます。  

<div class="md-code" style="width:100%">
```java
// 拡張プラグインの命名
iBurpExtenderCallbacks.setExtensionName("Hello, Burp Suite");
```
</div>

### 5. 「Output」タブに表示

[f:id:motikan2010:20180211204024p:plain]  

<div class="md-code" style="width:100%">
```java
// メッセージを表示
PrintWriter stdout = new PrintWriter(iBurpExtenderCallbacks.getStdout(), true);
stdout.println("INFO : Hello, Burp Suite");
```
</div>

### 6. 「Errors」タブに表示

<div class="md-code" style="width:100%">
```java
// エラーメッセージを表示
PrintWriter stderr = new PrintWriter(iBurpExtenderCallbacks.getStderr(), true);
stderr.println("ERROR : Hello, Burp Suite");
```
</div>

### 7. 例外メッセージを表示

　例外メッセージはエラーと同じタブに表示されるようになっています。  
[f:id:motikan2010:20180211204047p:plain]  

<div class="md-code" style="width:100%">
```java
throw new RuntimeException("Burp Suite exceptions");
```
</div>

### 8. 「アラート」タブに表示

[f:id:motikan2010:20180211204103p:plain]  

<div class="md-code" style="width:100%">
```java
iBurpExtenderCallbacks.issueAlert("Burp Suite Alerts");
```
</div>

## コンパイル から プラグインの読み込み まで

　最終的に完成したコードは下記の通りになります。  
このファイルをコンパイル（ビルド）し、Burp Suite で読み込んで実行させます。  

- BurpExtender.java
<div class="md-code" style="width:100%">
```java
package burp;

import java.io.PrintWriter;

public class BurpExtender implements IBurpExtender {

    public void registerExtenderCallbacks(IBurpExtenderCallbacks iBurpExtenderCallbacks) {
        iBurpExtenderCallbacks.setExtensionName("Hello, Burp Suite");

        PrintWriter stdout = new PrintWriter(iBurpExtenderCallbacks.getStdout(), true);
        stdout.println("INFO : Hello, Burp Suite");

        PrintWriter stderr = new PrintWriter(iBurpExtenderCallbacks.getStderr(), true);
        stderr.println("ERROR : Hello, Burp Suite");

        iBurpExtenderCallbacks.issueAlert("Burp Suite Alerts");

        throw new RuntimeException("Burp Suite exceptions");
    }
}
```
</div>

### プロジェクトの設定

・`File` > `Project Structure` > `Artifacts` > `From modules with dependencies...`
[f:id:motikan2010:20180211204139p:plain]

・`OK`  
[f:id:motikan2010:20180211204329p:plain]

・`Apply` > `OK`  
[f:id:motikan2010:20180211204720p:plain]

### ビルド

・`Build` > `Build Artifacts`  
[f:id:motikan2010:20180211204812p:plain]

### 拡張プラグインの読込み

・`Extender タブ` > `Extensions タブ` > `Add ボタン`
[f:id:motikan2010:20180211213544p:plain]

・`Select file ...`
[f:id:motikan2010:20180211213540p:plain]

　以上、「Hello, world!」拡張プラグインの作成と実行ができました。  

　次回は、Burp Suiteで取得した<span style="color: #ff0000">リクエストとレスポンスのヘッダ・ボディを表示する</span>拡張プラグインを作成していきます。  
[https://blog.motikan2010.com/entry/2018/02/13/Burp_Suite%E3%81%AE%E6%8B%A1%E5%BC%B5%E3%82%92%E4%BD%9C%E6%88%90%E5%85%A5%E9%96%80_%E3%81%9D%E3%81%AE%EF%BC%92_-_%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%26%E3%83%AC%E3%82%B9%E3%83%9D%E3%83%B3:embed:cite]

## 更新履歴

- 2018年2月11日 新規作成