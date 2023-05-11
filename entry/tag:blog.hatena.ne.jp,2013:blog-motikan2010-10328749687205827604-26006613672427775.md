<div style="text-align:center;">[f:id:motikan2010:20210101181451p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　「CVE-2020-XXXXXX」が採番された脆弱性の人気TOP 10です。  

脆弱性の PoC リポジトリ数やそのリポジトリへのスター数からポイントを算出しています。  
算出方法については下の2019年版に記載しています。  

[https://blog.motikan2010.com/entry/2019/12/31/2019%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub:embed:cite]  

## TOP 10

### 10位 495 ポイント 『SaltStack Salt における入力確認に関する脆弱性 (CVE-2020-11651)』

公表日:`2020/04/30`

<figure class="figure-image figure-image-fotolife" title="SaltStack のロゴ">[f:id:motikan2010:20210101171734p:plain:w400]<figcaption>SaltStack のロゴ</figcaption></figure>

#### SaltStack とは

> SaltStack/Saltは、システム管理者がサーバーのプロビジョニングおよび管理タスクを自動化するための構成管理（Confuguration Management、CM）とオーケストレーションのツールです。

[https://blog.ipswitch.com/jp/what-is-saltstack:embed:cite]

#### 脆弱性の内容

> 脆弱性が悪用された場合、リモートからの攻撃によって、認証不要でマスターサーバ上のユーザトークンが窃取されたり、管理対象サーバ上で任意のコマンドを実行されたりするなどの可能性があります。

[https://www.jpcert.or.jp/at/2020/at200020.html:embed:cite]

### 9位 508 ポイント 『複数の Microsoft Windows 製品における権限を昇格される脆弱性 (CVE-2020-0787)』

公表日:`2020/03/12`

> ベンダは、本脆弱性を「Windows Background Intelligent Transfer Service の特権の昇格の脆弱性」として公開しています。

#### Background Intelligent Transfer Service (BITS) とは

> BITSとは、Windowsの機能の一つで、通信回線の空いている伝送容量を用いて他のプログラムの通信を阻害せずにファイルの送受信を行うもの。

[http://e-words.jp/w/BITS.html:embed:cite]

#### 脆弱性の内容

[https://www.terabitweb.com/2020/03/10/cve-2020-0787-windows-bits-eop/:embed:cite]

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/_Ukmk4tstkk" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### 8位 1026 ポイント 『Apache Tomcat の不適切な認可処理 (CVE-2020-1938 通称「Ghostcat」)』

公表日:`2020/02/24`

<figure class="figure-image figure-image-fotolife" title="Ghostcat のロゴ">[f:id:motikan2010:20201231200539p:plain:w200]<figcaption>Ghostcat のロゴ</figcaption></figure>

#### 脆弱性の内容

　脆弱性を発見したリサーチャ(Chaitin Tech)の記事。  
[https://www.chaitin.cn/en/ghostcat:embed:cite]

　脆弱性の解説記事。  
[https://blog.trendmicro.co.jp/archives/24748:embed:cite]

### 7位 1076 ポイント 『Microsoft Exchange Server におけるリモートでコードを実行される脆弱性 (CVE-2020-0688)』

公表日:`2020/02/11`

#### Exchange Server とは

> Exchange Serverとは、Microsoftが開発しているサーバーソフトウェアの一種である。メールサーバーの機能とグループウェアの機能を統合的に管理することができる。

[https://www.weblio.jp/content/Exchange+Server:embed:cite]

#### 脆弱性の内容

[https://blog.macnica.net/blog/2020/06/exchangeserver-shodan.html:embed:cite]

[https://jp.tenable.com/blog/cve-2020-0688-microsoft-exchange-server-static-key-flaw-could-lead-to-remote-code-execution:embed:cite]

### 6位 1185 ポイント 『複数の BIG-IP 製品におけるコードインジェクションの脆弱性 (CVE-2020-5902)』

公表日:`2020/07/01`

#### 脆弱性の内容

　製品提供元である F5 社による脆弱性情報です。  
<a href="https://support.f5.com/csp/article/K52145254" target="_blank">K52145254: TMUI RCE vulnerability CVE-2020-5902</a>  

　日本語での情報は下の記事が参考になります。  
[https://medium.com/anti-pattern-engineering/big-ip%E3%81%AE%E8%84%86%E5%BC%B1%E6%80%A7%E3%81%AE%E5%AF%BE%E5%BF%9C-1d719ad9cddd:embed:cite]

### 5位 1520 ポイント 『Oracle Fusion Middleware の Oracle WebLogic Server における WLS Core Components に関する脆弱性 (CVE-2020-2551)』

公表日:`2020/01/15`

<div style="text-align:center;">[f:id:motikan2010:20210101175240j:plain:w400]</div>


#### IIOP プロトコル関連

> IIOPを使用すると、異なるプログラミング言語で記述された分散プログラムどうしがインターネット経由で通信できるようになります。

[https://docs.oracle.com/cd/E84527_01/wls/FMWCH/pagehelp/Corecoreserverserverprotocolsiioptitle.html:title]

[https://docs.oracle.com/cd/E80149_01/wls/WLRMI/iiop_config.htm:embed:cite]

#### 脆弱性の内容

[https://medium.com/@qazbnm456/cve-2020-2551-unauthenticated-remote-code-execution-in-iiop-protocol-via-malicious-jndi-lookup-119bac7c1eb2:embed:cite]

### 4位 1709 ポイント 『Microsoft Windows CryptoAPI における Elliptic Curve Cryptography (ECC) 証明書の検証不備の脆弱性 (CVE-2020-0601)』

公表日:`2020/01/14`

#### 脆弱性の内容

[https://jovi0608.hatenablog.com/entry/2020/01/18/145515:embed:cite]

### 3位 3149 ポイント 『複数の Microsoft Windows 製品における権限昇格の脆弱性 (Netlogon) (CVE-2020-1472 通称「Zerologon」)』

公表日:`2020/08/17`

#### Netlogon とは

　脆弱性と共に<b> Netlogon リモートプロトコル</b>の解説が記載されています。  
[https://www.nri-secure.co.jp/blog/looking-back-on-the-vulnerability-zerologon#TOC_HL1:embed:cite]

#### 脆弱性の内容

　脆弱性の発表元である Secura のホワイトペーパーです。  
[https://www.secura.com/blog/zero-logon:embed:cite]

解説動画  
<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/1h_wlMWwyB4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### 2位 3451 ポイント 『Oracle Fusion Middleware の Oracle WebLogic Server における Console に関する脆弱性 (CVE-2020-14882)』

公表日:`2020/10/21`

<div style="text-align:center;">[f:id:motikan2010:20210101175240j:plain:w400]</div>

#### 脆弱性の内容

[https://nekochansecurity555.hatenablog.com/entry/2020/11/12/205137:embed:cite]

[http://www.npa.go.jp/cyberpolice/important/2020/202012241.html:embed:cite]

### 1位 3696 ポイント 『Microsoft SMBv3 の接続処理にリモートコード実行の脆弱性 (CVE-2020-0796)』

公表日:`2020/03/12`

#### 脆弱性の内容

　脆弱性の検証レポートです。  
[http://www.intellilink.co.jp/article/vulner/200319.html:embed:cite]

　脆弱性に関する情報がまとめられています。  
[https://piyolog.hatenadiary.jp/entry/2020/03/13/073949:embed:cite]

#### 脆弱性に関連するアクセスの観測状況

<figure class="figure-image figure-image-fotolife" title="Microsoft SMBv3の脆弱性（CVE-2020-0796）に関連するアクセスの観測状況（グラフ）">[f:id:motikan2010:20201231223324p:plain:w500]<figcaption>Microsoft SMBv3の脆弱性（CVE-2020-0796）に関連するアクセスの観測状況（グラフ）</figcaption></figure>  

[https://www.npa.go.jp/cyberpolice/important/2020/202003131.html:title]

### TOP 10 一覧

|  | スコア | CVE ID | リポ数 | スター合計 |
| -: | -: | - | -: | -: |
|  1位 | 3,696 | CVE-2020-11651 / Microsoft SMBv3 | 69 | 3,006 |
|  2位 | 3,451 | CVE-2020-0787 / Oracle WebLogic Server | 23 | 3,221 |
|  3位 | 3,149 | CVE-2020-1938 / Netlogon | 45 | 2,699 |
|  4位 | 1,709 | CVE-2020-0688 / Microsoft Windows CryptoAPI | 32 | 1,389 |
|  5位 | 1,520 | CVE-2020-5902 / Oracle WebLogic Server |  8 | 1,440 |
|  6位 | 1,185 | CVE-2020-2551 / BIG-IP 製品 | 51 | 675 |
|  7位 | 1,076 |  CVE-2020-0601 / Microsoft Exchange Server | 17 | 906 |
|  8位 | 1,026 | CVE-2020-1472 / Apache Tomcat | 23 | 796 |
|  9位 |  508 | CVE-2020-14882<br/> / Windows Background Intelligent Transfer Service |  3 | 478 |
| 10位 |  495 | CVE-2020-0796 / SaltStack Salt | 12 | 375 |

## おまけ

| <span style="white-space: nowrap;">リポ数</span> | CVE ID | 概要 (上のTOP 10に入っているものは「-」表記) |
| - | - | - |
| 69 | CVE-2020-0796 | - |
| 51 | CVE-2020-5902 | - |
| 45 | CVE-2020-1472 | - |
| 32 | CVE-2020-0601 | - |
| 23 | CVE-2020-14882 | - |
| 23 | CVE-2020-1938 | - |
| 17 | CVE-2020-0688 | - |
| 15 | <span style="white-space: nowrap;">CVE-2020-16898</span> | Microsoft Windows 10 および Windows Server におけるリモートでコードを実行される脆弱性<br/>(Windows TCP/IP のリモートでコードが実行される脆弱性) |
| 14 | <span style="white-space: nowrap;">CVE-2020-1350</span> | 複数の Microsoft Windows 製品におけるリモートでコードを実行される脆弱性<br/>(Windows DNS サーバーのリモートでコードが実行される脆弱性) |
| 13 | <span style="white-space: nowrap;">CVE-2020-3452</span> | Cisco Adaptive Security Appliance ソフトウェアおよび Firepower Threat Defense ソフトウェアにおける入力確認に関する脆弱性 |

## 参考

- JPCERT コーディネーションセンター
https://www.jpcert.or.jp/
- Japan Vulnerability Notes
https://jvn.jp/

## 更新履歴

- 2020年1月1日 新規作成
