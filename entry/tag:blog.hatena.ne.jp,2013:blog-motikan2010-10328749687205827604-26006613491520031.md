<div style="text-align: center;">
[f:id:motikan2010:20191231225911p:plain]
</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　2019年も終わりに近づいているので、2019年に公表されたCVEが採番されている脆弱性の中から人気なものをGitHubから検索しTOP 10を決めていきます。  

検索結果のリポジトリについては下記のリポジトリで管理されています。  
[https://github.com/nomi-sec/PoC-in-GitHub:embed:cite]  


## TOP 10

　TOP 10の算出方法としては、「リポジトリの数」と「それらのリポジトリのスターの数」を使用しています。  
リポジトリ毎に10ポイント、スター毎に1ポイント 加算される形式で算出しています。  
　※各脆弱性のタイトルはJVNから引用しています。

### 10位 501 ポイント『Pulse Secure Pulse Connect Secure におけるパーミッションに関する脆弱性(CVE-2019-11510)』

[https://jp.tenable.com/blog/cve-2018-13379-cve-2019-11510-fortigate-and-pulse-connect-secure-vulnerabilities-exploited-in:embed:cite]  

> 攻撃者はユーザー名とプレーンテキストパスワードを取得するためにCVE-2019-11510を使用して脆弱性のあるシステムを探し出しています。

とある通り、任意のファイルが取得できる脆弱性となっています。

Twitter社にも該当する脆弱性があったようですね。
[https://piyolog.hatenadiary.jp/entry/2019/09/09/070000:title]  

こちらの脆弱性はBlack Hat 2019でも触れられていた脆弱性でした。
[https://i.blackhat.com/USA-19/Wednesday/us-19-Tsai-Infiltrating-Corporate-Intranet-Like-NSA.pdf]

人気リポジトリ    
[https://github.com/projectzeroindia/CVE-2019-11510:title]  

<!-- more -->

### 9位 522 ポイント『Canonical snapd における入力確認に関する脆弱性(CVE-2019-7304)』

[https://ameblo.jp/miyou55mane/entry-12439917523.html:embed:cite]

人気リポジトリ  
[https://github.com/initstring/dirty_sock:title]

### 8位 545 ポイント『Firefox および Thunderbird における入力確認に関する脆弱性(CVE-2019-11708)』

[https://github.com/0vercl0k/CVE-2019-11708:embed:cite]

### 7位 572 ポイント『Android 用 ES File Explorer File Manager アプリケーションにおける入力確認に関する脆弱性(CVE-2019-6447)』

[f:id:motikan2010:20191231213243p:plain]

脆弱性の詳細  
[https://medium.com/@knownsec404team/analysis-of-es-file-explorer-security-vulnerability-cve-2019-6447-7f34407ed566:embed:cite]

人気リポジトリ  
[https://github.com/fs0c131y/ESFileExplorerOpenPortVuln:title]  


### 6位 629 ポイント『Espressif ESP-IDF および ESP8266_NONOS_SDK における入力確認に関する脆弱性(CVE-2019-12586)』

<div style="text-align: center;">
[f:id:motikan2010:20191231213350p:plain]
</div>

[https://matheus-garbelini.github.io/home/post/esp32-esp8266-eap-crash/:embed:cite]  

> ESP-IDF is the official development framework for the ESP32 chip.

　ESPデバイス上で動作するプログラムを書く際に用いられるフレームワーク(ESP-IDF)で見つかった脆弱性っぽいですね。  
　攻撃者の無線範囲内のESPデバイスをクラッシュさせることが可能な脆弱性らしいです。  
　珍しくハードウェアによりな脆弱性がランクインしました。  

ESP-IDFのドキュメント  
[https://docs.espressif.com/projects/esp-idf/en/latest/:title]  

人気リポジトリ  
[https://github.com/Matheus-Garbelini/esp32_esp8266_attacks:title]  

### 5位 704 ポイント『Oracle Fusion Middleware の Oracle WebLogic Server における WLS Core Components に関する脆弱性(CVE-2019-2618)』

　攻撃するために認証情報(メールアドレス & パスワード)は必要であり、任意のファイルがアップロードをすることが可能です。  
さらに、スクリプトを記述したファイルをアップロードすることで任意のコマンドが実行可能です。  

[https://github.com/jas502n/cve-2019-2618:embed:cite]

人気リポジトリ  
[https://github.com/dr0op/WeblogicScan:title]

### 4位 785 ポイント『Docker および runc におけるコンテナエラーの脆弱性(CVE-2019-5736)』

　悪意あるコンテナを実行することで、ホスト上でroot権限でコマンドが実行される可能性がある脆弱性です。  
普段からDocker Hub等の外部からコンテナイメージを利用する方は今使っているバージョンに気を付けたほうがいいでしょう。

[https://security.sios.com/vulnerability/runc-security-vulnerability-20190211.html:embed:cite]

人気リポジトリ  
[https://github.com/Frichetten/CVE-2019-5736-PoC:title]

### 3位 935 ポイント『Oracle Fusion Middleware の Oracle WebLogic Server における Web Services に関する脆弱性(CVE-2019-2725)』

　認証不要で任意のコマンドが実行できるという脆弱性です。  
攻撃通信は観測されており警視庁が注意喚起のレポートを公開するほど目立っているようです。  

[https://www.ipa.go.jp/security/ciadr/vul/20190428_WebLogicServer.html:embed:cite]

警視庁が出したレポート  
[https://www.npa.go.jp/cyberpolice/important/2019/201906211.html:title]

攻撃通信が既に観測されており、その通信の詳細内容。
[https://wizsafe.iij.ad.jp/2019/05/658/:title]

人気リポジトリ  
[https://github.com/lufeirider/CVE-2019-2725:title]


### 2位 1868 ポイント『PHP における境界外書き込みに関する脆弱性(CVE-2019-11043)』
 
<div style="text-align: center;">
[f:id:motikan2010:20191231222943p:plain]
</div>

[http://www.intellilink.co.jp/article/vulner/191106.html:embed:cite]  

　実際に攻撃に悪用されており、既に攻撃通信が観測されているとのことです。  
[http://www.security-next.com/110032:title]

人気リポジトリ  
[https://github.com/neex/phuip-fpizdam:title]

### 1位 5413 ポイント『複数の Microsoft Windows 製品のリモートデスクトップ サービスにおけるリモートでコードを実行される脆弱性(CVE-2019-0708 通称「BlueKeep」)』

<div style="text-align: center;">
[f:id:motikan2010:20191231213328p:plain]
</div>

　 1位はダントツの「BlueKeep」でした。  
 Windowsのセキュリティに詳しくない私でも名前ぐらいは知っているほどです。Wikipediaのページも作成されているようでした(ロゴも存在している！！)。

[https://blog.trendmicro.co.jp/archives/21466:embed:cite]  

Wikipediaのページ
[https://en.wikipedia.org/wiki/BlueKeep:title]

人気リポジトリ  
[https://github.com/zerosum0x0/CVE-2019-0708:title]

### TOP 10 一覧

|| スコア | CVE ID | リポ数| スター合計 |
| - | - | - | - | - | - |
|  1位 | 5413 | CVE-2019-0708 / BlueKeep | 111 | 4303 |
|  2位 | 1868 | CVE-2019-11043 / PHP-FPM |  16 | 1708 |
|  3位 |  935 | CVE-2019-2725 / Oracle WebLogic |  17 |  765 |
|  4位 |  785 | CVE-2019-5736 / Docker & runc |  19 |  595 |
|  5位 |  704 | CVE-2019-2618 / Oracle WebLogic |   6 |  644 |
|  6位 |  629 | CVE-2019-12586 / Espressif ESP-IDF |   1 |  619 |
|  7位 |  572 | CVE-2019-6447 / ES File Explorer File Manager |   1 |  562 |
|  8位 |  545 | CVE-2019-11708 / Firefox & Thunderbird |   1 |  535 |
|  9位 |  522 | CVE-2019-7304 / Canonical snapd |   2 |  502 |
| 10位 |  501 | CVE-2019-11510 / Pulse Connect Secure(VPN) |   9 |  411 |

## おまけ

### リポジトリ数ランキング

　2019年下期に公表された「sudo」「WhatsApp」などの脆弱性がランクインしました。  

| リポ数 | CVE ID | 概要(上のTOP 10に入っているものは「-」表記) |
| - | - | - |
| 111 | CVE-2019-0708  | - |
|  19 | CVE-2019-5736  | - |
|  17 | CVE-2019-2725  | - |
|  16 | CVE-2019-11043 | - |
|  14 | CVE-2019-3396  | Atlassian Confluence Server におけるパストラバーサルの脆弱性 |
|  11 | CVE-2019-14287 | Sudo における入力確認に関する脆弱性 |
|  10 | CVE-2019-11932 | WhatsApp および libpl_droidsonroids_gif における二重解放に関する脆弱性 |
|   9 | CVE-2019-11510 | - |
|   9 | CVE-2019-10149 | Exim における入力確認に関する脆弱性 |
|   8 | CVE-2019-5418  | Action View(Rails) における情報漏えいに関する脆弱性 |

## 更新履歴

- 2019年12月31日 新規作成
