<div class="contents-box">
  <p>[:contents]</p>
</div>

### はじめに

　<b>Secure属性を付与したCookie</b>の取り扱いなど簡単な検証で重宝しています。  

Python製のWebフレームワークである **bottle** を使用すると、複雑な開発環境・動作環境を用意せずにHTTPS通信を実現するWebサーバを作成することが可能です。  

### HTTPS通信に使用するサーバ証明書の作成

<div class="md-code" style="width:100%">
```
# openssl req -new -x509 -keyout server.pem -out server.pem -days 365 -nodes
```
</div>

<!-- more -->


### アプリケーションの準備

#### 依存Pythonライブラリのインストール

<div class="md-code" style="width:100%">
```
$ pip install bottle gevent
```
</div>

#### ソースコード

　ここではWebフレームワークの**bottle**を利用しています。

<div class="md-code" style="width:100%">
```python
import sys, os, datetime
from bottle import route, run, request, HTTPResponse, ServerAdapter
from gevent.pywsgi import WSGIServer

class SSLWebServer(ServerAdapter):

    def run(self, handler):
        srv = WSGIServer((self.host, self.port), handler,
                certfile='./server.pem',
                keyfile='./server.pem')
        srv.serve_forever()

@route('/', method='GET')
def index():
    body = "0"
    if request.get_cookie("counter"):
        body = str(int(request.get_cookie("counter")) + 1)
    res = HTTPResponse(status=200, body=body)
    res.set_cookie('counter', body, secure=1)
    return res

run(host='127.0.0.1', port=8080, server=SSLWebServer)
```
</div>

### 動作確認

####ブラウザ

[f:id:motikan2010:20161225194304p:plain:w300]

#### curlコマンド

`-k`オプション(証明書エラーを無視)でHTTPS通信を行います。  

<div class="md-code" style="width:100%">
```
# curl -v -k https://127.0.0.1:8080/
* Trying 127.0.0.1...
* Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
* TLS 1.2 connection using TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
* Server certificate: Internet Widgits Pty Ltd
> GET / HTTP/1.1
> Host: 127.0.0.1:8080
> User-Agent: curl/7.43.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Length: 1
< Content-Type: text/html; charset=UTF-8
< Set-Cookie: counter=0; Secure
< Date: Sun, 25 Dec 2016 10:31:46 GMT
<
* Connection #0 to host 127.0.0.1 left intact
0
```
</div>

### 参考

[http://dgtool.treitos.com/2011/12/ssl-encryption-in-python-bottle.html:title]

### 更新履歴

- 2016年12月25日 新規作成
- 2019年10月3日 体裁修正
