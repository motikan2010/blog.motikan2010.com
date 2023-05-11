<div style="text-align:center;">[f:id:motikan2010:20230104081346p:plain]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　新年あけましておめでとうございます！

　新年一発目の記事ということで、年に一度の人気脆弱性の紹介となります。

### 過去の人気脆弱性

　参考までに 2020年 と 2021年 の人気脆弱性を記載しておきます。

**▼ 2020年**

　2020年は、SMBv3 に対して認証無しでRCE可能な脆弱性 CVE-2020-11651 が1位でした。

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

**▼ 2021年**

　2021年は、最悪と評価された脆弱性である Log4Shell (Log4jの脆弱性) が1位でした。

|| スコア | 公表日 | CVE ID | リポ数 | スター合計 |
| -: | -: | - | - | -: | -: |
| 1位 | 15,134 | 2021/12/10 | CVE-2021-44228 / Apache Log4j | 345 | 11,684 |
| 2位 | 2,847 | 2021/01/26 | CVE-2021-3156 / sudo | 52 | 2,327 |
| 3位 |  2,696 | 2021/09/15 | CVE-2021-40444 / Microsoft MSHTML |  35 |  2,346 |
| 4位 | 2,034 | 2021/06/08 | CVE-2021-1675 / Windows 印刷スプーラー | 11 | 1,924 |
| 5位 | 1,667 | 2021/03/02 | CVE-2021-26855 / Microsoft Exchange Server | 41 | 1,257 |
| 6位 | 1,557 | 2021/10/05 | CVE-2021-41773 / Apache HTTP Server | 76 | 797 |
| 7位 | 1,149 | 2021/02/24 | CVE-2021-21972 / VMware vCenter Server | 26 | 889 |
| 8位 | 1,068 | 2021/11/09 | CVE-2021-42278 / Microsoft Windows Server | 5 | 1,018 |
| 9位 | 1,005 | 2021/01/29 | CVE-2021-25646 / Apache Druid | 7 | 935 |
| 10位 | 975 | 2021/11/09 | CVE-2021-42287 / Microsoft Windows Server | 1 | 965 |

## 2022年 人気脆弱性 TOP 10 in GitHub

　では、2022年の人気脆弱性を紹介していきます。

### 10位 - 753 pt 『macOS における権限昇格される脆弱性 (CVE-2022-46689)』

[f:id:motikan2010:20230105001607j:plain]

リポジトリ数: 3 / スター数: 723

> App がカーネル権限で任意のコードを実行できる可能性があります。  
「MacDirtyCow」と呼称されています。

　様々な機器に利用されている macOS の脆弱性であるためランクインしたと思われます。

PoC：<span><a href="https://github.com/ginsudev/WDBFontOverwrite" target="_blank">ginsudev / WDBFontOverwrite</a></span>

参考

