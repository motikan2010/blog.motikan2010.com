<div style="text-align:center;">[f:id:motikan2010:20231224073558p:plain:w500]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

2023年も終わりが近づいてきたということで、毎年恒例の『人気脆弱性トップ10（2023年版）』を発表していきます。  

ちなみに**前年**のランキングは次のようになっています。  

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

過去の結果はコチラから

- [2022年](https://blog.motikan2010.com/entry/2023/01/05/2022%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)
- [2021年](https://blog.motikan2010.com/entry/2022/01/19/2021%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)
- [2020年](https://blog.motikan2010.com/entry/2021/01/01/2020%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)
- [2019年](https://blog.motikan2010.com/entry/2019/12/31/2019%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)
- [2018年](https://blog.motikan2010.com/entry/2020/01/06/2018%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)


### スコアの計算方法

ランキングに利用するスコアの算出方法です。  

- ① リポジトリ毎に 10pt
- ② スター毎に 1pt

「①＋②の合計pt」でランキングを付けています。


### 対象となる脆弱性（2022年からの変更あり）

対象となる範囲を前回から変更し、**12月に公開された脆弱性は当年の調査対象から除外する**ようにしました。  
12月に公開された脆弱性が不利にならないようにすることが目的となっています。  

（例）2023年が調査対象になる場合

- 変更前
  - 「CVE-2023-*」
- 変更後
  - 「CVE-2023-*」かつ「公表日が 2022/11/30 以前」
  - 「CVE-2022-*」かつ「公表日が 2022/12/01 以降」

（変更した経緯があり、前回ランクインした脆弱性が今回もランクインしています。）


## 2023年 人気脆弱性 TOP 10 in GitHub

### 10位 573 pt - Atlassian社の Confluence Data Center および Server におけるアクセス制御破損の脆弱性（CVE-2023-22515）

<div style="text-align:center;">[f:id:motikan2010:20231223222935p:plain:w400]</div>

リポジトリ数: **18** / スター数: **393**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>Atlassian社が提供している「Confluence Server」「Data Center」の脆弱性</li>
<li><strong>リモートより承認なしに「管理者アカウント」を作成</strong>し、インスタンスに対してアクセスすることが可能</li>
<li>本脆弱性を利用したと思われる不正なアクセスが一部顧客（ユーザ）より報告を受けている</li>
<li>既知の国家攻撃者が CVE-2023-22515 を積極的に悪用していることを示唆する証拠を得られている</li>
<li>CISA KEV に追加（2023年10月5日）</li>
</ul>
</div>


[https://twitter.com/ptswarm/status/1711699559359865200:embed]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【PoC】

- Chocapikk/CVE-2023-22515  
https://github.com/Chocapikk/CVE-2023-22515

【参考】

- アトラシアン サポート | Confluence Data Center および Server におけるアクセス制御破損の脆弱性  
https://ja.confluence.atlassian.com/security/cve-2023-22515-privilege-escalation-vulnerability-in-confluence-data-center-and-server-1295682276.html
- Security NEXT | 「Atlassian Confluence」にゼロデイ脆弱性 - 侵害状況の確認を  
https://www.security-next.com/149956
</section>

### 9位 574 pt - Citrix社のネットワーク機器 Citrix ADC および Citrix Gateway に認証不要のRCEの脆弱性（CVE-2023-3519）

<div style="text-align:center;">[f:id:motikan2010:20231224043754p:plain]</div>

リポジトリ数: **14** / スター数: **434**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>Citrix社の「Citrix NetScaler ADC（Citrix ADC）」および「NetScaler Gateway（Citrix Gateway）」の脆弱性</li>
<li>悪用されると、<strong>認証されていない遠隔の第三者が任意のコードを実行</strong>が可能</li>
<li>Citrixは脆弱性を悪用する攻撃を確認している</li>
<li>CISA KEV に追加（2023年7月19日）</li>
<li>JPCERT/CC から注意喚起あり</li>
</ul>
</div>

[https://twitter.com/bishopfox/status/1687608669087916032:embed]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【PoC】

- BishopFox/CVE-2023-3519  
https://github.com/BishopFox/CVE-2023-3519

【参考】

- JPCERT/CC | Citrix ADCおよびCitrix Gatewayの脆弱性（CVE-2023-3519）に関する注意喚起  
https://www.jpcert.or.jp/at/2023/at230013.html
- Palo Alto Networks | 脅威に関する情報: 顧客管理下の Citrix サーバーにリモートコード実行の脆弱性  
https://unit42.paloaltonetworks.jp/threat-brief-citrix-cve-2023-3519/
</section>

### 8位 605 pt - Linux kernel の権限昇格の可能性の脆弱性 （CVE-2023-0386）

<div style="text-align:center;">[f:id:motikan2010:20231224013302p:plain]</div>

リポジトリ数: **8** / スター数: **525**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>一般権限アカウントで脆弱な Linux システムへの侵入に成功した攻撃者は、<strong>root 権限への昇格</strong>が可能</li>
</ul>
</div>

[https://twitter.com/cyber_advising/status/1654886791344685056:embed]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【PoC】

- xkaneiki/CVE-2023-0386  
https://github.com/xkaneiki/CVE-2023-0386

【参考】

- ScanNetSecurity | Oferlayfs ファイルシステムにおいて異なるマウント間でのファイルマッピング不備により権限昇格が可能となる脆弱性（Scan Tech Report）
https://scan.netsecurity.ne.jp/article/2023/08/29/49878.html
- SIOS SECURITY BLOG | Linux Kernelの脆弱性(Important: CVE-2023-0386)  
https://security.sios.jp/vulnerability/kernel-security-vulnerability-20230323/
</section>

### 7位 686 pt - パスワード管理ツール「KeePass」にメモリダンプでマスターパスワードを復元できる脆弱性（CVE-2023-32784）

<div style="text-align:center;">[f:id:motikan2010:20231224012415p:plain]</div>

リポジトリ数: **8** / スター数: **606**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>KeePassは Windows・Mac・Linux 上で動作する人気のオープンソースのパスワードマネージャー</li>
<li>実行中のプロセスの<strong>メモリからマスターキーを平文で抽出</strong>することができる脆弱性

<ul>
<li>マスターキーにより攻撃者は保存されているすべての認証情報にアクセスすることができるようになる</li>
</ul>
</li>
</ul>
</div>

[https://twitter.com/hack_git/status/1659089216859406337:embed]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【PoC】

- vdohney/keepass-password-dumper: Original PoC for CVE-2023-32784  
https://github.com/vdohney/keepass-password-dumper

【参考】

- Sysdig | KeePass CVE-2023-32784：プロセスメモリーダンプの検知
https://sysdig.jp/blog/keepass-cve-2023-32784-detection/
- 窓の杜 | フリーのパスワード管理ツール「KeePass」に脆弱性、マスターパスワードを復元される  
https://forest.watch.impress.co.jp/docs/news/1501981.html
</section>

### 6位 732 pt - Linuxの標準Cライブラリ（GNU Cライブラリ）におけるバッファオーバーフローの脆弱性（CVE-2023-4911 / Looney Tunables）

<div style="text-align:center;">[f:id:motikan2010:20231224013302p:plain]</div>

リポジトリ数: **13** / スター数: **602**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>2023年10月に発見されたLinuxの標準Cライブラリ（GNU Cライブラリあるいはglibc）におけるバッファオーバーフローの脆弱性</li>
<li>攻撃者は脆弱性を悪用し、root権限への昇格や不正アクセスを行うことが可能</li>
<li>CISA KEV に追加（2023年11月21日）</li>
</ul>
</div>

[https://twitter.com/rdjgr/status/1709603179300524388:embed]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【POC】

- leesh3288/CVE-2023-4911  
https://github.com/leesh3288/CVE-2023-4911

【参考】

- SOMPO CYBER SECURITY | Looney Tunablesとは【用語集詳細】
https://www.sompocybersecurity.com/column/glossary/looney-tunables
- IoT OT Security News | CISA KEV 警告 23/11/21：Linux の Looney Tunables を悪用リストに追加  
https://iototsecnews.jp/2023/11/21/cisa-orders-federal-agencies-to-patch-looney-tunables-linux-bug/
</section>

### 5位 770pt - Microsoft社の WinSock 用 Windows Ancillary Function Driver に権限昇格の脆弱性 （CVE-2023-21768）

<div style="text-align:center;">[f:id:motikan2010:20231224014520p:plain]</div>

リポジトリ数: **9** / スター数: **680**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>Microsoft Windows 11 とMicrosoft Windows Server 2022 が対象</li>
<li>Microsoft Windows OS に <strong>SYSTEM 権限への昇格</strong>が可能となる脆弱性</li>
</ul>
</div>

[https://twitter.com/chompie1337/status/1633498392125997056:embed]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【PoC】

- chompie1337/Windows_LPE_AFD_CVE-2023-21768  
https://github.com/chompie1337/Windows_LPE_AFD_CVE-2023-21768

【参考】

- CVE-2023-21768 - セキュリティ更新プログラム ガイド - Microsoft - WinSock 用 Windows Ancillary Function Driver の特権の昇格の脆弱性  
https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-21768
- ScanNetSecurity Microsoft Windows OS における Winsock でのデータ検証不備に起因する権限昇格が可能となる脆弱性（Scan Tech Report）  
https://scan.netsecurity.ne.jp/article/2023/05/18/49378.html
- JVNDB-2023-001036 - JVN iPedia - 脆弱性対策情報データベース  
https://jvndb.jvn.jp/ja/contents/2023/JVNDB-2023-001036.html
</section>

### 4位 795 pt - ImageMagick の検証不備により任意のファイルが読み取り可能となる脆弱性（CVE-2022-44268）

<div style="text-align:center;">[f:id:motikan2010:20231224023233p:plain]</div>

リポジトリ数: **23** / スター数: **565**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>悪意のある PNG ファイルの変換により、システム上の任意のファイルが読み取られる</li>
</ul>
</div>

[https://twitter.com/h4x0r_dz/status/1621193760385163266:embed]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【PoC】

- duc-nt/CVE-2022-44268-ImageMagick-Arbitrary-File-Read-PoC  
https://github.com/duc-nt/CVE-2022-44268-ImageMagick-Arbitrary-File-Read-PoC

【参考】

- ScanNetSecurity | ImageMagick において PNG 画像処理中の profile 情報の検証不備により任意のファイルが読み取り可能となる脆弱性  
https://scan.netsecurity.ne.jp/article/2023/04/05/49167.html
- 【ImageMagick】ImageMagickであった情報漏洩の脆弱性を詳しく解説！【cve-2022-44268】【悪用厳禁】 - YouTube  
https://www.youtube.com/watch?v=KbCR_lkVU5w
- CVE-2022-44268 ImageMagick Arbitrary File Read - shimojubox  
https://scrapbox.io/shimoju/CVE-2022-44268_ImageMagick_Arbitrary_File_Read
</section>

### 3位 998 pt - Windows社の Microsoft Outlook における特権昇格の脆弱性 （CVE-2023-23397）

<div style="text-align:center;">[f:id:motikan2010:20231224021024p:plain]</div>

リポジトリ数: **25** / スター数: **748**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>Windows Microsoft Outlook クライアントの脆弱性</li>
<li>脆弱性を悪用することで、攻撃対象ネットワークのアカウントと認証情報を窃取すること、もしくはマルウェアなどのペイロードを送り込むことが可能になる</li>
<li>ユーザの介在を必要としない（ゼロタッチ）の脆弱性</li>
<li>CISA KEV に追加（2023年3月14日）</li>
</ul>
</div>

[https://www.youtube.com/watch?v=SuDWMYpImj0:embed:cite]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【PoC】

- api0cradle/CVE-2023-23397-POC-Powershell  
https://github.com/api0cradle/CVE-2023-23397-POC-Powershell

【参考】

- トレンドマイクロ | Outlook for Windowsに深刻な特権昇格の脆弱性「CVE-2023-23397」：対処すべき内容と注意点  
https://www.trendmicro.com/ja_jp/research/23/c/patch-cve-2023-23397-immediately-what-you-need-to-know-and-do.html
- Palo Alto Networks | 脅威に関する情報: Microsoft Outlookにおける特権昇格の脆弱性(CVE-2023-23397)  
https://unit42.paloaltonetworks.jp/threat-brief-cve-2023-23397/
- TryHackMe | Outlook NTLM Leak  
https://tryhackme.com/room/outlookntlmleak
</section>

### 2位 1,543 pt - RARLAB WinRAR のZIPファイル閲覧時に任意のコード実行となる脆弱性 （CVE-2023-38831）

<div style="text-align:center;">[f:id:motikan2010:20231224022714p:plain]</div>

リポジトリ数: **38** / スター数: **1,163**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>ZIP アーカイブ内の良性のファイルを閲覧しようとする際に攻撃者による<strong>任意のコード実行</strong>が行われる可能性</li>
<li>Group-IB の調査結果によると、この脆弱性は2023年4月以降にトレーダーを標的にするゼロデイ攻撃で利用された</li>
<li>CISA KEV に追加（2023年8月24日）</li>
</ul>
</div>

[https://www.youtube.com/watch?v=B1J0yHoQhdc:embed:cite]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【PoC】

- b1tg/CVE-2023-38831-winrar-exploit  
https://github.com/b1tg/CVE-2023-38831-winrar-exploit

【参考】

- 窓の杜 | 「WinRAR」に政府系ハッカーが悪用する脆弱性 ～Google脅威分析グループが警告  
https://forest.watch.impress.co.jp/docs/news/1540646.html
- IoT OT Security News | WinRAR のゼロデイ CVE-2023-38831 の悪用：世界中のトレーダーが狙われている  
https://iototsecnews.jp/2023/08/23/winrar-vulnerability-affects-traders-worldwide/
- Group-IB Blog | Traders' dollars in danger: CVE-2023-38831 zero-day vulnerability in WinRAR exploited by cybercriminals to target traders  
https://www.group-ib.com/blog/cve-2023-38831-winrar-zero-day/
</section>

### 1位 1,847 pt - macOS における権限昇格される脆弱性（CVE-2022-46689 / MacDirtyCow）

<div style="text-align:center;">[f:id:motikan2010:20230105001607j:plain]</div>

リポジトリ数: **13** / スター数: **1,717**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>iPhoneの改造に利用されている脆弱性
<ul>
<li><strong>一般(?)iPhoneユーザが利用する脆弱性のため人気</strong>と考えられます</li>
</ul>
</li>
<li>App がカーネル権限で任意のコードを実行できる可能性があります。</li>
</ul>
</div>

[https://www.youtube.com/watch?v=MoEdFeB4Rw8:embed:cite]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【PoC】

- ginsudev/WDBFontOverwrite  
https://github.com/ginsudev/WDBFontOverwrite

【参考】

- Worth Doing Badly | Get root on macOS 13.0.1 with CVE-2022-46689, the macOS Dirty Cow bug  
https://worthdoingbadly.com/macdirtycow/
- Tools 4 Hack | iOS 16.1.2以下で使用可能なExploitが公開、更に実際にExploitを利用したPoCアプリもリリースされる
https://tools4hack.santalab.me/release-cve-2022-46689-macdirtycow-exploit-and-poc-trolllock-reborn-for-ios.html
- OffensiveCon23 | Ian Beer - MacDirtyCow – Auditing and Exploiting XNU Virtual Memory - YouTube  
[https://www.youtube.com/watch?v=jrxOhGGxDZ8:embed:cite]
</section>

## まとめ

今年の結果をまとめたのが以下の表です。  

半分の脆弱性が CISA KEV に記載されており、「人気の脆弱性 ≒ 悪用される脆弱性」と言えるでしょう。  

また KEV に登録されていなくても、「Linux Kernel」や「ImageMagick」のようなシェアの大きいソフトウェアの脆弱性も人気であることが分かります。  

<section style="overflow-x: scroll;white-space: nowrap;">

|| スコア | 公表日 | CVE ID | リポ数 | スター<br>合計 | CVSS<br>基本値 | KEV | EPSS<br>(2023/12/18) | EPSS<br>Percentile | |
| -: | -: | - | - | -: | -: | :-: | :-: | :- | :- | - |
|  1位 | 1,847 | 2022/12/15 | CVE-2022-46689 | 13 | 1,717 | 7.0 |  | 0.004670000 | 0.728290000 | macOS における権限昇格される脆弱性 |
|  2位 | 1,543 | 2023/08/23 | CVE-2023-38831 | 38 | 1,163 | 7.8 | ✅ | 0.234040000 | 0.961100000 | RARLAB WinRAR のZIPファイル閲覧時に任意のコード実行となる脆弱性 |
|  3位 | 998 | 2023/03/14 | CVE-2023-23397 | 25 | 748 | <span style="color: #ff0000">9.8</span> | ✅ | <span style="color: #ff0000">0.892850000</span> | 0.984770000 | Windows社の Microsoft Outlook における特権昇格の脆弱性 |
|  4位 | 795 | 2023/02/06 | CVE-2022-44268 | 23 | 565 | 6.5 |  | 0.013800000 | 0.848370000 | ImageMagick の検証不備により任意のファイルが読み取り可能となる脆弱性 |
|  5位 | 770 | 2023/01/10 | CVE-2023-21768 | 9 | 680 | 7.8 |  | 0.016680000 | 0.862570000 | Microsoft社の WinSock 用 Windows Ancillary Function Driver に権限昇格の脆弱性 |
|  6位 | 732 | 2023/10/03 | CVE-2023-4911 | 13 | 602 | 7.8 | ✅  | 0.018070000 | 0.867950000 | Linuxの標準Cライブラリ（GNU Cライブラリ）におけるバッファオーバーフローの脆弱性 |
|  7位 | 686 | 2023/05/15 | CVE-2023-32784 | 8 | 606 | 7.5 |  | 0.001040000 | 0.422090000 | パスワード管理ツール「KeePass」にメモリダンプでマスターパスワードを復元できる脆弱性 |
|  8位 | 605 | 2023/03/22 | CVE-2023-0386 | 8 | 525 | 7.8 |  | 0.000420000 | 0.057550000 | Linux kernel の権限昇格の可能性の脆弱性 |
|  9位 | 574 | 2023/07/19 | CVE-2023-3519 | 14 | 434 | <span style="color: #ff0000">9.8</span> | ✅ | <span style="color:#ff0000;">0.890420000</span> | 0.984580000 | Citrix社のネットワーク機器 Citrix ADC および Citrix Gateway に認証不要のRCEの脆弱性 |
| 10位 | 573 | 2023/10/04 | CVE-2023-22515 | 18 | 393 | <span style="color: #ff0000">9.8</span> | ✅ | <span style="color: #ff0000">0.955290000</span> | 0.992300000 | Atlassian社の Confluence Data Center および Server におけるアクセス制御破損の脆弱性 |

</section>

2024年にはどのような脆弱性が出てくるのか楽しみです。  
では良いお年を！！

[blog:g:12921228815726579926:banner]
