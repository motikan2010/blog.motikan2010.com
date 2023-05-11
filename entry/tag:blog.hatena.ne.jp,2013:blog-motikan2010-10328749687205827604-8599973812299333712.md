<b><span style="color: #ff0000">2017/11/07 ※追記 こちらのやり方が簡単！！  GJ</span></b>

<div class="md-code" style="width:100%">
```
$ phpbrew ext install intl -- --with-icu-dir="$(brew --prefix icu4c)"
```
</div>

[https://qiita.com/sizuhiko/items/aebdbb05f0824cae4bdd:embed:cite]  
　
　上記の方法ではできなかったよ。という方が本記事の方法も試してみてもいいかと。  

---

## ことの始まり

　CakePHP3を入れたのだが、起動ができない。  
**intl**を有効にしないといけないらしい。

<div class="md-code" style="width:100%">
```
$ bin/cake server -p 8765

PHP Fatal error:  You must enable the intl extension to use CakePHP.
 in /Users/admin/PhpstormProjects/cake3app/config/requirements.php on line 31

Fatal error: You must enable the intl extension to use CakePHP.
 in /Users/admin/PhpstormProjects/cake3app/config/requirements.php on line 31
```
</div>

## 動作環境

||バージョン|
|-|-|
|macOS|10.12.6|
|PHPBrew|1.22.6|
|PHP|7.1.0|

## 問題発生

<div class="md-code" style="width:100%">
```
// インストール有無の確認
$ phpbrew ext | grep intl
 [ ] intl

// 拡張intlをインストール
$ phpbrew ext install intl

//・・・

Error: Command failed: /usr/bin/make -C '/Users/admin/.phpbrew/build/php-7.1.0/ext/intl' 'all'  >> '/Users/admin/.phpbrew/build/php-7.1.0/ext/intl/build.log' 2>&1 returns:
The last 5 lines in the log file:
/usr/local/Cellar/icu4c/59.1/include/unicode/unistr.h:3180:7: error: delegating constructors are permitted only in C++11

      UnicodeString(Char16Ptr(buffer), buffLength, buffCapacity) {}

      ^~~~~~~~~~~~~

2 warnings and 3 errors generated.

make: *** [intl_convertcpp.lo] Error 1
```
</div>

エラー\_(:3 」∠ )_

## 解決方法

　下記の記事を参考にした。  
[Install the PHP INTL extension on a Mac](https://gist.github.com/redefinelab/4188331)

<div class="md-code" style="width:100%">
```
$ cd /Users/admin/.phpbrew/build/php-7.1.0/ext/intl/
$ vim Makefile
CXXFLAGS = -g -O2
↓に変更（33行目付近："-std=c++11"を追記）
CXXFLAGS = -g -O2 -std=c++11

// ビルド
$ make
$ make install
Installing shared extensions:     /Users/admin/.phpbrew/php/php-7.1.0/lib/php/extensions/no-debug-non-zts-20160303/

// .soファイル生成の確認
$ ls -l /Users/admin/.phpbrew/php/php-7.1.0/lib/php/extensions/no-debug-non-zts-20160303/ | grep intl
-rwxr-xr-x  1 admin  staff   512764  9 19 01:16 intl.so

// "intl.ini"ファイルの作成と、１行記述
$ vim ~/.phpbrew/php/php-7.1.0/var/db/intl.ini
extension=intl.so
```
</div>

## intlインストールの確認

**intl**が有効になっていることを確認

<div class="md-code" style="width:100%">
```
$ phpbrew ext | grep intl
 [*] intl         1.1.0
```
</div>

## 動作確認

　CakePHP3も動作していることを確認できた。
<div class="md-code" style="width:100%">
```
$ bin/cake server -p 8765
Welcome to CakePHP v3.5.0 Console
```
</div>