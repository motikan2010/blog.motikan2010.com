[f:id:motikan2010:20170421183808p:plain]

[https://github.com/nsarno/knock:embed:cite]



JSON Web Tokenの説明は下記の記事を参照。
[http://qiita.com/kaiinui/items/21ec7cc8a1130a1a103a:embed:cite]



<!-- more -->



[:contents]  

## 事前準備
### 新規アプリを生成

```
$ rails new railsJWT --api
```
### Gemfileに追記
必要なライブラリをGemfileに追記します。
```
$ vim Gemfile

# 下記を追記
gem "faker"
gem "bcrypt"
gem "jsonapi-resources"
gem "knock"

$ bundle install
```


[https://github.com/cerebris/jsonapi-resources:title]



## モデルの作成
### Postモデル
「タイトル」「内容」「公開/非公開の指定」のカラムを保持したPostモデルを作成します。
```
$ rails g model Post title:string body:text type:string
```

```
$ touch app/models/private_post.rb
$ touch app/models/public_post.rb
```

```ruby
$ vim app/models/private_post.rb

class PrivatePost < Post
end

$ vim app/models/public_post.rb

class PublicPost < Post
end
```

```ruby
$ vim app/models/post.rb

class Post < ApplicationRecord
  validates :body, presence: true
  validates :title, presence: true
  validates :type, presence: true

  POST_TYPES = %w(PublicPost PrivatePost)
  validates :type, :inclusion => { :in => POST_TYPES }
end
```
### Userモデル
次に「パスワード」「名前」「メールアドレス」のカラムを保持したUserモデルを作成します。
```
$ rails g model user password_digest:string name:string email:string
```
```ruby
$ vim app/models/user.rb

class User < ActiveRecord::Base
  has_secure_password

  validates :name, presence: true
  validates :email, presence: true
end
```
```
$ rails db:migrate
```

### テストデータの追加
```ruby
$ vim db/seeds.rb

Post.destroy_all
User.destroy_all

# ユーザを作成
User.create!({
  name: 'User1',
  email: 'user1@example.com',
  password: 'passwd1',
  password_confirmation: 'passwd1'
})

User.create!({
  name: 'User2',
  email: 'user2@example.com',
  password: 'passwd2',
  password_confirmation: 'passwd2'
})

3.times do
  # 公開記事を作成
  PublicPost.create!(
    title: Faker::Lorem.sentence,
    body: Faker::Lorem.paragraphs.join(' ')
  )

  # 非公開記事を作成
  PrivatePost.create!(
    title: Faker::Lorem.sentence,
    body: Faker::Lorem.paragraphs.join(' ')
  )
end
```
```
$ rails db:seed
```
## コントローラの作成
### PublicPostsコントローラ
```ruby
$ vim app/controllers/application_controller.rb

class ApplicationController < ActionController::API
  include Knock::Authenticable # 追記
end
```

```ruby
$ rails g controller PublicPosts
$ vim vim app/controllers/public_posts_controller.rb

class PublicPostsController < ApplicationController
  include JSONAPI::ActsAsResourceController # 追記
end
```

```ruby
$ rails generate jsonapi:resource public_posts
$ vim app/resources/public_post_resource.rb

class PublicPostResource < JSONAPI::Resource
  immutable
  attributes :title, :body
end
```
#### ルーティング設定
```
$ vim config/routes.rb

jsonapi_resources :public_posts # 追記
```

### PrivatePostsコントローラ
```
$ rails generate knock:install
```
```
$ rails generate knock:token_controller user
```
「**before_action :authenticate_user**」を追記することによって、認証が必要なコントローラにすることができます。
```ruby
$ rails g controller PrivatePosts
$ vim app/controllers/private_posts_controller.rb

class PrivatePostsController < ApplicationController
  include JSONAPI::ActsAsResourceController # 追記
  before_action :authenticate_user # 追記
end
```

```ruby
$ rails generate jsonapi:resource private_posts
$ vim app/resources/private_post_resource.rb

class PrivatePostResource < JSONAPI::Resource
  immutable
  attributes :title, :body
end
```
#### ルーティング設定
```ruby
$ vim config/routes.rb

jsonapi_resources :private_posts
```
## 動作確認
### リクエスト
#### "/public-posts"にアクセス
認証を行わずにアクセスすることが可能です。
```
$ curl -X "GET" "http://example.jp:3000/public-posts"

HTTP/1.1 200 OK
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: application/vnd.api+json
ETag: W/"ae93de1833f5e081219472e78b408c0a"
Cache-Control: max-age=0, private, must-revalidate
X-Request-Id: a0b4df38-9374-4801-97c3-714599305b00
X-Runtime: 0.010128
Transfer-Encoding: chunked

{"data":[{"id":"1","type":"public-posts","links":{"self":"http://example.jp:3000/public-posts/1"},"attributes":{"title":"Necessitatibus et sit alias.","body":"Numquam...（中略）..."}}]}%
```

#### "/private-posts"にアクセス
レスポンスで「**HTTP/1.1 401 Unauthorized**」と返ってきており、認証が必要ということが分かります。
```
$ curl -X "GET" "http://example.jp:3000/private-posts"

HTTP/1.1 401 Unauthorized
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: text/html
Cache-Control: no-cache
X-Request-Id: 7cde37bd-abdd-421f-a6dc-5667e8cce0d0
X-Runtime: 0.002747
Transfer-Encoding: chunked
```

### 認証を行う
"/private-posts"に対してアクセスを行うためには、認証後に発行されるトークンをリクエストに含める必要があります。
#### トークンを取得する認証リクエスト
```
$ curl -X "POST" "http://nuconuco.com:3000/user_token" \
> -H "Content-Type: application/json" \
> -d '{"auth": {"email": "user1@example.com", "password": "passwd1"}}'

{"jwt":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTI4MTY4MzYsInN1YiI6NX0.EzBo2BZatWc-80HAfioQYbL1gPH90tf9YV00yAnHBr8"}%
```
JSON形式で返ってきている「eyJ0eXAiOiJKV1QiL・・・」が認証トークンです。

#### トークンを使用してアクセス
Authorizationヘッダの値に取得したトークンを指定します。
```
$ curl -X "GET" "http://example.jp:3000/private-posts" \
> -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTI4MTY4MzYsInN1YiI6NX0.EzBo2BZatWc-80HAfioQYbL1gPH90tf9YV00yAnHBr8" \
> -H "Content-Type: application/json"

GET /private-posts HTTP/1.1
Host: example.jp:3000
User-Agent: curl/7.43.0
Accept: */*
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0OTI4MTY4MzYsInN1YiI6NX0.EzBo2BZatWc-80HAfioQYbL1gPH90tf9YV00yAnHBr8
Content-Type: application/json

HTTP/1.1 200 OK
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: application/vnd.api+json
ETag: W/"df8eaf13cb9cd4dd8f47df9f4ec65bb3"
Cache-Control: max-age=0, private, must-revalidate
X-Request-Id: 5050c781-bbfb-43a6-a7f7-2b887682d3a0
X-Runtime: 0.022478
Transfer-Encoding: chunked

{"data":[{"id":"2","type":"private-posts","links":{"self":"http://example.jp:3000/private-posts/2"},"attributes":{"title":"Qui voluptas nemo tenetur.","body":"Nemo...(中略)..."}}]}%
```
正常に"/private-posts"にアクセスすることができています。

これでJSON Web Tokenの実装が完了となります。