結論から言いますとGDライブラリはyumでインストールすることができなかったので、  
結局ソースからインストールしました。

## "phpbrew install"が失敗した
phpbrewでGDライブラリを入れようとしたら下記のようなエラーが表示された。
```
$ phpbrew install 5.6.26 +default +gd +openssl=/usr -- --with-libdir=lib64
*WARNING* You're runing phpbrew as root/sudo. Unless you're going to install
system-wide phpbrew or this might cause problems.
===> phpbrew will now build 5.6.26

(省略)

checking for gdSetErrorMethod in -lgd... no

configure: error: Unable to find libgd.(a|so) >= 2.1.0 anywhere under /usr

Please checkout the build log file for more details:
	 tail /root/.phpbrew/build/php-5.6.26/build.log
```
「Unable to find libgd.(a|so) >= 2.1.0 anywhere under」というエラーを見る限り、  
libgdのバージョン2.1.0以上が必要らしい。


<!-- more -->



## yumからlibgdをインストール

```
$ yum install gd
パッケージ gd-2.0.35-11.el6.x86_64 はインストール済みか最新バージョンです
何もしません
```
デフォルトのyumリポジトリでは、バージョン2.1.0以上を入れられない。

## ソースからlibgdをインストール
少し手間だがソースからインストールしてみる。
http://d.hatena.ne.jp/end0tknr/20150313/1426224469
バージョン2.1.1は古いと思われるが、2.1.0の用件は満たしているのでこのバージョンを入れることにする。  
ちなみに最新バージョンは、2.2.4です。(2017/2/6現在)
https://libgd.github.io/
```
$ wget https://bitbucket.org/libgd/gd-libgd/downloads/libgd-2.1.1.tar.gz 
$ tar -zxvf libgd-2.1.1.tar.gz 
$ cd libgd-2.1.1
$ ./configure --prefix=/usr/local/gd
$ make
$ make install
```

### libにシンボリックリンク
これでlibgdは"/usr/local/gd/"以下に配置されているので、
```
$ phpbrew install 5.6.26 +default +openssl=/usr +gd=/usr/local/gd -- --with-libdir=lib
```
として実行したのだが、今度はopensslがインストールされなかった。"--with-libdir=lib64"と指定しないといけなさそう。
なので
```
$ cd /usr/local/gd
$ ln -s lib lib64
```
を実行してlib64という名でlibにアクセスするようにする。

## phpbrewでGDライブラリをインストール
```
$ phpbrew install 5.6.26 +default +openssl=/usr +gd=/usr/local/gd -- --with-libdir=lib64
*WARNING* You're runing phpbrew as root/sudo. Unless you're going to install
system-wide phpbrew or this might cause problems.
===> phpbrew will now build 5.6.26

(省略)

---> Found date.timezone, patching config timezone with Asia/Tokyo
Congratulations! Now you have PHP with 5.6.26 as php-5.6.26
```
phpbrewでGDライブラリを入れることができた。

