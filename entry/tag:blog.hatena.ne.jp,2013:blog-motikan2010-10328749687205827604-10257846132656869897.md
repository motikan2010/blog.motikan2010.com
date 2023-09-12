<div style="text-align:center;">[f:id:motikan2010:20201130181851p:plain:w600]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>


## GUIアプリケーション型

　玄人向けのアプリケーションが多い印象です。

### Burp Suite

| | |
|-|-|
|公式|Burp Suite Scanner \| PortSwigger<br/>https://portswigger.net/burp|

-  診断を実施する上で便利な拡張プラグインが公開されている。独自に作成・公開することも可能です。
[https://github.com/PortSwigger:title]

- GUIアプリケーション
- シナリオ型の設定可能

### ZAP

| | |
|-|-|
| 公式 | ZAP<br/>https://www.zaproxy.org/ |
| GitHub | https://github.com/zaproxy/zaproxy |

- GUIアプリケーション
- スパイダー型(簡易スキャン)
- 2023年8月にOWASPから離れた。そのため「OWASP ZAP」ではなくなった。

### zap-cli

- Cookieの設定はコンテキストを読み込ませる必要があり難しそう

Dockerfile-zap-cli
https://gist.github.com/Grunny/6ea8d48d711c6ad28064

### Fiddler

　Windows上のデバッグでよく用いられるプロキシ。  今はMac OS向けもあります。  

　アドオン（Watcher、X5S）を追加することによって診断を行うことが可能になります。

[https://www.owasp.org/index.php/OWASP_Fiddler_Addons_for_Security_Testing_Project:title]

### Vex

日本の会社が開発しているツールです。

| | |
|-|-|
| 公式 | https://www.ubsecure.jp/vex |

### Watcher

| | |
|-|-|
| 公式 | Watcher: Web security testing tool and passive vulnerability scanner - CodePlex Archive<br/>https://archive.codeplex.com/?p=websecuritytool |

- パッシブスキャン

### X5S

| | |
|-|-|
| 公式 | x5s - test encodings and character transformations to find XSS hotspots - CodePlex Archive<br/>https://archive.codeplex.com/?p=xss |

- アクティブスキャン

<!-- more -->


## コンソールアプリケーション型（CUI型）

　コマンド１つで実行できるなど、簡単に利用できる印象です。

### SQLMap

| | |
|-|-|
| 公式 | sqlmap: automatic SQL injection and database takeover tool<br/>http://sqlmap.org/ |
| GitHub | https://github.com/sqlmapproject/sqlmap |

　GitHubスター数 15,000 を超える言わずと知れたツール。SQLインジェクション専用。

### NoSQLMap

| | |
|-|-|
| GitHub | https://github.com/codingo/NoSQLMap |

　SQLMap の NoSQL版

### w3af

| | |
|-|-|
| 公式 | w3af - Open Source Web Application Security Scanner<br/>http://w3af.org/ |
| GitHub | https://github.com/andresriancho/w3af |

[http://inaz2.hatenablog.com/entry/2016/03/07/193732:title]
[http://d.hatena.ne.jp/seezoo-cms/20110602/1306985390:title]

### Arachni (終了)

| | |
|-|-|
| 公式 | Home - Arachni - Web Application Security Scanner Framework<br/>http://www.arachni-scanner.com/ |
| Github | https://github.com/Arachni/arachni |
| Dockerイメージ | https://hub.docker.com/r/arachni/arachni/ |

　2020年1月28日をもってメンテナンスは終了されました。  
[https://www.arachni-scanner.com/blog/arachni-is-no-longer-maintained/:title]  

- スパイダー式

[http://inaz2.hatenablog.com/entry/2016/03/07/193828:title]

### SCNR

Arachni の後継プロダクトです。

> Arachni の開発で培った10年以上の経験が生かされており、最終的には自然な陳腐化によって Arachni に取って代わられることになるのです。

| | |
|-|-|
| 公式 | SCNR<br/>https://ecsypno.com/scnr-documentation/ |


### Scan My Server (※提供終了)

| | |
|-|-|
| 公式 | ScanMyServer<br/>https://scanmyserver.com/my_account/ |

### WhatWeb

| | |
|-|-|
| 公式|https://www.morningstarsecurity.com/research/whatweb |
| GitHub|https://github.com/urbanadventurer/WhatWeb |
| Dockerコンテナ|https://hub.docker.com/r/guidelacour/whatweb/ |

- フィンガープリントの取得

### Skipfish

| | |
|-|-|
|公式|Google Code Archive - Long-term storage for Google Code Project Hosting.<br/>https://code.google.com/archive/p/skipfish/|
|Dockerイメージ|https://hub.docker.com/r/k0st/alpine-skipfish/|

- DVWSをスキャンして「3時間7分」掛かった

[https://knowledge.sakura.ad.jp/389/:title]

### Nikto

| | |
|-|-|
|公式|Nikto2 \| CIRT.net<br>https://cirt.net/Nikto2|
|GitHub|https://github.com/sullo/nikto|

### Vega

| | |
|-|-|
|公式|Vega Vulnerability Scanner<br/>https://subgraph.com/vega/|
|GitHub|https://github.com/subgraph/Vega|

### Grabber

| | |
|-|-|
|公式|http://rgaucher.info/beta/grabber/|

### Wapiti

| | |
|-|-|
|公式|Wapiti : a Free and Open-Source web-application vulnerability scanner in Python for Windows, Linux, BSD, OSX<br/>http://wapiti.sourceforge.net/|
|||

### WebScarab

| | |
|-|-|
|公式|Category:OWASP WebScarab Project - OWASP<br/>https://www.owasp.org/index.php/Category:OWASP_WebScarab_Project|
|GitHub|https://github.com/OWASP/OWASP-WebScarab|

### Ratproxy

| | |
|-|-|
|公式|https://code.google.com/archive/p/ratproxy/|

- Google製
- 2009/05/14から更新が行われていない

### Wfuzz

| | |
|-|-|
|公式|Wfuzz: The Web fuzzer — Wfuzz 2.1.4 documentation<br/>https://wfuzz.readthedocs.io/en/latest/|
|Github|https://github.com/xmendez/wfuzz|

### Grendel-Scan

| | |
|-|-|
| 公式 | https://sourceforge.net/projects/grendel/ |
| Source | https://sourceforge.net/p/grendel/code/ci/master/tree/ |

### WAScan

| | |
|-|-|
|公式|-|
|GitHub|https://github.com/m4ll0k/WAScan|

　2018月7月20日に公開され、比較的新しいツールではあるが、既にGitHubで1,000スターを取得している診断ツール。

### Paros

| | |
|-|-|
|Wiki|https://github.com/zaproxy/zap-core-help/wiki/HelpParos|

- OWASP ZAPの元となった診断ツール
- 2006/08/08から更新が行われない

## SaaS型

　ブラウザさえあれば実施することが可能。  （多いため日本で提供されているものを優先的に調査・記載しています。）

### VAddy（バディ）

| | |
|-|-|
| 公式 | VAddy - 株式会社ビットフォレスト<br/>https://vaddy.net/ja/ |

- 【2015.09.07】継続的Webセキュリティテストサービス「VAddy（バディ）」、製品版／有料プランの提供を開始しました | VAddy クラウド型Web脆弱性診断ツール  
https://vaddy.net/ja/release/20150907.html

### AeyeScan（エーアイスキャン）

| | |
|-|-|
| 公式 | AeyeScan- 株式会社エーアイセキュリティラボ<br/>https://www.aeyescan.jp/ |

- 【2019.12.17】AIとクラウドを活用した、ホームページ向けサイバーセキュリティ自動診断(AI診断)サービス提供開始  
https://prtimes.jp/main/html/rd/p/000000002.000051317.html

### komabato（コマバト）

| | |
|-|-|
| 公式 | komabato - 株式会社ユービーセキュア<br/>https://www.ubsecure.jp/komabato |


- 【2021.10.01】 ユービーセキュア、システム開発プロセスの中で担当者自らセキュリティのテストができるツール「komabato」を提供開始  
https://prtimes.jp/main/html/rd/p/000000010.000060051.html

### Securify（セキュリファイ）

| | |
|-|-|
| 公式 | Securify - 株式会社スリーシェイク | Securify<br/>https://www.securify.jp/securify-scan |

- 【2021.12.01】 スリーシェイク、自動脆弱性診断ツール「Securify」の無料提供を開始  
https://prtimes.jp/main/html/rd/p/000000045.000024873.html

### secuas（セキュアズ）

| | |
|-|-|
| 公式 | secuas - 有限会社アズリアル<br/>https://secuas-cloud.com/ |

- 【2022.03.29】多くの企業が情報漏洩対策に取り組めるよう、「誰でも」「わかりやすく」「安価」な診断ツール「secuas(セキュアズ)」がリリース  
https://prtimes.jp/main/html/rd/p/000000001.000092747.html

### Walti (※提供終了)

<b>※ 2021年7月20日 サービス終了</b>

| | |
|-|-|
| 公式 |サーバーサイドのセキュリティスキャンをもっと身近に - 株式会社ウォルティ。</br>https://walti[.]io/|

　診断対象サイトのURLを指定するだけで、診断を実施することが可能。

- 【2014.12.17】Webサイトのセキュリティを簡単にチェックできるSaaS「Walti.io(ウォルティ・アイオー）」正式リリース  
https://prtimes.jp/main/html/rd/p/000000001.000012243.html

#### 特徴

　サービストップページに「スキャン１回５円〜」と記載があるだけあり、他のSasS型の診断ツールと比べて価格が安い。

#### 診断種別

[f:id:motikan2010:20190920222845p:plain:w400]  
　ターゲット(ドメイン)毎に初回スキャンは無料となっておりましたので、スキャンを試してみました。

|スキャン対象|利用ソフト|診断種別|
|-|-|-|
|Firewall|nmap|ネットワーク診断|
|Web Server|nikto|プラットフォーム診断|
|Web App|skipfish|Webアプリケーション診断|

#### 「Web Server」のスキャン結果

[f:id:motikan2010:20190920223047p:plain:w400]


### Acunetix WVS

| | |
|-|-|
|公式|Acunetix Vulnerability Scanner: Web Application Security<br/>https://www.acunetix.com/vulnerability-scanner/|

- 質は良さそうだが、お値段がそこそこ(20ターゲット以下60万円)

### Qualys - Web Application Scanning

| | |
|-|-|
|公式|Web Application Scanning \| Qualys, Inc.<br/>https://www.qualys.com/apps/web-app-scanning/|

[https://www.qualys.com/docs/qualys-was-getting-started-guide.pdf:title]

### UpGuard - Cloud Scanner

| | |
|-|-|
|公式|UpGuard - Cloud Scanner<br/>https://webscan.upguard.com/|

- 登録不要
- サイトURLを入力するだけで診断が可能

#### 診断結果

[f:id:motikan2010:20190920223220p:plain:w400]

### Tinfoil Security

| | |
|-|-|
|公式|Website Security \| Recurring, Affordable, and Usable<br/>https://www.tinfoilsecurity.com/|

- 診断の起点となるサイトのURL1つ設定して、そこからスパイダー式で診断が行われる
- 30日間のトライアル版あり(2018/10/23現在)
- PDF形式で診断レポートの出力可能

#### 診断結果

##### 診断結果トップ
[f:id:motikan2010:20190920223331p:plain:w400]  

##### 脆弱性一覧

[f:id:motikan2010:20190920223405p:plain:w400]  

##### 脆弱性詳細

- 「Rescan」ボタンがあり、対象の脆弱性のみを診断も可能?(未検証)  
[f:id:motikan2010:20190920223443p:plain:w400]  


### Sucuri - Free Website Malware Scanner

| | |
|-|-|
| 公式 | Sucuri SiteCheck<br/>https://sitecheck.sucuri.net/ |

　無料のマルウェアスキャンサービス。  
URLで指定したサイトがマルウェアに感染していないか、ブラックリストに登録されていないかを確認することができる。

#### 診断結果

[f:id:motikan2010:20190920223518p:plain:w400]  

### Web Inspector - Free Website Malware Scanner

| | |
|-|-|
| 公式 | Website Malware Scanner \| Free Online Web Scan for Malware Infections<br/>https://app.webinspector.com/ |

　無料のマルウェアスキャンサービス。  
診断対象がマルウェア感染・バックドア設置がされていないかを確認することができる。

#### 診断結果

[f:id:motikan2010:20190920223552p:plain:w400]  

## 参考

- <span><a href="http://sectoolmarket.com/price-and-feature-comparison-of-web-application-scanners-unified-list.html" target="_blank">The Prices vs. Features of Web Application Vulnerability Scanners</a></span>
- <span><a href="https://owasp.org/www-community/Vulnerability_Scanning_Tools" target="_blank">Vulnerability Scanning Tools | OWASP</a></span>
- <span><a href="https://resources.infosecinstitute.com/topic/14-popular-web-application-vulnerability-scanners/#gref" target="_blank">14 best open-source web application vulnerability scanners [updated for 2020] - Infosec Resources</a></span>
- <s>https:// www.techworld.com/picture-gallery/security/best-vulnerability-scanners-3664450/</s>

