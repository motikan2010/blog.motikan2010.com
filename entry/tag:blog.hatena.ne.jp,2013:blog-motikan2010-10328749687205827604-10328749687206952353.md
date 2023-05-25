<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　Webサーバのベンチマークツールである"weighttp"のインストール・動作確認までを行っていきます。  
私の環境ではPythonのバージョン3がデフォルトして動作しており、それが原因でつまずいた点がありましたので、記載しておきます。

[https://github.com/lighttpd/weighttp:embed:cite]

## 導入

### configure 実施

<div class="sm-code">
```
$ yum install -y --enablerepo=epel libev libev-devel
$ wget http://github.com/lighttpd/weighttp/zipball/master
$ unzip master
$ cd lighttpd-weighttp-f680bec/
```
</div>

<!-- more -->

#### configure 実行（失敗: Python3で動かしたことが原因）

<div class="sm-code">
```
$ ./waf configure
Checking for program gcc,cc              : ok /usr/bin/gcc
Checking for program cpp                 : ok /usr/bin/cpp
Checking for program ar                  : ok /usr/bin/ar
Checking for program ranlib              : ok /usr/bin/ranlib
Checking for gcc                         : ok
Checking for library ev                  : ok
Traceback (most recent call last):
  File "/tmp/weighttp-master/.waf3-1.5.9-d1e0349fc8937631a656fb8ea7e99063/wafadmin/Utils.py", line 414, in recurse
    txt=readf(base+'_'+name,m='rU')
  File "/tmp/weighttp-master/.waf3-1.5.9-d1e0349fc8937631a656fb8ea7e99063/wafadmin/Utils.py", line 379, in readf
    f=open(fname,m)
FileNotFoundError: [Errno 2] No such file or directory: '/tmp/weighttp-master/wscript_configure'
```
</div>

　"FileNotFoundError"が発生している。下のサイトで解決することができた。
バージョン2のPythonを使用すればいいらしい。


[https://bbs.archlinux.org/viewtopic.php?id=118160#p928077:title]

#### configure 実行（2度目の失敗: ev.hが見つからなかった）

<div class="sm-code">
```
$ python2 ./waf configure

Checking for program gcc,cc              : ok /usr/bin/gcc
Checking for program cpp                 : ok /usr/bin/cpp
Checking for program ar                  : ok /usr/bin/ar
Checking for program ranlib              : ok /usr/bin/ranlib
Checking for gcc                         : ok
Checking for library ev                  : ok
Checking for header ev.h                 : not found
 error: the configuration failed (see '/tmp/weighttp-master/build/config.log')
```
</div>

「**`Checking for header ev.h                 : not found`**」と出力されているので、ヘッダファイルを移動させます。  

#### ヘッダファイル"ev.h"を移動

<div class="sm-code">
```
$ cp /usr/include/libev/ev.h /usr/include/
```
</div>

#### configure 実行(成功!!)

<div class="sm-code">
```
$ python2 ./waf configure

Checking for program gcc,cc              : ok /usr/bin/gcc
Checking for program cpp                 : ok /usr/bin/cpp
Checking for program ar                  : ok /usr/bin/ar
Checking for program ranlib              : ok /usr/bin/ranlib
Checking for gcc                         : ok
Checking for library ev                  : ok
Checking for header ev.h                 : ok
Checking for library pthread             : ok
Checking for header pthread.h            : ok
Checking for header unistd.h             : ok
Checking for header stdint.h             : ok
Checking for header fcntl.h              : ok
Checking for header inttypes.h           : ok
'configure' finished successfully (0.481s)
```
</div>

### ビルド

<div class="sm-code">
```
$ python2 ./waf build

Waf: Entering directory `/tmp/weighttp-master/build'
[1/4] cc: src/client.c -> build/default/src/client_1.o
[2/4] cc: src/weighttp.c -> build/default/src/weighttp_1.o
[3/4] cc: src/worker.c -> build/default/src/worker_1.o
[4/4] cc_link: build/default/src/client_1.o build/default/src/weighttp_1.o build/default/src/worker_1.o -> build/default/weighttp
Waf: Leaving directory `/tmp/weighttp-master/build'
'build' finished successfully (0.754s)
```
</div>

### インストール

<div class="sm-code">
```
$ python2 ./waf install

Waf: Entering directory `/tmp/weighttp-master/build'
* installing build/default/weighttp as /usr/local/bin/weighttp
Waf: Leaving directory `/tmp/weighttp-master/build'
'install' finished successfully (0.007s)
```
</div>

## 「weighttp」コマンドの動作確認

　これで `weighttp`コマンドを利用することができるようになりました。  
試しにヘルプコマンドを実行してみます。

<div class="sm-code">
```
$ weighttp --help
weighttp 0.4 - a lightweight and simple webserver benchmarking tool

error: unkown option: --

weighttp <options> <url>
  -n num   number of requests    (mandatory)
  -t num   threadcount           (default: 1)
  -c num   concurrent clients    (default: 1)
  -k       keep alive            (default: no)
  -6       use ipv6              (default: no)
  -H str   add header to request
  -h       show help and exit
  -v       show version and exit

example: weighttpd -n 10000 -c 10 -t 2 -k -H "User-Agent: foo" localhost/index.html
```
</div>

## 「weighttp」コマンドでベンチマーク取得

　helpコマンドで出力されたコマンド例通りに動かしてみます。

<div class="sm-code">
```
$ weighttp -n 10000 -c 10 -t 2 -k -H "User-Agent: foo" localhost/index.php

weighttp 0.4 - a lightweight and simple webserver benchmarking tool

starting benchmark...
spawning thread #1: 5 concurrent requests, 5000 total requests
spawning thread #2: 5 concurrent requests, 5000 total requests
progress:  10% done
progress:  20% done
progress:  30% done
progress:  40% done
progress:  50% done
progress:  60% done
progress:  70% done
progress:  80% done
progress:  90% done
progress: 100% done

finished in 5 sec, 34 millisec and 237 microsec, 1986 req/s, 387 kbyte/s
requests: 10000 total, 10000 started, 10000 done, 10000 succeeded, 0 failed, 0 errored
status codes: 10000 2xx, 0 3xx, 0 4xx, 0 5xx
traffic: 2000000 bytes total, 1920000 bytes http, 80000 bytes data
```
</div>


　Webサーバログにも通信ログが残っておりしっかりとベンチマークを取得できていそう。  
今までのは"Apache Bench"を使っていましたが、"weighttp"が代替ツールになるかこれからいろいろ検証していく予定。


