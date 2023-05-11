<div style="text-align:center;">[f:id:motikan2010:20200318151440p:plain:w600]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　Nginx では「proxy_pass」ディレクティブを指定することでフォワードプロキシ(Forward Proxy)として動作させることが可能です。  


　しかし、モジュール等を追加していない<span class="m-y">素の Nginx の場合、 HTTPS 通信をフォワードプロキシすることはできません</span>。  
　後述する「**ngx_http_proxy_connect_module**」を利用して解決します。  

<div style="text-align:center;">
[f:id:motikan2010:20200318153058p:plain:w500]
</div>

<!-- more -->

　試しにNginxプロキシ経由でHTTPSのサイトにアクセスしたら以下のようにエラーとなりました。
<div class="md-code">
```
$ curl -Lv https://github.com/ -x 127.0.0.1:3128
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 3128 (#0)
* Establish HTTP proxy tunnel to github.com:443
> CONNECT github.com:443 HTTP/1.1
> Host: github.com:443
> User-Agent: curl/7.54.0
> Proxy-Connection: Keep-Alive
>
< HTTP/1.1 400 Bad Request
< Server: nginx/1.16.1
< Date: Wed, 18 Mar 2020 05:24:05 GMT
< Content-Type: text/html
< Content-Length: 157
< Connection: close
<
* Received HTTP code 400 from proxy after CONNECT
* Closing connection 0
curl: (56) Received HTTP code 400 from proxy after CONNECT
```
</div>

### HTTPS 通信をプロキシする「ngx_http_proxy_connect_module」モジュール

　<span class="m-y">「ngx_http_proxy_connect_module」モジュールを利用することで HTTPS サイトへのサクセスをプロキシ経由で行うことができます。</span>

[https://github.com/chobits/ngx_http_proxy_connect_module:embed:cite]

↓ このモジュールの説明
> This module provides support for the CONNECT method request. This method is mainly used to tunnel SSL requests through proxy servers.


## 環境構築

### Nginx と モジュール の ダウンロード & インストール

　<span class="m-y">configure 時に `--add-module=../ngx_http_proxy_connect_module` オプションを指定</span>することで、 Nginx に ngx_http_proxy_connect モジュールを組み込むことができます。  
　導入にあたって Nginx に対してパッチを当てる必要があるため、Nginxをコンパイルする必要があります。  
<div class="md-code">
```
# 作業用ディレクトリを作成
$ mkdir work && cd work

# Nginxの取得
$ wget http://nginx.org/download/nginx-1.16.1.tar.gz
$ tar zxvf nginx-1.16.1.tar.gz
$ rm -f nginx-1.16.1.tar.gz

# ngx_http_proxy_connect_moduleの取得（現時点の最新版は「0.0.1」）
$ git clone https://github.com/chobits/ngx_http_proxy_connect_module.git -b v0.0.1

# 作業用ディレクトリの内容は以下のようになっています
$ ls
nginx-1.16.1/
ngx_http_proxy_connect_module/

# Nginxに対してのパッチ適用 と Nginxのインストール
$ cd nginx-1.16.1
$ patch -p1 < ../ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_101504.patch
$ ./configure --prefix=/path/to/nginx --add-module=../ngx_http_proxy_connect_module
$ make && make install
```
</div>

### Nginx の設定 & 起動

<div class="md-code">
```
# Nginxのディレクトリに移動
$ cd /path/to/nginx

# 設定を以下のように修正します
$ vim conf/nginx.conf
---------------------------------------------
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

     server {
         listen                         3128;

         # dns resolver used by forward proxying
         resolver                       8.8.8.8;

         # forward proxy for CONNECT request
         proxy_connect;
         proxy_connect_allow            443 563;
         proxy_connect_connect_timeout  10s;
         proxy_connect_read_timeout     10s;
         proxy_connect_send_timeout     10s;

         # forward proxy for non-CONNECT request
         location / {
             proxy_pass $scheme://$http_host$request_uri;
             proxy_set_header Host $host;
         }
     }
}
---------------------------------------------

# Nginx の起動
$ ./sbin/nginx
```
</div>

## 動作確認

<div class="md-code">
```
$ curl -Lv https://github.com/ -x 127.0.0.1:3128
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 3128 (#0)
* Establish HTTP proxy tunnel to github.com:443
> CONNECT github.com:443 HTTP/1.1

< HTTP/1.1 200 Connection Established
< Proxy-agent: nginx

> GET / HTTP/1.1
> Host: github.com
> User-Agent: curl/7.54.0

< HTTP/1.1 200 OK
< date: Tue, 17 Mar 2020 09:25:15 GMT
```
</div>


## ダイナミック(動的)モジュール

　ngx_http_proxy_connect_module は動的モジュールとして `.so` ファイルを作成することが可能です。  
  
　しかし導入の際に Nginx に対してパッチを適用する必要があるため、すでにインストールされている Nginx にこのモジュールを導入することはできなさそうです。
  
　ダイナミックモジュールが動作しない件については、いくつか ISSUE があるようでした。  
そんな感じでダイナミックモジュールとして組み込むことは厳しそう。

## 参考

- [How to Use NGINX as an HTTPS Forward Proxy Server - Alibaba Cloud Community](https://www.alibabacloud.com/blog/how-to-use-nginx-as-an-https-forward-proxy-server_595799)

## 更新履歴

- 2020年3月18日 新規作成


[blog:g:12921228815726579926:banner]
