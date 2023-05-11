<div style="text-align: center;">[f:id:motikan2010:20200106223239p:plain]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　2020年 1発目の投稿となりますが、時代に逆行して2018年に人気だった脆弱性を紹介します。  
先日書いた「2019年 人気脆弱性 TOP 10 in GitHub」の2018年版です。  

<b>↓ 2019年版 ↓</b>  
[https://blog.motikan2010.com/entry/2019/12/31/2019%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub:embed:cite]

## TOP 10

　TOP 10の算出方法としては、「リポジトリの数」と「それらのリポジトリのスターの数」を使用しています。  
リポジトリ毎に10ポイント、スター毎に1ポイント 加算される形式で算出しています。  
　※各脆弱性のタイトルはJVNから引用しています。  

### 10位 541 ポイント 『Intel Core マイクロプロセッサを搭載したシステムにおける情報漏えいに関する脆弱性(CVE-2018-3620 通称「Foreshadow」)』

[https://access.redhat.com/ja/security/cve/cve-2018-3620:embed:cite]

人気リポジトリ  
[https://github.com/ionescu007/SpecuCheck:title]

<!-- more -->


### 9位 575 ポイント 『Intel ハードウェアアーキテクチャのデバッグ例外を適切に処理していない問題(CVE-2018-8897)』

[https://qiita.com/IK_PE/items/e53e0cd4ba1d01f18d20:embed:cite]  

[Arbitrary Code Execution at Ring 0 using CVE-2018-8897 – Can.ac](https://blog.can.ac/2018/05/11/arbitrary-code-execution-at-ring-0-using-cve-2018-8897/)

人気リポジトリ  
[https://github.com/can1357/CVE-2018-8897:title]

### 8位 582 ポイント 『WinRAR における入力確認に関する脆弱性(CVE-2018-20250)』

[https://jp.tenable.com/blog/winrar-absolute-path-traversal-vulnerability-leads-to-remote-code-execution-cve-2018-20250-0:embed:cite]

人気リポジトリ  
[https://github.com/WyAtu/CVE-2018-20250:title]

### 7位 592 ポイント 『TBK DVR4104 および DVR4216 デバイスにおける証明書・パスワードの管理に関する脆弱性(CVE-2018-9995)』

下画像のようにIPカメラの映像が勝手に見られてしまう可能性が・・・。  
<div style="text-align: center;">[f:id:motikan2010:20200106223309p:plain:w400]</div>

　珍しくIPカメラ・レコーダーの製品の脆弱性です。  
悪用することでカメラの映像を不正に見ることができるとのこと。  
　脆弱性だけに着目したら認証回避ですが、その結果不正にカメラの映像が見られる点がこの脆弱性が注目されている理由だと考えられます。  

[https://www.bleepingcomputer.com/news/security/new-hacking-tool-lets-users-access-a-bunch-of-dvrs-and-their-video-feeds/:embed:cite]

リクエストを1つ送信するだけで管理画面のパスワードが取得できる脆弱性らしいです。  
[https://windabaft.co.jp/blog_ceo/?p=458:title]

人気リポジトリ  
[https://github.com/ezelf/CVE-2018-9995_dvr_credentials:title]


### 6位 605 ポイント 『Microsoft Exchange Server における権限を昇格される脆弱性(CVE-2018-8581)』

Microsoftが公表している脆弱性情報  
[CVE-2018-8581 | Microsoft Exchange Server の特権の昇格の脆弱性](https://portal.msrc.microsoft.com/ja-jp/security-guidance/advisory/CVE-2018-8581)

人気リポジトリ  
[https://github.com/Ridter/Exchange2domain:title]

### 5位 627 ポイント『OpenSSH における情報漏えいに関する脆弱性(CVE-2018-15473)』

[https://security.sios.com/vulnerability/openssh-security-vulnerability-20180818.html:embed:cite]

人気リポジトリ  
[https://github.com/Rhynorater/CVE-2018-15473-Exploit:title]

### 4位 674 ポイント 『Apache Struts2 における任意のコードが実行可能な脆弱性(CVE-2018-11776、S2-057)』

　Apache Struts2のRCE。  
　CVSS の Attack Complexity (AC)が「High」ということで、Base Scoreは「8.1」となっています。  
2017年に騒がせていたStruts2の脆弱性「CVE-2017-5638(S2-045・Base Score「10.0」)」に比べるとインパクトはやや劣っている印象。  

[http://www.intellilink.co.jp/article/vulner/180830.html:embed:cite]  

脆弱性の詳細情報  
[https://cwiki.apache.org/confluence/display/WW/S2-057:title]

人気リポジトリ  
[https://github.com/mazen160/struts-pwn_CVE-2018-11776:title]

### 3位 946 ポイント『複数の Microsoft Windows 製品における権限を昇格される脆弱性(CVE-2018-8120)』

LPE(local privilege escalation = ローカル権限昇格)

Microsoftが公表している脆弱性情報  
[CVE-2018-8120 | Microsoft Graphics コンポーネントの特権の昇格の脆弱性](https://portal.msrc.microsoft.com/ja-JP/security-guidance/advisory/CVE-2018-8120)

人気リポジトリ  
[https://github.com/unamer/CVE-2018-8120:title]

### 2位 1236 ポイント『Drupal における入力確認に関する脆弱性(CVE-2018-7600 通称「Drupalgeddon2」)』

<div style="text-align: center;">[f:id:motikan2010:20200106223514p:plain:w150]</div>

　Drupalのコアで発見されたRCEです。  
  
　Drupalで作成されているサイトのほぼ全てがパッチの対象となっていたということもあり、影響範囲が大きいことから注目されたと考えられます。  

[Drupal core - Highly critical - Remote Code Execution - SA-CORE-2018-002 | Drupal.org](https://www.drupal.org/sa-core-2018-002)

影響を受けるサイトは100万サイト以上あるとのことです。
[https://annai.co.jp/article/faq-about-sa-core-2018-002#toc--:title]

[Drupalの脆弱性（CVE-2018-8120） | MBSD](https://www.mbsd.jp/Whitepaper/CVE-2018-7600.pdf)

人気リポジトリ  
[https://github.com/dreadlocked/Drupalgeddon2:title]

### 1位 1242 ポイント『libssh における認証に関する脆弱性(CVE-2018-10933)』

　サーバ認証にlibSSHが用いられている場合に認証回避されてしまう脆弱性となっています。  
  
　脆弱であるサーバをShodanで簡単に検索できることから注目されたと考えられます(約2,000台のデバイスがlibsshバージョン0.6以上)。

[https://jp.tenable.com/blog/libssh-vulnerable-to-authentication-bypass-cve-2018-10933:embed:cite]

人気リポジトリ
[https://github.com/blacknbunny/CVE-2018-10933:title]


### TOP 10 一覧

|| スコア | CVE ID | リポ数 | スター合計 |
|-|-|-|-|-|
|  1位 | 1242 | CVE-2018-10933 | 27 | 972 |
|  2位 | 1236 | CVE-2018-7600  | 25 | 986 |
|  3位 |  946 | CVE-2018-8120  |  9 | 856 |
|  4位 |  674 | CVE-2018-11776 | 16 | 514 |
|  5位 |  627 | CVE-2018-15473 | 14 | 487 |
|  6位 |  605 | CVE-2018-8581  |  3 | 575 |
|  7位 |  592 | CVE-2018-9995  | 12 | 472 |
|  8位 |  582 | CVE-2018-20250 | 15 | 432 |
|  9位 |  575 | CVE-2018-8897  |  4 | 535 |
| 10位 |  541 | CVE-2018-3260  |  1 | 531 |


### おまけ

#### リポジトリ数ランキング

| リポ数 | CVE ID | 概要(上のTOP 10に入っているものは「-」表記) |
|-|-|-|
| 27 | CVE-2018-10933 | - |
| 26 | CVE-2018-6574  | Go におけるアクセス制御に関する脆弱性 |
| 25 | CVE-2018-7600  | - |
| 20 | CVE-2018-6389  | WordPress におけるサービス運用妨害 (DoS) の脆弱性 |
| 19 | CVE-2018-2628  | Oracle Fusion Middleware の Oracle WebLogic Server における<br/> WLS Core Components に関する脆弱性 |
| 16 | CVE-2018-11776 | - |
| 15 | CVE-2018-11235 | Git におけるセキュリティ機能に関する脆弱性 |
| 15 | CVE-2018-20250 | - |
| 14 | CVE-2018-4407  | 複数の Apple 製品におけるメモリ破損の脆弱性 |
| 14 | CVE-2018-15473 | - |


## 更新履歴

- 2020年1月6日 新規作成
