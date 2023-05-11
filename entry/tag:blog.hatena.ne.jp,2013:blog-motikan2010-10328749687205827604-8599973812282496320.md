<div style="text-align:center;">[f:id:motikan2010:20170725011556j:plain:w400]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　今回はJava製のHTTPライブラリ「LittleProxy」を使ってみます。  

　Java製のHTTPライブラリは種類豊富だと思っていたのだが、想像していたよりも圧倒的に少なかった。  
そんなことでデファクトスタンダードとなっているライブラリがなさそうなので、ほどほどにメンテナンスされている「LittleProxy」を選定しました。（とはいっても最後の修正は 2017/09/25 です）

[https://github.com/adamfisk/LittleProxy:embed:cite]

　特徴として、LittleProxy の内部では Netty 4.1 が利用されており軽快に動作します。  
リポジトリの説明にも High performance HTTP proxy と記載あります。  

<!-- more -->

## 準備

　プロジェクト管理ツールは"Maven"を使用して「LittleProxy」を導入しています。

<div class="md-code" style="width:100%">
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.motikan2010</groupId>
    <artifactId>littleproxysample</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>org.littleshoot</groupId>
            <artifactId>littleproxy</artifactId>
            <version>1.1.2</version>
        </dependency>
    </dependencies>

</project>
```
</div>

## 動作確認

　今回、使用したコードは下記のリポジトリにあります。

[https://github.com/motikan/SampleLittleProxy/tree/15a48892c2370e24013f4cd36078ae198433658d:title]  

　プロキシを作成するにあたって主に利用するメソッドは下記の４つのメソッドであり、オーバーライドして利用します。  

[f:id:motikan2010:20170725011900j:plain:w600]  

・クライアント　→　プロキシ  
・プロキシ　　　→　サーバ  
・プロキシ　　　←　サーバ   
・クライアント　←　プロキシ  
　のように各ノード間の通信毎に、特定のメソッド呼び出されるようになっております。

　プロキシの動作は下記の2種類のcurlコマンドで確認しています。

<div class="md-code" style="width:100%">
```
$ curl -x 127.0.0.1:8080 http://example.com
$ curl -x 127.0.0.1:8080 http://example.com -d "testKey=testValue" -d "testKey2=testValue2" --cookie 'CookieKey1=CookieVal1'
```
</div>

### クライアント → プロキシ 通信（clientToProxyRequest メソッド）

[f:id:motikan2010:20170725012019j:plain:w600]  
　「クライアント」から「プロキシ」へのリクエスト通信を取得できます。

<div class="md-code" style="width:100%">
```java
@Override
public HttpResponse clientToProxyRequest(HttpObject httpObject) {
  System.out.println("=== clientToProxyRequest ===");
  if (httpObject instanceof HttpRequest) {
    System.out.println(httpObject.toString());
  }
  return null;
}
```
</div>

<div class="md-code" style="width:100%">
```
GET http://example.com/ HTTP/1.1
Host: example.com
User-Agent: curl/7.43.0
Accept: */*
Proxy-Connection: Keep-Alive
Content-Length: 0
```
</div>

#### HttpRequestオブジェクトに用意されているメソッド

<div class="md-code" style="width:100%">
```java
// メソッド
System.out.println("HttpRequest.getMethod() => "
        + ((HttpRequest) httpObject).getMethod());
        //=> GET

// URI
System.out.println("HttpRequest.getUri() => "
        + ((HttpRequest) httpObject).getUri());
        //=> http://example.com/

// HTTPバージョン
System.out.println("HttpRequest.getProtocolVersion() => "
        + ((HttpRequest) httpObject).getProtocolVersion());
        //=> HTTP/1.1

// ヘッダー
HttpHeaders httpHeaders = ((HttpRequest) httpObject).headers();
List<Map.Entry<String,String>> headerList = httpHeaders.entries();
for (Map.Entry<String, String> header : headerList){
    System.out.println(header.getKey() + ": " + header.getValue());
}
// Host: example.comUser-Agent: curl/7.43.0
// Accept: */*
// Proxy-Connection: Keep-Alive
// Content-Length: 0
// Content-Length: 37
// Content-Type: application/x-www-form-urlencoded
```
</div>

#### HttpContentオブジェクトに用意されているメソッド

<div class="md-code" style="width:100%">
```java
if (httpObject instanceof HttpContent) {
    HttpContent httpContent = (HttpContent) httpObject;
    String resposeBody = httpContent.content().toString(Charset.defaultCharset());
    System.out.println(resposeBody);
    //=> testKey=testValue&testKey2=testValue2
}
```
</div>

### プロキシ → サーバ 通信（proxyToServerRequest メソッド）

[f:id:motikan2010:20170725012101j:plain:w600]  

　「プロキシ」から「サーバ」へのリクエスト通信を取得できます。

<div class="md-code" style="width:100%">
```java
@Override
public HttpResponse proxyToServerRequest(HttpObject httpObject) {
  System.out.println("=== proxyToServerRequest ===");
  if (httpObject instanceof HttpRequest) {
    System.out.println(httpObject.toString());
  }
  return null;
}
```
</div>

<div class="md-code" style="width:100%">
```
GET / HTTP/1.1
Host: example.com
User-Agent: curl/7.43.0
Accept: */*
Content-Length: 0
Via: 1.1 XXXXX-no-MacBook-Pro.local
```
</div>

### プロキシ ← サーバ 通信（serverToProxyResponse メソッド）

[f:id:motikan2010:20170725012151j:plain:w600]  

　「サーバ」から「プロキシ」へのレスポンス通信を取得できます。

<div class="md-code" style="width:100%">
```java
@Override
public HttpObject serverToProxyResponse(HttpObject httpObject) {
  // レスポンスヘッダ
  if (httpObject instanceof HttpResponse) {
    HttpResponse httpResponse = (HttpResponse) httpObject;
    System.out.println(httpResponse.toString());
    System.out.println();
  }

  // レスポンスボディ
  if (httpObject instanceof HttpContent) {
    HttpContent httpContent = (HttpContent) httpObject;
    String resposeBody = httpContent.content().toString(Charset.defaultCharset());
    System.out.println(resposeBody);
  }
  return httpObject;
}
```
</div>

<div class="md-code" style="width:100%">
```
HTTP/1.1 200 OK
Cache-Control: max-age=604800
Content-Type: text/html
Date: Mon, 24 Jul 2017 14:01:08 GMT
Etag: "359670651+ident"
Expires: Mon, 31 Jul 2017 14:01:08 GMT
Last-Modified: Fri, 09 Aug 2013 23:54:35 GMT
Server: ECS (rhv/818F)
Vary: Accept-Encoding
X-Cache: HIT
Content-Length: 1270

<!doctype html>
<html>
<head>
    <title>Example Domain</title>

（中略）

</div>
</body>
</html>
```
</div>

#### HttpRequestオブジェクトに用意されているメソッド

<div class="md-code" style="width:100%">
```java
System.out.println("HttpResponse.getProtocolVersion() => "
        + ((HttpResponse) httpObject).getProtocolVersion());
        //=> HTTP/1.1

System.out.println("HttpResponse.getStatus() => "
        + ((HttpResponse) httpObject).getStatus());
        //=> 200 OK

// ヘッダー
HttpHeaders httpHeaders = ((HttpResponse) httpObject).headers();
List<Map.Entry<String,String>> headerList = httpHeaders.entries();
for (Map.Entry<String, String> header : headerList){
    System.out.println(header.getKey() + ": " + header.getValue());
}
// Accept-Ranges: bytes
// Cache-Control: max-age=604800
// Content-Type: text/html
// Date: Mon, 24 Jul 2017 15:16:04 GMT
// Etag: "359670651"
// Expires: Mon, 31 Jul 2017 15:16:04 GMT
// Last-Modified: Fri, 09 Aug 2013 23:54:35 GMT
// Server: EOS (lax004/2816)
// Content-Length: 1270
```
</div>

### クライアント ← プロキシ 通信（proxyToClientResponse メソッド）

[f:id:motikan2010:20170725012125j:plain:w600]  

　「プロキシ」から「クライアント」へのレスポンス通信を取得できます。

<div class="md-code" style="width:100%">
```java
@Override
public HttpObject proxyToClientResponse(HttpObject httpObject) {
  // レスポンスヘッダ
  if (httpObject instanceof HttpResponse) {
    HttpResponse httpResponse = (HttpResponse) httpObject;
    System.out.println(httpResponse.toString());
    System.out.println();
  }

  // レスポンスボディ
  if (httpObject instanceof HttpContent) {
    HttpContent httpContent = (HttpContent) httpObject;
    String resposeBody = httpContent.content().toString(Charset.defaultCharset());
    System.out.println(resposeBody);
  }
  return httpObject;
}
```
</div>

<div class="md-code" style="width:100%">
```
HTTP/1.1 200 OK
Cache-Control: max-age=604800
Content-Type: text/html
Date: Mon, 24 Jul 2017 14:01:08 GMT
Etag: "359670651+ident"
Expires: Mon, 31 Jul 2017 14:01:08 GMT
Last-Modified: Fri, 09 Aug 2013 23:54:35 GMT
Server: ECS (rhv/818F)
Vary: Accept-Encoding
X-Cache: HIT
Content-Length: 1270
Via: 1.1 XXXXX-no-MacBook-Pro.local

<!doctype html>
<html>
<head>
    <title>Example Domain</title>

（中略）

</body>
</html>
```
</div>

## プロキシでいろいろ

### レスポンスボディの改ざん

[https://github.com/MediumOne/littleproxy-example/blob/master/src/main/java/m1/learning/littleproxy/example/filters/ReplacePostContentFilterProxy.java:title]

　プロキシの得意分野である、レスポンスボディの改ざん（リプレース）をやってみます。  
「プロキシ」から「ブラウザ」に返されるレスポンスボディを「Example」から「motikan2010」へ文字列を置き換えています。

<div class="md-code" style="width:100%">
```java
@Override
public HttpObject proxyToClientResponse(HttpObject httpObject) {
  if (httpObject instanceof FullHttpResponse) {
      httpObject = doReplace((FullHttpResponse) httpObject);
  }
  return httpObject;
}

private FullHttpResponse doReplace(FullHttpResponse fullHttpResponse){

  CompositeByteBuf contentBuf = (CompositeByteBuf) fullHttpResponse.content();

  String contentStr = contentBuf.toString(CharsetUtil.UTF_8);
  String newBody = contentStr.replace("Example", "motikan2010");

  ByteBuf bodyContent = Unpooled.copiedBuffer(newBody, CharsetUtil.UTF_8);

  contentBuf.clear().writeBytes(bodyContent);
  HttpHeaders.setContentLength(fullHttpResponse, newBody.length());
  return fullHttpResponse;
}
```
</div>

<div class="md-code" style="width:100%">
```
$ curl -x 127.0.0.1:8080 http://example.com <!doctype html>
<html>
<head>
    <title>motikan2010 Domain</title>

（中略）

<body>
<div>
    <h1>motikan2010 Domain</h1>
    <p>This domain is established to be used for illustrative examples in documents. You may use this
    domain in examples without prior coordination or asking for permission.</p>
    <p><a href="http://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>
```
</div>

### リクエストのブロック

　プロキシはクライアントからの特定の通信をブロックするのも得意分野です。  
試しに画像ファイルの取得リクエストの場合にエラーを返すようにしてみます。

[https://github.com/MediumOne/littleproxy-example/blob/master/src/main/java/m1/learning/littleproxy/example/filters/BlockingFilterProxy.java:title]

<div class="md-code" style="width:100%">
```java
@Override
public HttpResponse clientToProxyRequest(HttpObject httpObject) {
  if(httpObject instanceof  HttpRequest) {
      HttpRequest request = (HttpRequest) httpObject;
      if(request.getUri().endsWith("png") || request.getUri().endsWith("jpeg")){ // 画像ファイルの判定
          return getBadGatewayResponse();
      }
  }

  return null;
}

// エラーレスポンスの生成
private HttpResponse getBadGatewayResponse() {
  String body = "<!DOCTYPE HTML \"-//IETF//DTD HTML 2.0//EN\">\n"
          + "<html><head>\n"
          + "<title>"+"Bad Gateway"+"</title>\n"
          + "</head><body>\n"
          + "An error occurred"
          + "</body></html>\n";
  byte[] bytes = body.getBytes(Charset.forName("UTF-8"));
  ByteBuf content = Unpooled.copiedBuffer(bytes);
  HttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.BAD_GATEWAY, content);
  response.headers().set(HttpHeaders.Names.CONTENT_LENGTH, bytes.length);
  response.headers().set("Content-Type", "text/html; charset=UTF-8");
  response.headers().set("Date", ProxyUtils.formatDate(new Date()));
  response.headers().set(HttpHeaders.Names.CONNECTION, "close");
  return response;
}
```
</div>

<div class="md-code" style="width:100%">
```
$ curl -x 127.0.0.1:8080 http://hatenablog.com/images/touch/guide-app/apple-badge@2x.png -vv
*   Trying 127.0.0.1...
* Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
> GET http://hatenablog.com/images/touch/guide-app/apple-badge@2x.png HTTP/1.1
> Host: hatenablog.com
> User-Agent: curl/7.43.0
> Accept: */*
> Proxy-Connection: Keep-Alive
>
< HTTP/1.1 502 Bad Gateway
< Content-Length: 130
< Content-Type: text/html; charset=UTF-8
< Date: Mon, 24 Jul 2017 16:04:17 GMT
< Connection: close
<
<!DOCTYPE HTML "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>Bad Gateway</title>
</head><body>
An error occurred</body></html>
* Closing connection 0
```
</div>

## まとめ

　意外にもJavaでフォワードプロキシ開発の情報が少なかった。  

　FiddlerCoreが有名なこともあってか、C#にシェアが取られているのかな。
そんなことも思いながらLittleProxyをさわってみましたが、想像していたより、十分な機能を持っていると思います。  

　今回は試しておりませんが、SSLにも対応しているという情報もありましたので、次に試してみる。HTTPSの通信を取れないプロキシなんて・・。  
Javaという利点を活かして最終的にはOWASP ZAPからコードを拝借して、SaaS型の診断ツールなんかを作ってみたい...。  

## 更新履歴

- 2017年7月25日 新規作成
- 2020年10月24日 「はじめに」を微修正
