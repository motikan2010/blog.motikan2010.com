<figure class="figure-image figure-image-fotolife" title="NAXSIのロゴ">[f:id:motikan2010:20190604003128p:plain:alt=NAXSI-Logo]<figcaption>NAXSIのロゴ</figcaption></figure>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに
　NAXSI（Nginx Anti XSS & SQL Injection）はOSSのWAFであり、スター数が3,000を超える人気を誇っており、利用してみたくなったので、インストールしてみました。  
　ちなみにOSSのWAFとして有名なModSecurityのスター数は約2,800です。（NAXSIすごい...）
![](https://i.imgur.com/r37TBw1.png)


[https://github.com/nbs-system/naxsi:embed:cite]

## 環境
- CentOS 7.4
- Nginx 1.16.0
- NAXSI

<!-- more -->

## 導入作業
### Nginxインストール
　NAXSIはNginxのモジュールとして動作しますので、最初にYumでNginxをインストールします。  
バージョン「1.16.0」がインストールされました。
#### リポジトリ追加
```
# vim /etc/yum.repos.d/nginx.repo
-----
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

#### インストール
```
# yum -y install nginx
# nginx -v
nginx version: nginx/1.16.0
```

### NAXSIモジュールをビルド
　YumでインストールしたNginxと同じバージョンのNginxのソースを取得・展開します。  
```
# cd /usr/local/src/
# wget http://nginx.org/download/nginx-1.16.0.tar.gz
# tar xvfz nginx-1.16.0.tar.gz
```
　NAXSIをGitHubから取得し、モジュールをビルドします。
```
# git clone https://github.com/nbs-system/naxsi.git
# cd nginx-1.16.0
# ./configure --with-compat --add-dynamic-module=../naxsi/naxsi_src
# make modules
```
　共有モジュールが作成されたことが確認できます。
```log-text
# file objs/ngx_http_naxsi_module.so
objs/ngx_http_naxsi_module.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=0b1276f7ce6796d2388e5efeae83705f0bda9c52, not stripped
```

### NAXSIモジュールのロード
```
# cp objs/ngx_http_naxsi_module.so /etc/nginx/modules
# vim /etc/nginx/nginx.conf
----------
load_module modules/ngx_http_naxsi_module.so;
----------
# cp /usr/local/src/naxsi/naxsi_config/naxsi_core.rules /etc/nginx/
```

### ログファイルの作成
```
# touch /var/log/naxsi.log
# chown nginx /var/log/naxsi.log
```

### Nginx設定ファイル修正
```config
# vim /etc/nginx/conf.d/default.conf
----------
# ルールの読み込み
include /etc/nginx/naxsi_core.rules;

server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;

        # NAXSIの設定
        SecRulesEnabled;
        DeniedUrl "/50x.html";

        CheckRule "$SQL >= 8" BLOCK;
        CheckRule "$RFI >= 8" BLOCK;
        CheckRule "$TRAVERSAL >= 4" BLOCK;
        CheckRule "$EVADE >= 4" BLOCK;
        CheckRule "$XSS >= 8" BLOCK;
        
        # ログの出力先
        error_log /var/log/naxsi.log;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### Nginxの起動
```
# sudo systemctl start nginx
```

### 動作確認

#### 不正なリクエストを送信
　GETパラメータにタグを指定した場合にブロックされることが確認できました。  
ですが、デフォルトのルールでは誤検知（フォールスポジティブ）が多めなので、即本番環境への反映はしないほうがよさそうです。
```
http://3.112.xxx.xxx/?param=<script>
```
![](https://i.imgur.com/EYFjFSH.png)

#### 検出ログ
　ブロックしたログ詳細は、`error_log`ディレクティブで指定したファイルに書き込まれます。
```log-text
# tail -f /var/log/naxsi.log
2019/06/03 15:13:02 [error] 31644#31644: *1 NAXSI_FMT: ip=163.zzz.yyy.zzz&server=3.112.yyy.zzz&uri=/&vers=0.56&total_processed=2&total_blocked=2&config=block&cscore0=$XSS&score0=8&zone0=ARGS&id0=1302&var_name0=param, client: 163.xxx.yyy.zzz, server: localhost, request: "GET /?param=%3Cscript%3E HTTP/1.1", host: "3.112.yyy.zzz"
```

### ブロックはしたくないけど、検出はしたい
　`LearningMode`を設定するとブロックは無効化されますが、検出は行われログに詳細が出力されます。
```
# vim /etc/nginx/conf.d/default.conf
-----
SecRulesEnabled;
LearningMode; # 追加
```

###　参考

[https://github.com/nbs-system/naxsi/wiki:title]

