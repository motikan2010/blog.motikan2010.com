<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに
　Cookieの属性に「Domain」「path」「Max-Age」「Expires」「Secure」「HttpOnly」があるが、  『<span class="m-y">SameSite</span>』というのも存在していることを知っていたか!?　私は最近まで知らなかった<s>（恥ずかしい）</s>。  
  
　2016年から存在しており現在、主要なブラウザではサポートされているらしい。  

<b>--- 追記 2020/2/4  ---</b>  
　2020年2月リリース予定の Chrome 80 からはSameSite属性の値がないCookieは`SameSite=Lax`が付与されたものとして扱われるとのこと。  

　SameSite属性を無効にしたい場合は明示的に`SameSite=None`を設定する必要があります。  
[https://developers-jp.googleblog.com/2019/11/cookie-samesitenone-secure.html:title]

　LaravelのデフォルトではSameSite属性を付与しないようになっているため、SameSite属性を無効にする場合は「`none`」を明示的に指定する必要があります。  
<b>----- ここまで -----</b>


 そんな認知度が低そうな属性をLaravelで設定できるのかを確認してみる。

## TL;DR
- Laravel 5.5.0 からサポート
- config/session.php を編集することでSameSite属性の値(Strict / Lax)の設定可能

<!-- more -->

## 検証環境
- PHP 7.1.3
- Laravel 5.7.23

## SameSite属性とは

　簡単に説明すると<span class="m-y">遷移元が外部ドメインのリクエストの場合はCookieが送信されないように制御できる</span>というもの。  

　これにより<b>ほとんどの</b>CSRF攻撃を防ぐことが可能になる。  
  
そもそもSameSite属性を聞いたことないという人向け画像
[f:id:motikan2010:20190130231232p:plain:w600]  
<span style="color: #ff0000">※Cookieが送信される条件は、SameSite属性の値(Strict/Lax)によって異なります。</span>

## SameSite属性が付与されるように設定

### 設定変更

#### 設定前に発行されるCookie情報

　設定前に発行されるCookie情報は下記のようになっており、SameSite属性はありません。  
<span class="m-y">デフォルトでは無効</span>となっているようです。
```
Set-Cookie: laravel_session=eyJpdiI6Ik...;
expires=Wed, 30-Jan-2019 00:11:04 GMT; 
Max-Age=7200; 
path=/; 
httponly
```

#### 対応Laravelバージョンについて

　Laravelのリリースノートを確認してみますと、<span class="m-y">バージョン 5.5.0 からSameSite属性をサポートしている</span>そうです。
> - Added SameSite support to CookieJar

[https://github.com/laravel/framework/releases/tag/v5.5.0:title]  

#### 設定ファイルの編集

　SameSite属性の付与の設定は「 **config/session.php** 」ファイルから行うことができる。  
　デフォルトでは"null"が設定されており、SameSite属性を付与しないようになっていることが確認できます。  
値に「`none`」を指定することも可能です。

```
(省略)

    /*
    |--------------------------------------------------------------------------
    | Same-Site Cookies
    |--------------------------------------------------------------------------
    |
    | This option determines how your cookies behave when cross-site requests
    | take place, and can be used to mitigate CSRF attacks. By default, we
    | do not enable this as other CSRF protection services are in place.
    |
    | Supported: "lax", "strict"
    |
    */

    'same_site' => null,
```

### 確認
#### 設定後に発行されるCookie情報

　発行されるCookieにSameSite属性が付与されるようになりました。  
「<span class="m-y">samesite=lax</span>」の属性が付与されています。
```
Set-Cookie: laravel_session=eyJpdiI6Im...; 
expires=Wed, 30-Jan-2019 01:35:33 GMT; 
Max-Age=7200; 
path=/; 
httponly; 
samesite=lax
```

## まとめ

　Spring Boot 2.xからCookieが発行される際に、Lax値が設定されたSameSite属性が付与されるのがデフォルトになり、1.5.xから移行した際にファーストパーティCookieが送信されないという事象に遭遇しました。そこでSameSite属性の存在を知らなかった為、解決に時間が掛かったという経緯があり、本記事を書きました。  

　今後、LaravelでもデフォルトでSameSite属性が設定されることも考えられますね。主要なブラウザでは既にサポートされるようになっているらしいので。  

[https://caniuse.com/#feat=same-site-cookie-attribute:title]  

## 参考
[https://tools.ietf.org/html/draft-west-first-party-cookies-07:title]

[https://blog.ssrf.in/post/samesite-cookie/:title]

## 更新履歴
- 2019年1月30日 新規作成
- 2020年2月4日 Chrome 80について追記
