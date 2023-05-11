[f:id:motikan2010:20170421183524p:plain:w500]

前回の続きです。  
[http://motikan2010.hatenadiary.com/entry/2017/04/21/%E3%80%8EJSON_Web_Token%28JWT%29%E3%80%8F%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B%28%E5%AE%9F%E8%A3%85%E7%B7%A8%29:title]  
今回はJWTのセキュリティにふれてみます。

<div class="contents-box">
  <p>[:contents]</p>
</div>

<!-- more -->

  
## JWTは危険なのか

　認証成功時に発行されるJSONは軽く見たかんじ、少し長い乱数のセッションIDのように見えますがそうではありません。
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTI4MTY4MzYsInN1YiI6NX0.EzBo2BZatWc-80HAfioQYbL1gPH90tf9YV00yAnHBr8
```  
規則性があり、下記の記事のように危険がふくまれているそうです。

[https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/:embed:cite]



[http://christina04.hatenablog.com/entry/2016/06/07/123000:embed:cite]


発行されるJSONはこのような形式になっています。

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
eyJleHAiOjE0OTI4MTY4MzYsInN1YiI6NX0
EzBo2BZatWc-80HAfioQYbL1gPH90tf9YV00yAnHBr8
```
が発行された場合に、まずは「.」で区切る。  
個々の値をBase64でデコードします。これだけです。

||*base64デコード*|
|-|-|
|eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9|{"typ":"JWT","alg":"HS256"}|
|eyJleHAiOjE0OTI4NDY3MTUsInN1YiI6NX0|{"exp":1492816836,"sub":5}|
|EzBo2BZatWc-80HAfioQYbL1gPH90tf9YV00yAnHBr8|(署名バイナリデータ)|

### ユーザによって値が改ざんされる危険性
ここで重要なのが「**"sub":5**」の"5"という数値がユーザidということです。  
Webアプリケーション側では、この数値でユーザを識別されており、この値を他ユーザの値に改ざんして送信することによって、なりすましを行うことが可能となっています。  
本来は「EzBo2BZa・・・」の値が検証トークンとなっており、値が改ざんされたことを検出できるが、algの指定にnoneが用いられた時に検証されないとのことです。  
つまり、**「{"typ":"JWT","alg":"none"} 」の場合に、なりすましが行われてしまう**。  
そのことがJWTのセキュリティ上の懸念となっている。  

ここではknockのみに焦点を当てて、改ざんが検出されるか、またはされないかの確認をしていきます。

## knockはどうなのか
試しに「"sub":5」を「**sub":6**」に改ざんしてリクエストを送信してみます。
### アルゴリズムを「HS256」
まずは、knockのデフォルトアルゴリズムで確認します。
#### ユーザid：5で認証
```
$ curl -X "POST" "http://nuconuco.com:3000/user_token" \
-H "Content-Type: application/json" \
-d '{"auth": {"email": "user1@example.com", "password": "passwd1"}}'

{"jwt":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTI4NDc5OTAsInN1YiI6NX0.L_inYpObtUsQE_lEP_Kk2FNgP8888ppMICykuGa7AVQ"}%
```
||Base64デコード|
|-|-|
|eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9|{"typ":"JWT","alg":"HS256"} |
|eyJleHAiOjE0OTI4NDc5OTAsInN1YiI6NX0|{"exp":1492847990,"sub":5}|

#### 「"sub":6」に改ざんして送信
||Base64デコード|
|-|-|
|eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9|{"typ":"JWT","alg":"HS256"}|
|eyJleHAiOjE0OTI4NDc5OTAsInN1YiI6Nn0=|{"exp":1492847990,"sub":6}|
```
$ curl -X "GET" "http://example.jp:3000/private-posts" -v \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTI4NDc5OTAsInN1YiI6Nn0=.L_inYpObtUsQE_lEP_Kk2FNgP8888ppMICykuGa7AVQ" \
-H "Content-Type: application/json"


HTTP/1.1 401 Unauthorized
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: text/html
Cache-Control: no-cache
X-Request-Id: dc65a24d-e7c6-4cd7-abb2-0eaa27680f0a
X-Runtime: 0.003603
Transfer-Encoding: chunked
```
改ざんが検知されて、「**HTTP/1.1 401 Unauthorized**」となっている。
### アルゴリズムを「none」
algにnoneを指定すれば、署名であるトークンが発行されずに改ざんができるのかを確認します。
#### 設定ファイルの編集
```
$ vim config/initializers/knock.rb

# config.token_signature_algorithm = 'HS256'
config.token_signature_algorithm = 'none' # 32行付近に追記
```
これで「{typ: "JWT", alg: "none"}」となってくれるはずです。
#### ユーザid：5で認証
```
$ curl -X "POST" "http://nuconuco.com:3000/user_token" \
-H "Content-Type: application/json" \
-d '{"auth": {"email": "user1@example.com", "password": "passwd1"}}'

{"jwt":"eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJleHAiOjE0OTI4NDg2MzUsInN1YiI6NX0."}%
```
||Base64デコード|
|-|-|
|eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0|{"typ":"JWT","alg":"none"} |
|eyJleHAiOjE0OTI4NDg2MzUsInN1YiI6NX0|{"exp":1492848635,"sub":5}|

予想通り「{“typ”:“JWT”,“alg”:“none”}」になり、認証トークンも発行されていないことが分かります 。
この状態で「"sub":5」を改ざんするとなりすましが可能であるか確認してみます。
  
#### 「"sub":6」に改ざんして送信
||Base64デコード|
|-|-|
|eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0|{"typ":"JWT","alg":"none"} |
|eyJleHAiOjE0OTI4NDg2MzUsInN1YiI6Nn0=|{"exp":1492848635,"sub":6}|
```
$ curl -X "GET" "http://example.jp:3000/private-posts" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJleHAiOjE0OTI4NDg2MzUsInN1YiI6Nn0=." \
-H "Content-Type: application/json"

HTTP/1.1 401 Unauthorized
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: text/html
Cache-Control: no-cache
X-Request-Id: 55fda114-d9c3-435f-aa49-22816180bd97
X-Runtime: 0.004454
Transfer-Encoding: chunked
```
結果は「HTTP/1.1 401 Unauthorized」  
なんと「"alg":"none"」を指定しているがエラーになった。  
knockでは「"alg":"none"」に指定しても、認証トークンが検証されることが確認できた。  
でも何故なのか。  
  
---
knockのソースを見て原因を調べてみる。  
## なぜknockでは「"alg":"none"」で検証が行われたのか
JWTの実装を確認してみる。

### ruby-jwtの仕様を確認
knockは内部でruby-jwtを利用しているので、まずはruby-jwtのREADMEを見てみます。  
[https://github.com/jwt/ruby-jwt:embed:cite]

を見てみると、
```
decoded_token = JWT.decode token, nil, false
```
JSONのでコード時つまりトークンの検証時に、第３引数に検証有無を指定する必要があるらしく、上記のように**false**が指定されていると検証が行われません。 

### knockの実装を確認
[https://github.com/nsarno/knock/blob/7fb00e36b8a1db188d2258eb28dbc56441385302/app/model/knock/auth_token.rb#L10:title]   
```ruby
# 10行付近
@payload, _ = JWT.decode token, decode_key, true, options.merge(verify_options)
```
knockだと**true**が指定されており、検証が必須となっている。
そのため「"alg":"none"」となっていても検証が行われていたわけです。

### knockでトークン検証なしにしてみる
ソースコードを少し変更してみて、トークンの検証が行われないようにしてみます。  
試しに第３引数を"false"に変更して、動作を確認してみる。
```ruby
$ vim vendor/bundler/ruby/2.3.0/gems/knock-2.1.1/app/model/knock/auth_token.rb

@payload, _ = JWT.decode token, decode_key, false, options.merge(verify_options) # 変更
```
再度改ざんしたリクエストを送信してみる。認証者をレスポンスで返すようにする。
```ruby
$ vim app/controllers/private_posts_controller.rb

class PrivatePostsController < ApplicationController
  include JSONAPI::ActsAsResourceController
  before_action :authenticate_user

  def index
    render :json => current_user
  end

end
```
```
curl -X "GET" "http://example.jp:3000/private-posts" -v \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJleHAiOjE0OTI4NDg2MzUsInN1YiI6Nn0=." \
-H "Content-Type: application/json"

HTTP/1.1 200 OK
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: application/json; charset=utf-8
ETag: W/"f7e6dd29a636a3d858fab2dd8c67e57d"
Cache-Control: max-age=0, private, must-revalidate
X-Request-Id: e4fdf9a4-49e7-4bff-be60-ae8351dc4483
X-Runtime: 0.004916
Transfer-Encoding: chunked

{"id":6,"password_digest":"$2a$10$2IL4VyZ2m2ojfjrWpMxiDOTL6Ctu43cm2o6423z7xCET71HtZVSRC","name":"User2","email":"user2@example.com","created_at":"2017-04-20T18:46:10.245Z","updated_at":"2017-04-20T18:46:10.245Z"}%
```
改ざんしたユーザidの情報を取得できていることが確認できる。  

## 結論

　knockの場合だと、変にknock自体のソースコードを変更しない限り、**トークンの検証は必ず行われる**と考えられます。  
　今回使用したJWTライブラリでは改ざんが検出されましたが、他のライブラリではどのようになっているのか。  
設定次第では署名の検証を無効にできる等のライブラリがあるのかなどを探していきます。

