<div style="text-align:center;">
[f:id:motikan2010:20200914142235p:plain]
</div>

## はじめに

　LaravelでRailsの"link_to"のようにHTMLを生成できないのかと思い、探してみたら「<b>Laravel Collective</b>」というものがありました。  

　導入方法などは下記を参照。

[https://laravelcollective.com/docs/5.3/html:title]

　HTML出力のイメージがしづらい部分があると思われるので、<span class="m-y">実際にHTMLを出力</span>させてみて、それをまとめてみました。

<div class="contents-box">
  <p>[:contents]</p>
</div>

<!-- more -->

## Formタグ

### 基本

#### Form::open(['url' => 'home'])

　自動的にCSRFトークンが追加されて出力されます。

<div class="md-code" style="width:100%">
```html
<form method="POST" action="http://localhost:3000/home" accept-charset="UTF-8">
<input name="_token" type="hidden" value="eAFowzR2efsBNQYrVzQ4VymRbYQB2afVyEDijzqd">
```
</div>

#### Form::close()

<div class="md-code" style="width:100%">
```html
</form>
```
</div>

### 送信先にルーティング名

#### Form::open(['route' => 'users.index'])

　action属性にルーティングを適用することができます。  
<div class="md-code" style="width:100%">
```html
<form method="POST" action="http://localhost:3000/users" accept-charset="UTF-8">
<input name="_token" type="hidden" value="eAFowzR2efsBNQYrVzQ4VymRbYQB2afVyEDijzqd">
```
</div>

### 送信先にURLパラメータ

#### Form::model($user, ['route' => ['users.update', $user->id]])

　ユーザIDといったURLパラメータをaction属性に含めることもできます。
<div class="md-code" style="width:100%">
```html
<form method="POST" action="http://localhost:3000/users/1" accept-charset="UTF-8">
<input name="_token" type="hidden" value="eAFowzR2efsBNQYrVzQ4VymRbYQB2afVyEDijzqd">
```
</div>

### PUTメソッド

#### Form::open(['url' => 'home', 'method' => 'put'])

　メソッドを区別するため「name="\_method"」属性を持っているinputタグが一緒に出力されます。
<div class="md-code" style="width:100%">
```html
<form method="POST" action="http://localhost:3000/home" accept-charset="UTF-8">
<input name="_method" type="hidden" value="PUT">
<input name="_token" type="hidden" value="eAFowzR2efsBNQYrVzQ4VymRbYQB2afVyEDijzqd">
```
</div>

### マルチパートフォーム

#### Form::open(['url' => 'home', 'files' => true])

　`enctype="multipart/form-data"` 属性が付与される点が特徴です。  
<div class="md-code" style="width:100%">
```html
<form method="POST" action="http://localhost:3000/home" accept-charset="UTF-8" enctype="multipart/form-data">
<input name="_token" type="hidden" value="eAFowzR2efsBNQYrVzQ4VymRbYQB2afVyEDijzqd">
```
</div>

## ラベル

#### Form::label('email', 'E-Mail Address')

<div class="md-code" style="width:100%">
```html
<label for="email">E-Mail Address</label>
```
</div>

## 入力フォーム

### テキスト

#### Form::text('username')

　第１引数にname属性の値を指定します。
<div class="md-code" style="width:100%">
```html
<input name="username" type="text">
```
</div>

#### Form::text('email', 'example@example.com')

　第２引数に初期値(value属性の値)を指定します。
<div class="md-code" style="width:100%">
```html
<input name="email" type="text" value="example@example.com" id="email">
```
</div>

### パスワード

#### Form::password('password', ['class' => 'awesome'])

<div class="md-code" style="width:100%">
```html
<input class="awesome" name="password" type="password" value="">
```
</div>

### メールアドレス

#### Form::email('mail'', $value = null, $attributes = [])

<div class="md-code" style="width:100%">
```html
<input name="mail" type="email">
```
</div>

### 数値

#### Form::number('name', 'value')

<div class="md-code" style="width:100%">
```html
<input name="name" type="number" value="value">
```
</div>

### 日付

#### Form::date('name', \Carbon\Carbon::now())

<div class="md-code" style="width:100%">
```html
<input name="name" type="date" value="2017-01-28">
```
</div>

### ファイル

#### Form::file('name', $attributes = [])

<div class="md-code" style="width:100%">
```html
<input name="name" type="file">
```
</div>

### チェックボックス

#### Form::checkbox('name', 'value')

<div class="md-code" style="width:100%">
```html
<input checked="checked" name="name" type="checkbox" value="value">
```
</div>

### ラジオボタン

#### Form::radio('name', 'value')

<div class="md-code" style="width:100%">
```html
<input name="name" type="radio" value="value">
```
</div>

## セレクトボックス

### 基本

#### Form::select('size', ['L' => 'Large', 'S' => 'Small'])

<div class="md-code" style="width:100%">
```html
<select name="size">
  <option value="L">Large</option>
  <option value="S">Small</option>
</select>
```
</div>

### 初期値を指定

#### Form::select('size', ['L' => 'Large', 'S' => 'Small'], 'S')

　第３引数に初期値を指定することができます。  
<div class="md-code" style="width:100%">
```html
<select name="size">
  <option value="L">Large</option>
  <option value="S" selected="selected">Small</option>
</select>
```
</div>

### 空の初期値を指定

#### Form::select('size', ['L' => 'Large', 'S' => 'Small'], null, ['placeholder' => 'Pick a size...'])

　空の初期値を設定することもできます。  
<div class="md-code" style="width:100%">
```html
<select name="size">
  <option selected="selected" value="">Pick a size...</option>
  <option value="L">Large</option>
  <option value="S">Small</option>
</select>
```
</div>

### multiple属性を付与

#### Form::select('size', ['L' => 'Large', 'S' => 'Small'], null, ['multiple' => true])

　multiple属性を付与するし、複数選択することができます。  
<div class="md-code" style="width:100%">
```html
<select multiple="1" name="size">
  <option value="L">Large</option>
  <option value="S">Small</option>
</select>
```
</div>

### 選択肢をグループ化

#### Form::select('animal', [ 'Cats' => ['leopard' => 'Leopard'], 'Dogs' => ['spaniel' => 'Spaniel'], ])

<div class="md-code" style="width:100%">
```html
<select name="animal">
  <optgroup label="Cats">
    <option value="leopard">Leopard</option>
  </optgroup>
  <optgroup label="Dogs">
    <option value="spaniel">Spaniel</option>
  </optgroup>
</select>
```
</div>

### 連続した値の選択肢

#### Form::selectRange('number', 10, 20)

　10 〜 20の値を指定することができます。  
<div class="md-code" style="width:100%">
```html
<select name="number">
  <option value="10">10</option>
  <option value="11">11</option>
  <option value="12">12</option>
  <option value="13">13</option>
  <option value="14">14</option>
  <option value="15">15</option>
  <option value="16">16</option>
  <option value="17">17</option>
  <option value="18">18</option>
  <option value="19">19</option>
  <option value="20">20</option>
</select>
```
</div>

### 月の選択肢

#### Form::selectMonth('month')

<div class="md-code" style="width:100%">
```html
<select name="month">
  <option value="1">January</option>
  <option value="2">February</option>
  <option value="3">March</option>
  <option value="4">April</option>
  <option value="5">May</option>
  <option value="6">June</option>
  <option value="7">July</option>
  <option value="8">August</option>
  <option value="9">September</option>
  <option value="10">October</option>
  <option value="11">November</option>
  <option value="12">December</option>
</select>
```
</div>

## ボタン

#### Form::submit('Click Me!')

<div class="md-code" style="width:100%">
```html
<input type="submit" value="Click Me!">
```
</div>

## オリジナルのフィールド

#### Form::myField()

　フィールドを作成し、任意の名前で定義するこができます。  
<div class="md-code" style="width:100%">
```php
Form::macro('myField', function()
{
    return '<input type="awesome">';
});
```
</div>

<div class="md-code" style="width:100%">
```html
<input type="awesome">
```
</div>

## リンク

### 基本

#### link_to('foo/bar', $title = null, $attributes = [], $secure = null)

<div class="md-code" style="width:100%">
```html
<a href="http://localhost:3000/foo/bar">http://localhost:3000/foo/bar</a>
```
</div>

### リンク文字列を指定

#### link_to('/home', 'Home', $attributes = [], $secure = null)

<div class="md-code" style="width:100%">
```html
<a href="http://localhost:3000/home">Home</a>
```
</div>

### リンク先をHTTPS

#### link_to('foo/bar', $title = null, $attributes = [], $secure = true)

<div class="md-code" style="width:100%">
```html
<a href="https://localhost:3000/foo/bar">https://localhost:3000/foo/bar</a>
```
</div>

#### link_to_asset('foo/bar.zip', $title = null, $attributes = [], $secure = null)

<div class="md-code" style="width:100%">
```html
<a href="http://localhost:3000/foo/bar.zip">http://localhost:3000/foo/bar.zips</a>
```
</div>

### リンク先にルーティング名を使用

#### link_to_route('login', $title = null, $parameters = [], $attributes = [])

<div class="md-code" style="width:100%">
```html
<a href="http://localhost:3000/login">http://localhost:3000/login</a>
```
</div>

### URLパラメータを付与

#### link_to_route('login', $title = null, $parameters = ['id' => $user->id, 'name' => $user->name], $attributes = [])

　"$parameters"引数に配列を与えることで、リンク先にURLパラメータを付与することができます。

<div class="md-code" style="width:100%">
```html
<a href="http://localhost:3000/login?id=1&name=motikan2010">http://localhost:3000/login?id=1&name=motikan2010</a>
```
</div>

### リンク先に"コントローラ@アクション"形式

#### link_to_action('SessionController@getLogin', $title = null, $parameters = [], $attributes = [])

<div class="md-code" style="width:100%">
```html
<a href="http://localhost:3000/login">http://localhost:3000/login</a>
```
</div>