- [https://support.apple.com/ja-jp/HT213534:title]
- [https://worthdoingbadly.com/macdirtycow/:title]
- [https://tools4hack.santalab.me/release-cve-2022-46689-macdirtycow-exploit-and-poc-trolllock-reborn-for-ios.html:title]

### 9位 - 766 pt 『Cobalt Strike におけるクロスサイトスクリプティングの脆弱性 (CVE-2022-39197)』

[f:id:motikan2010:20230105001129j:plain]

リポジトリ数: 13 / スター数: 636

> HelpSystems 社の Cobalt Strike 4.7 までのバージョンに XSS (Cross Site Scripting) の脆弱性があり、リモートの攻撃者が Cobalt Strike teamserver上で HTML を実行できる可能性がありました。  
この脆弱性が RCE につながる可能性があると記載されています。

　XSSの脆弱性ですが RCE に繋げることができる(RCE via XSS) ためランクインしたと思われます。

PoC: <span><a href="https://github.com/its-arun/CVE-2022-39197" target="_blank">its-arun / CVE-2022-39197</a></span>

参考

[https://mp.weixin.qq.com/s/Eb0pQ-1ebLSKPUFC7zS6dg:embed:cite]

- [https://securityintelligence.com/posts/analysis-rce-vulnerability-cobalt-strike/:title]

### 8位 - 821 pt 『OpenSSL におけるバッファオーバーフローの脆弱性 (CVE-2022-3602)』

[f:id:motikan2010:20230105001115p:plain]

リポジトリ数: 8 / スター数: 741

> OpenSSLには、X.509証明書の検証処理を通じてバッファオーバーフローが発生する脆弱性があります。
脆弱性が悪用された場合、攻撃者が用意した悪意のある証明書により、4バイト（CVE-2022-3602）あるいは任意のバイト数（CVE-2022-3786）のオーバーフローを発生させられる可能性があります。
結果として、サービス運用妨害（DoS）状態にされたり（CVE-2022-3602, CVE-2022-3786）、遠隔からのコード実行が行われたりする可能性があります（CVE-2022-3602）。

　脆弱性公開前まで『OpenSSLで史上2度目の「致命的」レベルの脆弱性』という評価から多くの人が警戒していた脆弱性です。そのため詳細の公表前後問わず注目されていました。  

　公開されてると重大度が下がっていたりとガッカリした人は多いのではないでしょうか。

[https://gigazine.net/news/20221101-openssl-critical-vulnerability-fix/:embed:cite]

PoC: <span><a href="https://github.com/NCSC-NL/OpenSSL-2022" target="_blank">NCSC-NL / OpenSSL-2022</a></span>

参考

- [https://www.jpcert.or.jp/at/2022/at220030.html:title]

[https://twitter.com/futurevuls/status/1587518546866601985:embed]

### 7位 - 1,289 pt 『Spring Cloud Gateway における未認証の任意のコード実行の脆弱性 (CVE-2022-22947)』

[f:id:motikan2010:20230105001021p:plain]

リポジトリ数: 55 スター数: 739

> GatewayActuator エンドポイントが有効化され、公開され、安全でない場合、アプリケーションはコードインジェクション攻撃に対して脆弱です。
リモートの攻撃者が悪意をもって細工されたリクエストを作成し、リモートホストで任意のリモート実行を行う可能性があります。

　Java界では有名なSpringプロジェクトが提供しているAPI構築基盤である Spring Cloud Gateway の脆弱性でした。

PoC: <span><a href="https://github.com/lucksec/Spring-Cloud-Gateway-CVE-2022-22947" target="_blank">lucksec / Spring-Cloud-Gateway-CVE-2022-22947</a></span>

参考

- [https://jp.tenable.com/plugins/nessus/163631:title]

### 6位 - 1,475 pt 『BIG-IP 製品の iControlREST コンポーネントにおける未認証の任意のコード実行の脆弱性 (CVE-2022-1388)』

リポジトリ数: 61 / スター数: 865

> F5社 の BIG-IP 製品の iControl REST API 機能にリモートコード実行の脆弱性があります。
認証されていないリモートの攻撃者はこれを悪用して、認証をバイパスし、ルート権限で任意のコードを実行する可能性があります。

　有名なネットワーク機器ブランドである BIG-IP の脆弱性であるためランクインしたと考えられます。  

PoC: <span><a href="https://github.com/horizon3ai/CVE-2022-1388" target="_blank">horizon3ai / CVE-2022-1388</a></span>

参考

- [https://blogs.jpcert.or.jp/ja/2022/09/bigip-exploit.html:title]
- [https://jp.tenable.com/plugins/nessus/160726:title]

### 5位 - 1,479 pt 『VMware Workspace ONE Access、VMware Identity Manager における未認証の任意のコード実行の脆弱性 (CVE-2022-22954)』

リポジトリ数: 26 スター数: 1,219

> VMware Workspace ONE Access、VMware Identity Manager におけるサーバーサイド テンプレート インジェクションに起因するリモートコード実行(RCE)の脆弱性(CVE-2022-22954)は、エクスプロイトが非常に容易(trivial)で、脆弱なデバイスにHTTPリクエストを1つ送るだけで悪用可能です。

　実際のサイバー攻撃に利用された脆弱性であるためランクインしたと考えられます。  
[https://www.security-next.com/136609:embed:cite]

PoC: <span><a href="https://github.com/Schira4396/VcenterKiller" target="_blank">Schira4396 / VcenterKiller</a></span>

参考

- [https://unit42.paloaltonetworks.jp/cve-2022-22954-vmware-vulnerabilities/:title]

### 4位 - 1,683 pt 『マイクロソフトサポート診断ツールに任意のコード実行の脆弱性 (CVE-2022-30190)』

[f:id:motikan2010:20230105001101j:plain]

リポジトリ数: 11 / スター数: 1,924

> Word などの呼び出し元アプリケーションから URL プロトコルを使用して MSDT が呼び出されると、リモートでコードが実行される脆弱性が存在します。
攻撃者がこの脆弱性を悪用した場合、呼び出し元のアプリケーションの権限で任意のコードが実行される可能性があります。

　一般的によく利用されるツールである Microsoft Word が標的となる脆弱性であるためランクインしたと考えられます。

PoC: <span><a href="https://github.com/komomon/CVE-2022-30190-follina-Office-MSDT-Fixed" target="_blank">komomon / CVE-2022-30190-follina-Office-MSDT-Fixed</a></span>

参考

- [https://msrc-blog.microsoft.com/2022/05/30/guidance-for-cve-2022-30190-microsoft-support-diagnostic-tool-vulnerability-jp/:title]
- [https://jpn.nec.com/cybersecurity/blog/220620/index.html:title]

### 3位 - 2,131 pt 『Confluence Server、Data Center における未認証の任意のコード実行の脆弱性 (CVE-2022-26134)』

[f:id:motikan2010:20230105001038p:plain]

リポジトリ数: 62 / スター数: 1,511

> Atlassian では、Confluence Data Center と Server における、重大な深刻度を持つ未認証のリモートコード実行の脆弱性を把握しています。
認証されていないユーザーが、Confluence Server または Data Center インスタンスで任意のコードを実行できる、OGNL インジェクションの脆弱性が存在します。 

　インターネット上に公開されていることが多いかつシェアも大きいソフトウェアの脆弱性であるためランクインしたと考えられます。

PoC: <span><a href="https://github.com/W01fh4cker/Serein" target="_blank">W01fh4cker / Serein</a></span>

参考

- [https://ja.confluence.atlassian.com/doc/confluence-security-advisory-2022-06-02-1130377146.html:title]
- [https://www.jpcert.or.jp/at/2022/at220015.html:title]

### 2位 - 2,176 pt 『Spring Framework における不適切なデータバインディング処理による任意コード実行の脆弱性 (CVE-2022-22965)』

[f:id:motikan2010:20230105001021p:plain]

リポジトリ数: 70 / スター数: 1,476

> Spring Framework には、データバインディングで使用する、CachedIntrospectionResults クラス内の PropertyDescriptor オブジェクトを安全に処理しない脆弱性があります。
その結果、攻撃者により class.classLoader を呼び出され、システム内で任意の Java コードが実行される可能性があります。

　Java業界で有名なフレームワークである Spring Framework の脆弱性であるためランクインしたと考えられます。  

　特定の実装方法でないと再現しない脆弱性であるため、実際のサイバー攻撃に用いることは難しそうです。

PoC: <span><a href="https://github.com/BobTheShoplifter/Spring4Shell-POC" target="_blank">BobTheShoplifter / Spring4Shell-POC</a></span>

参考

- [https://jvndb.jvn.jp/ja/contents/2022/JVNDB-2022-001498.html:title]

### 1位 - 15,134 pt 『Linux Kernel における権限昇格される脆弱性 (CVE-2022-0847 通称 Dirty Pipe)』

[f:id:motikan2010:20230105001720p:plain]

リポジトリ数: 80 / スター数: 2,124

> Linux Kernel には、権限昇格の脆弱性があります。
結果として、第三者が管理者に権限昇格を行う可能性があります。
発見者は脆弱性を「Dirty Pipe」と呼称し、脆弱性を実証するコードも公開しています。

　絶大なシェアを誇る Linux Kernel の権限昇格の脆弱性です。

　近年の当ランキングから RCE ではなく権限昇格 が1位になるのは非常に珍しいと思っています。  

　これだけ有名な脆弱性だと手元で再現させたくなりますが TryHackMe に環境が用意されており手軽に試すことができそうです。（※有償のサービス）
[https://tryhackme.com/room/dirtypipe:embed:cite]

PoC: <span><a href="https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit" target="_blank">Arinerron / CVE-2022-0847-DirtyPipe-Exploit</a></span>

参考

- [https://www.jpcert.or.jp/wr/2022/wr221101.html#10:title]
- [https://knqyf263.hatenablog.com/entry/2022/03/29/050815:title]

## まとめ

　2021年に Log4Shell があったため、比較してしまうと2022年は大人しめだった印象です。

　2023年にはどのような脆弱性がでてくるか楽しみです。

### 簡易一覧

|| スコア | 公表日 | CVE ID | リポ数 | スター合計 |
| -: | -: | - | - | -: | -: |
|  1位 | 2,924 | 2022/03/07 | CVE-2022-0847  | 80 | 2,124 |
|  2位 | 2,176 | 2022/04/01 | CVE-2022-22965 | 70 | 1,476 |
|  3位 | 2,131 | 2022/06/03 | CVE-2022-26134 | 62 | 1,511 |
|  4位 | 1,683 | 2022/06/01 | CVE-2022-30190 | 75 |   933 |
|  5位 | 1,479 | 2022/04/11 | CVE-2022-22954 | 26 | 1,219 |
|  6位 | 1,475 | 2022/05/05 | CVE-2022-1388  | 61 |   865 |
|  7位 | 1,289 | 2022/03/03 | CVE-2022-22947 | 55 |   739 |
|  8位 |   821 | 2022/11/01 | CVE-2022-3602  |  8 |   741 |
|  9位 |   766 | 2022/09/21 | CVE-2022-39197 | 13 |   636 |
| 10位 |   753 | 2022/12/15 | CVE-2022-46689 |  3 |   723 |

## 参考

- <span><a href="https://nvd.nist.gov/" target="_blank">NVD</a></span>
- <span><a href="https://jvndb.jvn.jp/" target="_blank">JVN iPedia - 脆弱性対策情報データベース</a></span>

## 更新履歴

- 2023年01月05日 新規作成


[blog:g:12921228815726579926:banner]
