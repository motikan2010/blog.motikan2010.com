<div style="text-align: center;">[f:id:motikan2010:20170124011356p:plain]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　PHP製のWebフレームワークである『Laravel』を使って"認証あり"のアプリケーションを作成していきます。  
今回はLaravelバージョン5.3系を使います。

## 構築



### プロジェクトの作成
<div class="sm-code">
```
$ laravel new auth_scaffold
$ cd auth_scaffold
$ php artisan --version
Laravel Framework version 5.3.29
```
</div>

### Scaffoldの導入

[https://github.com/laralib/l5scaffold:embed:cite]


下記のコマンドで容易に導入することができます。
```
$ composer require 'laralib/l5scaffold' --dev
```

<!-- more -->

### サービスプロバイダーの追加

"config/app.php"内の"providers"に対して、「`Laralib\L5scaffold\GeneratorsServiceProvider::class`」を追加します。  


```php
$ vim config/app.php
ーーーーーーーーーーーーーーーーーーーー
<?php
(省略)
    'providers' => [

        /*
         * Laravel Framework Service Providers...
         */
        Illuminate\Auth\AuthServiceProvider::class,
        Illuminate\Broadcasting\BroadcastServiceProvider::class,
(省略)
        Laralib\L5scaffold\GeneratorsServiceProvider::class,
    ],
(以下省略)
```
上記の追加が完了したら、Scaffoldの準備完了です。  

### scaffoldの実行

　下記のコマンドで コントローラ・モデル・ビュー の作成が完了します。  
相変わらずscaffold便利すぎ！
```
$ php artisan make:scaffold Tweet --schema="title:string:default('Tweet #1'), body:text"

Configuring Tweet...
Migration created successfully
Seed created successfully.
Model created successfully.
Controller created successfully.
Layout created successfully.
Error created successfully.
Views created successfully.
Dump-autoload...
Route::resource("tweets","TweetController"); // Add this line in routes.php
```
予定通りに作成されたのかを確認してみます。
```
// コントローラ
$ ls app/Http/Controllers/TweetController.php
app/Http/Controllers/TweetController.php

// モデル
$ ls app/Tweet.php
app/Tweet.php

// ビュー
$ ls resources/views/tweets/
create.blade.php  edit.blade.php  index.blade.php  show.blade.php
```
しっかりと作成されているようです。  
相変わらず早い！！  

### ルーティングの設定
作成されたコントローラを呼び出せないと意味がありませんので、
ルーティングの設定を行っていきます。  
"routes/web.php"の末尾に「Route::resource("tweets","TweetController");」を記述します。  
$ vim routes/web.php
```php
<?php
(省略)

Route::get('/', function () {
    return view('welcome');
});

Route::resource("tweets","TweetController"); // 追記
```
ルーティングの設定が反映されているか確認してみます。
```
$ php artisan route:list

+--------+-----------+---------------------+----------------+----------------------------------------------+--------------+
| Domain | Method    | URI                 | Name           | Action                                       | Middleware   |
+--------+-----------+---------------------+----------------+----------------------------------------------+--------------+
|        | GET|HEAD  | /                   |                | Closure                                      | web          |
|        | GET|HEAD  | api/user            |                | Closure                                      | api,auth:api |
|        | POST      | tweets              | tweets.store   | App\Http\Controllers\TweetController@store   | web          |
|        | GET|HEAD  | tweets              | tweets.index   | App\Http\Controllers\TweetController@index   | web          |
|        | GET|HEAD  | tweets/create       | tweets.create  | App\Http\Controllers\TweetController@create  | web          |
|        | DELETE    | tweets/{tweet}      | tweets.destroy | App\Http\Controllers\TweetController@destroy | web          |
|        | PUT|PATCH | tweets/{tweet}      | tweets.update  | App\Http\Controllers\TweetController@update  | web          |
|        | GET|HEAD  | tweets/{tweet}      | tweets.show    | App\Http\Controllers\TweetController@show    | web          |
|        | GET|HEAD  | tweets/{tweet}/edit | tweets.edit    | App\Http\Controllers\TweetController@edit    | web          |
+--------+-----------+---------------------+----------------+----------------------------------------------+--------------+
```

### データベースの作成
ここの説明は簡単に流します。  
データベースの名前は「scaffold_app」にしておきます。
mysql> create database scaffold_app;
データベースの設定は下記のファイルに行います。
- ユーザ名：laravelusr
- パスワード：himitsu  
にしています。  
$ vim .env
```
(省略)

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=scaffold_app
DB_USERNAME=laravelusr
DB_PASSWORD=himitsu

(省略)
```
上記設定が完了したらテーブルの作成を行います。

### マイグレーション
下記のコマンドを実行することによって、テーブルが作成されます。  
ここでは、tweetsテーブル以外にも作成されていますが、無視しておきます。
```
$ php artisan migrate

Migration table created successfully.
Migrated: 2014_10_12_000000_create_users_table
Migrated: 2014_10_12_100000_create_password_resets_table
Migrated: 2017_01_23_143819_create_tweets_table
```
### Webアプリケーションへアクセス

これで、Webアプリケーションとして利用することが可能になりました。 
```
$ php artisan serve --host 0.0.0.0 --port 3000
```
"http://127.0.0.1:3000/tweets"にアクセス
[f:id:motikan2010:20170124004456p:plain]  
上記のような画面が表示されたら成功です。  
試しになにか投稿してみましょう。「Create」を押して投稿画面へ遷移できます。  
[f:id:motikan2010:20170124004459p:plain]  
投稿することができました。  
「View」を押すことで詳細画面へ遷移することもできます。  

[f:id:motikan2010:20170124004502p:plain]

これはこれで良いアプリケーションとなっていますが、これから認証機能を実装してきます。

### 認証機能(Auth)の導入
#### 魔法の呪文『make:auth』
下記のコマンドを実行します。  
$ php artisan make:auth  
認証に必要な コントローラ・モデル・ビュー の作成が完了します。  
Scaffold同様、作成されているかを確認します。
```
// コントローラ
$ ls app/Http/Controllers/Auth/
ForgotPasswordController.php  LoginController.php  RegisterController.php  ResetPasswordController.php

// モデル
$ ls app/User.php
app/User.php

// ビュー
$ ls resources/views/auth/
login.blade.php     passwords/      register.blade.php
```
作成されているのが確認できます。  
次にルーティングの設定ですが、Authの場合は自動的に記述されます。  
念のために確認してみます。  
$ cat routes/web.php
```php
<?php
(省略)

Route::get('/', function () {
    return view('welcome');
});

Route::resource("tweets","TweetController");

Auth::routes();    // 自動的に追加されている

Route::get('/home', 'HomeController@index');
```
ルーティングが反映されているか改めて確認してみます。
```
$ php artisan route:list

+--------+-----------+------------------------+----------------+------------------------------------------------------------------------+--------------+
| Domain | Method    | URI                    | Name           | Action                                                                 | Middleware   |
+--------+-----------+------------------------+----------------+------------------------------------------------------------------------+--------------+
|        | GET|HEAD  | /                      |                | Closure                                                                | web          |
|        | GET|HEAD  | api/user               |                | Closure                                                                | api,auth:api |
|        | GET|HEAD  | home                   |                | App\Http\Controllers\HomeController@index                              | web,auth     |
|        | POST      | login                  |                | App\Http\Controllers\Auth\LoginController@login                        | web,guest    |
|        | GET|HEAD  | login                  | login          | App\Http\Controllers\Auth\LoginController@showLoginForm                | web,guest    |
|        | POST      | logout                 | logout         | App\Http\Controllers\Auth\LoginController@logout                       | web          |
|        | POST      | password/email         |                | App\Http\Controllers\Auth\ForgotPasswordController@sendResetLinkEmail  | web,guest    |
|        | GET|HEAD  | password/reset         |                | App\Http\Controllers\Auth\ForgotPasswordController@showLinkRequestForm | web,guest    |
|        | POST      | password/reset         |                | App\Http\Controllers\Auth\ResetPasswordController@reset                | web,guest    |
|        | GET|HEAD  | password/reset/{token} |                | App\Http\Controllers\Auth\ResetPasswordController@showResetForm        | web,guest    |
|        | POST      | register               |                | App\Http\Controllers\Auth\RegisterController@register                  | web,guest    |
|        | GET|HEAD  | register               | register       | App\Http\Controllers\Auth\RegisterController@showRegistrationForm      | web,guest    |
|        | GET|HEAD  | tweets                 | tweets.index   | App\Http\Controllers\TweetController@index                             | web          |
(省略)
|        | GET|HEAD  | tweets/{tweet}/edit    | tweets.edit    | App\Http\Controllers\TweetController@edit                              | web          |
+--------+-----------+------------------------+----------------+------------------------------------------------------------------------+--------------+
```
問題なく認証機能へアクセスできそうです。  
#### 認証の確認
認証を行っていないユーザはログイン画面へ遷移させるようにしましょう。
$ vim app/Http/Controllers/TweetController.php
```php
<?php namespace App\Http\Controllers;

use App\Http\Requests;
use App\Http\Controllers\Controller;

use App\Tweet;
use Illuminate\Http\Request;

class TweetController extends Controller {

        public function __construct(){
            $this->middleware('auth');
        }
(省略)
```
これで認証を行っていないユーザはTweetControllerのアクションへアクセスできないようになります。

#### 認証後の遷移先を変更
実は認証に必要なアカウントの「登録後」や「認証後」には"/home"へ遷移されるように、Authのコントローラで設定されています。  "/home"には特になにもアプリケーションがありませんので、そこへ遷移されても楽しくありません。
　事前に「登録後」や「認証後」には"/tweets"へ遷移されるように設定します。  
行う作業は非常に簡単です。  
下記２つのファイルを修正するだけです。  
$ vim app/Http/Controllers/Auth/RegisterController.php
```php
<?php

namespace App\Http\Controllers\Auth;

use App\User;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Validator;
use Illuminate\Foundation\Auth\RegistersUsers;

class RegisterController extends Controller
{

(省略)

    protected $redirectTo = '/tweets'; // "home" を "tweets" に変更
(省略)
```

$ vim app/Http/Controllers/Auth/LoginController.php
```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\AuthenticatesUsers;

class LoginController extends Controller
{

(省略)
    
    protected $redirectTo = '/tweets'; // "home" を "tweets" に変更
(省略)
```
  
これで完成です。

## アプリケーションを確認

下記のコマンドでアプリケーションを起動しましょう。  
```
$ php artisan serve --host 0.0.0.0 --port 3000
```
  
「http://127.0.0.1:3000/」にアクセスすると下記のような画面が表示されます。  
[f:id:motikan2010:20170124004509p:plain]  
ためしに「http://127.0.0.1:3000/tweets」に直接アクセスしてみましょう。  
当然ながら認証を行っていませんので、ログイン画面へ遷移されます[f:id:motikan2010:20170124004523p:plain]  
まだ、ユーザは存在していないので「Register」からユーザの作成を行ってみます。
[f:id:motikan2010:20170124004517p:plain]  
ユーザの作成が完了すると、認証処理が行われTweets画面へ遷移されます。
[f:id:motikan2010:20170124004526p:plain]
  
　こんな短時間でこのようなアプリケーションが作成できるとは驚きです。  

今回は誰でもがアカウントを作成することができ、認証後のページにアクセスすることが可能となっていますが、コードをいじってみることで自分だけがアクセスできるサイトを作れそうですね。 