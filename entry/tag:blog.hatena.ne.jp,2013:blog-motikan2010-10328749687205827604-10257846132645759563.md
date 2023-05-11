<div style="text-align: center;">[f:id:motikan2010:20201130182936p:plain:w500]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

今更ながら<b>LDAPインジェクション</b>がどのようなものなのかの検証をやってみました。

　LDAPインジェクションは脆弱性としてそこそこ有名であり、名前だけは目にすることがあるが、イマイチ実際に検証を行う気になれない脆弱性でもあると思う。特にLDAPの環境構築は手間になりそうだし。  
  
　このままだとLDAPインジェクションを体験しないまま死んでしまってもおかしくないので、DockerでさくっとLDAPインジェクションの検証環境を構築し体験してみるとする。

## 検証環境

### ホスト

- Docker 18.06.1

### コンテナ

- OpenLDAP 2.4.44
  - [osixia/openldap - Docker Hub](https://hub.docker.com/r/osixia/openldap/)
- PHP 7.0 (php-ldap)

<!-- more -->


## 構築

### OpenLDAP

　特に設定ファイルなどを書き換えることなく、下記のコマンドでLDAPサーバの起動までを行ってくれる。

<div class="md-code" style="width:100%">
```
$ docker run -p 389:389 --name openldap-container --detach osixia/openldap:1.2.2
```
</div>

[https://github.com/osixia/docker-openldap:embed:cite]

### LDAPクライアントアプリケーション

　認証を行うクライアントアプリのソースは下記のリポジトリにあります。

[https://github.com/motikan2010/LDAP-Injection-Vuln-App:embed:cite]

#### 1. アプリケーションの用意

　ソースコード全体
[https://github.com/motikan2010/LDAP-Injection-Vuln-App/blob/20181004/src/public/index.php:title]

　以下がソースコードの抜粋です。  
LDAPに関係している部分のみを書き出しています。

<div class="md-code" style="width:100%">
```php
/**
 * src/public/index.php
 */

<?php

//LDAPの接続情報
const LDAP_HOST = "openldap-container";
const LDAP_PORT = 389;
const LDAP_DC = "dc=example,dc=org";
const LDAP_DN = "cn=admin,dc=example,dc=org";
const LDAP_PASS = "admin";

// 省略

// LDAPに接続
$ldapConn = ldap_connect(LDAP_HOST, LDAP_PORT);
if (!$ldapConn) {
    exit('ldap_conn');
}

// バインド
ldap_set_option($ldapConn, LDAP_OPT_PROTOCOL_VERSION, 3); // バージョンをOpenLDAPの方に合わせる
$ldapBind = ldap_bind($ldapConn, LDAP_DN,LDAP_PASS);
if ($ldapBind) {

    // ログイン処理
    // 「$userId」と「$password」はユーザの入力値が格納されます。
    $filter = '(&(cn=' . $userId . ')(userPassword=' . $password . '))'; // IDとパスワードのAND条件でフィルタを作成
    $ldapSearch = ldap_search($ldapConn, LDAP_DC, $filter);
    $getEntries = ldap_get_entries($ldapConn, $ldapSearch);
    if ($getEntries['count'] > 0) {
        // 成功
    }
} else {
    // 失敗
}
?>

// 以下省略
```
</div>

　ユーザの入力値をそのままフィルタに指定しているのが脆弱性となっています。

<div class="md-code" style="width:100%">
```php
$filter = '(&(cn=' . $userId . ')(userPassword=' . $password . '))';
```
</div>

#### 2. コンテナの準備

<div class="md-code" style="width:100%">
```
# Dockerfile

FROM php:7.0-apache

RUN \
    apt-get update && \
    apt-get install libldap2-dev -y && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ && \
    docker-php-ext-install ldap

ADD ./src/public /var/www/html/
```
</div>

#### 3. コンテナの実行

<div class="md-code" style="width:100%">
```
$ docker build -t ldap-client-container .
$ docker run --link openldap-container -p 8888:80 ldap-client-container
```
</div>

## 動作確認

### OpenLDAPの動作確認

　`-w`オプションでパスワードを指定していますが、「admin」がパスワードとなっています。  
末尾に記述されている「cn=admin」はフィルタであり、cn(Common Name)が「admin」のアカウントを表示しています。

<div class="md-code" style="width:100%">
```
$ ldapsearch -x -H ldap://localhost -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin 'cn=admin'
# extended LDIF
#
# LDAPv3
# base <dc=example,dc=org> with scope subtree
# filter: cn=admin
# requesting: ALL
#

# admin, example.org
dn: cn=admin,dc=example,dc=org
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword:: e1NTSEF9Z0RjWGl1QkR0d2xDcEZ5bVE4QWtoN09iRU1IZFVPN0s=

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```
</div>

### 予想外の事態発生

　「cn(Common Name)」と「userPassword」で認証する予定でしたが、userPasswordはハッシュ化されているらしい。

<div class="md-code" style="width:100%">
```
$ echo "e1NTSEF9Z0RjWGl1QkR0d2xDcEZ5bVE4QWtoN09iRU1IZFVPN0s=" | base64 -D ; echo
{SSHA}gDcXiuBDtwlCpFymQ8Akh7ObEMHdUO7K
```
</div>

　つまり、「(&(cn=admin)(userPassword=admin))」でフィルタした場合には、adminは表示されない。  

<div class="md-code" style="width:100%">
```
$ ldapsearch -x -H ldap://localhost -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin '(&(cn=admin)(userPassword=admin))'
# extended LDIF
#
# LDAPv3
# base <dc=example,dc=org> with scope subtree
# filter: (&(cn=admin)(userPassword=admin))
# requesting: ALL
#

# search result
search: 2
result: 0 Success

# numResponses: 1
```
</div>

　 だが、「`(&(cn=admin)(userPassword={SSHA}gDcXiuBDtwlCpFymQ8Akh7ObEMHdUO7K))`」でフィルタした場合には、adminを表示することができた。
 LDAPクライアント側からもこの文字列を入力する必要がありそう。


## 脆弱性の検証

### 正常系の動作確認

　まずはアプリケーションとして正常系の動作確認を実施してみます。  

1. 正しいパスワード入力  
<img src="https://i.imgur.com/jJxgCvj.png" width="400px">

2. 認証できた  
<img src="https://i.imgur.com/JQIKizX.png" width="400px">

　もちろんパスワードに別の文字列を入力した場合には、認証することはできない。というのがアプリケーションの正しい挙動なのだが、パスワードに「`*`」を入力してみる。

### 脆弱性の確認

1. 「*」をパスワードに入力  
<img src="https://i.imgur.com/KVHjgHF.png" width="400px">

2. 認証が成功した(本来は失敗しなくてはいけない)  
<img src="https://i.imgur.com/7SBSQvj.png" width="400px">

というのが、LDAPインジェクション。

OpenLDAP側のログを見てみると以下のようにフィルタが行われていた。  
入力した通りパスワードにワイルドカード「`*`」が指定されており、認証が成功していまうことが確認できる。

<div class="md-code" style="width:100%">
```
5bb61de0 conn=1009 op=1 SRCH base="dc=example,dc=org" scope=2 deref=0 filter="(&(cn=admin)(userPassword=*))"
```
</div>

## ダメな対策

　ユーザから入力されたパスワード内の「*」を削除すればワイルドカードが指定されることがなくなる。  
具体的にはフィルタの部分を以下の内容に修正する。

<div class="md-code" style="width:100%">
```
$filter = '(&(cn=' . $userId . ')(userPassword=' . str_replace('*', '', $password) . '))';
```
</div>

　これで再度アプリを動かしてみると、パスワードに「*」では認証が成功することはなくなった。  
だが、今度はログインIDとパスワードに以下の文字列を入力してみる。  

- ログインID：`admin)(|(cn=admin`
- パスワード：`hoge)`

<img src="https://i.imgur.com/zORx90d.png" width="400px">  

　表示上少しおかしいが、デタラメなパスワードでログインすることができた。  
<img src="https://i.imgur.com/zoCOxZS.png" width="400px">

　LDAPのログでは下記のようになっていた。

<div class="md-code" style="width:100%">
```
5bb62412 conn=1020 op=1 SRCH base="dc=example,dc=org" scope=2 deref=0 filter="(&(cn=admin)(|(cn=admin)(userPassword=hoge)))"
```
</div>

## 対策 ldap_escape関数

　`ldap_escape`関数を利用することで、入力文字列を無力化することが可能です。  
> ldap_escape — LDAP フィルタまたは DN で使われる文字列をエスケープする

　<span><a href="https://www.php.net/manual/ja/function.ldap-escape.php" target="_blank">ldap_escape関数</a></span>を利用することにより、どちらのパターンでも認証を突破することはできなくなりました。  
　以下、ldap_escape関数を用いたコード。

<div class="sm-code" style="width:100%">
```
$filter = '(&(cn=' . ldap_escape($userId) . ')(userPassword=' . ldap_escape($password) . '))';
```
</div>

## まとめ

　環境構築が面倒と思っていたLDAPですが、Dockerを利用したら1コマンドで構築できたというのが、1番の収穫。  

　LDAPインジェクションを手元て検証してみて分かったのですが、LDAPにはパスワードがハッシュ値で格納されており、脆弱性のサンプルのようにフィルタで認証している実装というのはあまりなさそう。だから世に出ている情報も少ないんですかね。  

　それとも昔のOpenLDAP or 別のLDAPでは平文でパスワードが格納される設定になっていたんですかね。それか今回利用したDockerイメージがそのような設定がなされてたいのか。  

　まだまだLDAPに関して分からないことだらけですので、また機会を見つけて学習しないと。。

## 参考

- <span><a href="https://bazuo.hatenadiary.org/entry/20100930/1285831559" target="_blank">LDAP認証してみた。 - ばずなダイアリー</a></span>
- <span><a href="https://www.exchangecore.com/blog/using-ldap-active-directory-authentication-php" target="_blank">Using LDAP Active Directory Authentication with PHP :: ExchangeCore</a></span>
- <span><a href="https://gitlab.langton.cloud/forks/SecLists/blob/4d6d6dcee29930587b26cee84fcd90e13eb38510/Fuzzing/LDAP_FUZZ.txt" target="_blank">Fuzzing/LDAP_FUZZ.txt · 4d6d6dcee29930587b26cee84fcd90e13eb38510 · forks / SecLists · GitLab</a></span>

## 更新履歴

- 2018年10月4日 新規作成
