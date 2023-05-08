<div style="text-align:center;">[f:id:motikan2010:20170514015521p:plain:w500]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　前回に引き続き「jwt-go」でいろいろ試してみます。  
今回は<span class="m-y">署名アルゴリズムを改ざんして送信</span>したときの挙動を確認していきます。  

[http://motikan2010.hatenadiary.com/entry/2017/05/12/jwt-go%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B:embed:cite]  

## 動作確認

### 署名アルゴリズムを改ざん

　なぜこんなことを試すのかというと、<span class="m-y">トークン内の署名アルゴリズムを改ざんしてリクエストを送信したときに改ざん後の署名アルゴリズムで署名の検証が行われる</span>実装があるようです。  
  
　詳しくは下記の記事を参照下さい。

[http://oauth.jp/blog/2015/03/16/common-jws-implementation-vulnerability/:embed:cite]  

　jwt-goでは署名アルゴリズムを改竄して送信したときにどのような動作をするのかを確認していきます。  

[f:id:motikan2010:20170514014545j:plain]  

<!-- more -->

　確認に使うソースコードは前回と同様です。  

[https://github.com/motikan/jwt-go_Sample/blob/master/main.go:title]  

#### ① トークンを取得

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/
GET /api/ HTTP/1.1
Host: example.jp:8080

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Sat, 13 May 2017 14:18:34 GMT
Content-Length: 144

{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTQ2OTM4NDgsInVzZXIiOiLjgrLjgrnjg4gifQ.iTEWurGMvi1d90yMW0OnqbQ0QDEyB-UD4TmYF9YQXYY"}
```
</div>

　トークンヘッダの署名アルゴリスムを改ざんします。

|||bsae64エンコード|
|-|-|-|
|改ざん前|{"alg":"HS256","typ":"JWT"}|eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9|
|改ざん後|{"alg":"none","typ":"JWT"}|eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K|

#### ② 署名アルゴリズムを"none"に改ざんしてリクエストを送信

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/private/ -H "Authorization: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJleHAiOjE0OTQ2OTM4NDgsInVzZXIiOiLjgrLjgrnjg4gifQ."
GET /api/private/ HTTP/1.1
Host: example.jp:8080
Authorization: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJleHAiOjE0OTQ2OTM4NDgsInVzZXIiOiLjgrLjgrnjg4gifQ.

HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8
Date: Sat, 13 May 2017 14:30:26 GMT
Content-Length: 49

{"error":"'none' signature type is not allowed"}
```
</div>

　ステータスコードは「401 Unauthorized」、レスポンスボディに「`'none' signature type is not allowed`」とある通り、
改ざん後の署名アルゴリズムが適用されず、<span style="color: #d32f2f">署名の検証には失敗しました</span>。  
[f:id:motikan2010:20170514014735j:plain]  

### トークン発行時「SHA256」、検証には「none」

　"none"にするため、ソースコードの下記の部分を変更します。

<div class="md-code" style="width:100%">
```go
/*
   署名の検証
*/
token, err := request.ParseFromRequest(c.Request, request.OAuth2Extractor, func(token *jwt.Token) (interface{}, error) {
	//b := []byte(secretKey)
	b := jwt.UnsafeAllowNoneSignatureType
	return b, nil
})
```
</div>

[f:id:motikan2010:20170514015036j:plain]  

#### 署名アルゴリズムを"none"に改ざんしてリクエストを送信

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/private/ -H "Authorization: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJleHAiOjE0OTQ2OTM4NDgsInVzZXIiOiLjgrLjgrnjg4gifQ."
GET /api/private/ HTTP/1.1
Host: example.jp:8080
Authorization: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJleHAiOjE0OTQ2OTM4NDgsInVzZXIiOiLjgrLjgrnjg4gifQ.

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Sat, 13 May 2017 15:46:04 GMT
Content-Length: 56

{"message":"こんにちは、「 ゲスト 」さん"}
```
</div>

　署名の検証が行われていないことがわかる。  

#### おまけ

　ちなみに署名アルゴリズムを<b>noneに指定した状態で、シグネチャを付与</b>しリクエストを送信した場合は、以下のようなエラーになりました。

||base64エンコード|
|-|-|
|{"alg":"none","typ":"JWT"}|eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K|

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/private/ -H "Authorization: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJleHAiOjE0OTQ2OTM4NDgsInVzZXIiOiLjgrLjgrnjg4gifQ.SetZ6qLSbfIObsaZSNGS4hVh5h8ob0Kr4h1fJGA75-s"
GET /api/private/ HTTP/1.1
Host: example.jp:8080
Authorization: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJleHAiOjE0OTQ2OTM4NDgsInVzZXIiOiLjgrLjgrnjg4gifQ.SetZ6qLSbfIObsaZSNGS4hVh5h8ob0Kr4h1fJGA75-s

HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8
Date: Sat, 13 May 2017 16:11:03 GMT
Content-Length: 59

{"error":"'none' signing method with non-empty signature"}
```
</div>

　"none"を指定した場合はシグネチャを付与するなと怒られました。

### トークン発行時「none」、検証には「SHA256」

[f:id:motikan2010:20170514014808j:plain]  

#### ① トークンを取得

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/
GET /api/ HTTP/1.1
Host: example.jp:8080
User-Agent: curl/7.43.0
Accept: */*

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Sat, 13 May 2017 16:18:49 GMT
Content-Length: 100

{"token":"eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJleHAiOjE0OTQ2OTU5MjksInVzZXIiOiLjgrLjgrnjg4gifQ."}
```
</div>

#### ② 受信したトークンを取得

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/private/ -H "Authorization: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJleHAiOjE0OTQ2OTU5MjksInVzZXIiOiLjgrLjgrnjg4gifQ."
GET /api/private/ HTTP/1.1
Host: example.jp:8080
Authorization: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJleHAiOjE0OTQ2OTU5MjksInVzZXIiOiLjgrLjgrnjg4gifQ.

HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8
Date: Sat, 13 May 2017 16:21:08 GMT
Content-Length: 49

{"error":"'none' signature type is not allowed"}
```
</div>
　エラーになりました。  

トークンの発行時に署名アルゴリズムに"none"が指定されたというのは、検証時には関係ありませんでした。  


<b>結論: 検証は検証時に使用する署名アルゴリズムに依存するようです。</b>
<b>(noneは指定するな。指定するための「UnsafeAllowNoneSignatureType」というワードはいかにも怪しいが・・・。)</b>  

おわり🏠  

<hr>

　次はもっとセキュリティ色の強い記事を書きたい...。

## 更新履歴

- 2017年5月14日 新規作成
