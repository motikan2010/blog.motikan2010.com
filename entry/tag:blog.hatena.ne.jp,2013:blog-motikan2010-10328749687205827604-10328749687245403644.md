<div style="text-align:center;">[f:id:motikan2010:20170512014516p:plain:w500]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　前回、RailsのJWTライブラリである"knock"を使って署名アルゴリズムにNoneを使えるのかを試してみた(使えなかった)ので、今回はGo言語のJWTライブラリである"**jwt-go**"を使って、署名アルゴリズムに「SHA256」と「none」を試した(使えた)ことを書いていきます。  
[http://motikan2010.hatenadiary.com/entry/2017/04/21/JSON_Web_Token%28JWT%29%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B%E3%80%90%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E7%B7%A8%E3%80%91:embed:cite]  

[https://github.com/dgrijalva/jwt-go:embed:cite]

<!-- more -->

　下記のソースコードをベースにしていろいろ試してみます。  
要認証URLでは、トークン内のデータ文字列をクライアントに返すようにしています。ここでは「ゲスト」という文字列で固定しています。  

　今回はginを使って記述しています。

`main.go`
<div class="md-code" style="width:100%">
```go
package main

import (
	"fmt"
	"time"

	jwt "github.com/dgrijalva/jwt-go"
	"github.com/dgrijalva/jwt-go/request"
	"github.com/gin-gonic/gin"
)

var secretKey = "75c92a074c341e9964329c0550c2673730ed8479c885c43122c90a2843177d5ef21cb50cfadcccb20aeb730487c11e09ee4dbbb02387242ef264e74cbee97213"

func main() {
	r := gin.Default()

	r.GET("/api/", func(c *gin.Context) {
		/*
		   アルゴリズムの指定
		*/
		token := jwt.New(jwt.GetSigningMethod("HS256"))
		
		token.Claims = jwt.MapClaims{
			"user": "ゲスト",
			"exp":  time.Now().Add(time.Hour * 1).Unix(),
		}

		/*
		   トークンに対して署名の付与
		*/
		tokenString, err := token.SignedString([]byte(secretKey))
		if err == nil {
			c.JSON(200, gin.H{"token": tokenString})
		} else {
			c.JSON(500, gin.H{"message": "Could not generate token"})
		}
	})

	r.GET("/api/private/", func(c *gin.Context) {
		/*
		   署名の検証
		*/
		token, err := request.ParseFromRequest(c.Request, request.OAuth2Extractor, func(token *jwt.Token) (interface{}, error) {
			b := []byte(secretKey)
			return b, nil
		})

		if err == nil {
			claims := token.Claims.(jwt.MapClaims)
			msg := fmt.Sprintf("こんにちは、「 %s 」さん", claims["user"])
			c.JSON(200, gin.H{"message": msg})
		} else {
			c.JSON(401, gin.H{"error": fmt.Sprint(err)})
		}
	})

	r.Run(":8080")
}
```
</div>

　実際に署名アルゴリズムに「HS256」と「none」を指定した動作を確認していきます。

## 動作確認

### サンプルなどでよく見られる"HS256"を使用

[f:id:motikan2010:20170512014541j:plain]  

#### ①トークン必要URLにアクセス

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/private/
GET /api/private/ HTTP/1.1
Host: example.jp:8080

HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8
Date: Thu, 11 May 2017 14:09:56 GMT
Content-Length: 40

{"error":"no token present in request"}
```
</div>

#### ②トークンを取得

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/
GET /api/ HTTP/1.1
Host: example.jp:8080

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Thu, 11 May 2017 13:51:09 GMT
Content-Length: 144

{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTQ1MTQyNjksInVzZXIiOiLjgrLjgrnjg4gifQ.jI9Sc22DdF3EclflQZyqxuR1mGjj3YcqljsBo2IRn4Y"}
```
</div>

||base64デコード|
|-|-|
|eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9|{"alg":"HS256","typ":"JWT"}|
|eyJleHAiOjE0OTQ1MTQyNjksInVzZXIiOiLjgrLjgrnjg4gifQ|{"exp":1494514269,"user":"ゲスト"}|
|jI9Sc22DdF3EclflQZyqxuR1mGjj3YcqljsBo2IRn4Y|署名(シグネチャ)|

本来であれば「ゲスト」という部分にユーザIDなどの識別子が格納され、ユーザを識別するために使用されます。

#### ③発行されたトークンを送信

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/private/ -H "Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTQ1MTQyNjksInVzZXIiOiLjgrLjgrnjg4gifQ.jI9Sc22DdF3EclflQZyqxuR1mGjj3YcqljsBo2IRn4Y"
GET /api/private/ HTTP/1.1
Host: example.jp:8080
User-Agent: curl/7.43.0
Accept: */*
Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTQ1MTQyNjksInVzZXIiOiLjgrLjgrnjg4gifQ.jI9Sc22DdF3EclflQZyqxuR1mGjj3YcqljsBo2IRn4Y

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Thu, 11 May 2017 13:53:30 GMT
Content-Length: 56

{"message":"こんにちは、「 ゲスト 」さん"}
```
</div>

　「ゲスト」を「管理者」という文字列に改ざんしてリクエストを送信したらどうなるでしょうか？

||base64エンコード|
|-|-|
|{"exp":1494514269,"user":"管理者"}|eyJleHAiOjE0OTQ1MTQyNjksInVzZXIiOiLnrqHnkIbogIUifQo=|

[f:id:motikan2010:20170512014603j:plain]  

#### ④トークンを改ざんして送信

<div class="md-code" style="width:100%">
```
curl -v http://example.jp:8080/api/private/ -H "Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTQ1MTQyNjksInVzZXIiOiLnrqHnkIbogIUifQo=.jI9Sc22DdF3EclflQZyqxuR1mGjj3YcqljsBo2IRn4Y"
GET /api/private/ HTTP/1.1
Host: example.jp:8080
Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTQ1MTQyNjksInVzZXIiOiLnrqHnkIbogIUifQo=.jI9Sc22DdF3EclflQZyqxuR1mGjj3YcqljsBo2IRn4Y

HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8
Date: Thu, 11 May 2017 14:03:53 GMT
Content-Length: 33

{"error":"signature is invalid"}
```
</div>

　署名の検証が行われ、結果的に改ざんされたことが検出されました。  
ステータスコードも「401 Unauthorized」となっていることが確認できます。  


### 危険と言われている署名の検証なし"None"を使用

　jwt-goではアルゴリズムの指定に「none」を指定することが可能になっています。  
「none」を指してトークンを発行することにより、トークンの検証を行われずに改ざんが行われたトークンが受け入れられるようになります。  
[f:id:motikan2010:20170514012635j:plain]  


「none」を指定するには、**jwt.UnsafeAllowNoneSignatureType**を指定します。
https://github.com/dgrijalva/jwt-go#user-content-compliance  
上記ソースコード内の３箇所を編集します。以下のようになります。

<div class="md-code" style="width:100%">
```go
/*
   アルゴリズムの指定
*/
//token := jwt.New(jwt.GetSigningMethod("HS256"))
token := jwt.New(jwt.GetSigningMethod("none")) // https://github.com/dgrijalva/jwt-go/pull/79/files
```
</div>

<div class="md-code" style="width:100%">
```go
/*
  トークンに対して署名の付与
*/
//tokenString, err := token.SignedString([]byte(secretKey))
tokenString, err := token.SignedString(jwt.UnsafeAllowNoneSignatureType)
```
</div>

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

#### ①トークン必要URLにアクセス

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/private/
GET /api/private/ HTTP/1.1
Host: example.jp:8080

HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8
Date: Thu, 11 May 2017 14:34:36 GMT
Content-Length: 40

{"error":"no token present in request"}
```
</div>

#### ②トークンを取得

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/
GET /api/ HTTP/1.1
Host: example.jp:8080

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Thu, 11 May 2017 14:37:51 GMT
Content-Length: 100

{"token":"eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0k.eyJleHAiOjE0OTQ1MTcwNzEsInVzZXIiOiLjgrLjgrnjg4gifQ."}
```
</div>

||base64デコード|
|-|-|
|eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0k|{"alg":"none","typ":"JWT}|
|eyJleHAiOjE0OTQ1MTcwNzEsInVzZXIiOiLjgrLjgrnjg4gifQ|{"exp":1494517071,"user":"ゲスト"}|
|※署名なし|–|

ここで受け取ったトークンをそのまま返しても正常にレスポンスが返ってくるのは確実なので、早速改ざんしたデータを送ってみます。  
  
ここで「ゲスト」を「管理者」という文字列に改ざんしてリクエストを送信したらどうなるでしょうか？
この時に、トークンが発行された時のように、シグネチャはリクエストに含めません。  

||base64エンコード|
|-|-|
|{"exp":1494517071,"user":"管理者"}|eyJleHAiOjE0OTQ1MTcwNzEsInVzZXIiOiLnrqHnkIbogIUifQ==|

#### ③トークンを改ざんして送信

<div class="md-code" style="width:100%">
```
$ curl -v http://example.jp:8080/api/private/ -H "Authorization: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0k.eyJleHAiOjE0OTQ1MTcwNzEsInVzZXIiOiLnrqHnkIbogIUifQ==."
GET /api/private/ HTTP/1.1
Host: example.jp:8080
Authorization: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0k.eyJleHAiOjE0OTQ1MTcwNzEsInVzZXIiOiLnrqHnkIbogIUifQ==.

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Thu, 11 May 2017 14:44:57 GMT
Content-Length: 56

{"message":"こんにちは、「 管理者 」さん"}
```
</div>

　レスポンス内に「{"message":"こんにちは、「 管理者 」さん"}」が表示され、改ざんしたリクエストが正常処理されてしまったことが確認できます。
このような動作をすることから「**UnsafeAllowNoneSignatureType**」の指定が必要となります。

<b>↓ 続き ↓</b>

　署名アルゴリズムを改ざんして送信したときの挙動を確認しています。

[https://blog.motikan2010.com/entry/2017/05/14/jwt-go%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B_-_%E3%81%9D%E3%81%AE2:embed:cite]


## 更新履歴

- 2017年5月12日 新規作成
