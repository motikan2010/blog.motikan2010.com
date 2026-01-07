<div style="text-align:center;">[f:id:motikan2010:20260106224411p:plain:w500]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

毎年恒例の「人気脆弱性トップ10（2025年版）」を発表します。  

過去のランキングは下記のリンクから

| 　年代　 | トップ脆弱性 |
| - | - |
| [2024年](https://blog.motikan2010.com/entry/2024/12/30/2024%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub) | 4,522 pt - XZ Utilsに悪意のあるコードが挿入された問題（CVE-2024-3094） |
| [2023年](https://blog.motikan2010.com/entry/2023/12/26/2023%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub) | 1,847 pt - macOS における権限昇格される脆弱性（CVE-2022-46689 / MacDirtyCow） |
| [2022年](https://blog.motikan2010.com/entry/2023/01/05/2022%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub) | 2,924 pt - Linux Kernel における権限昇格される脆弱性（CVE-2022-0847 / Dirty Pipe） |
| [2021年](https://blog.motikan2010.com/entry/2022/01/19/2021%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub) | 15,134 pt - Apache Log4j における任意のコードが実行可能な脆弱性 （CVE-2021-44228 / Log4Shell） |
| [2020年](https://blog.motikan2010.com/entry/2021/01/01/2020%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub) | 3,696 pt - Microsoft SMBv3 の接続処理にリモートコード実行の脆弱性 （CVE-2020-0796） |
| [2019年](https://blog.motikan2010.com/entry/2019/12/31/2019%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub) | 5,413 pt - Microsoft Windows 製品のリモートデスクトップサービスにおけるリモートでコードを実行される脆弱性 (CVE-2019-0708 / BlueKeep) |
| [2018年](https://blog.motikan2010.com/entry/2020/01/06/2018%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub) | 1,242 pt - libssh における認証に関する脆弱性 （CVE-2018-10933） |

### スコアの計算方法

ランキングに利用するスコアの算出方法です。  

- リポジトリ数 × 10pt ・・・ ①
- スター数 × 1pt ・・・ ②

「①＋② の合計pt」でランキングを付けています。

### 対象となる脆弱性

　下記の条件の統計対象の脆弱性となっています。  

  - 「CVE ID が CVE-2024-*」かつ「公表日が 2024/12/01 以降」
  - 「CVE ID が CVE-2025-*」かつ「公表日が 2025/11/30 以前」

## 2025年 人気脆弱性 TOP 10 in GitHub

### 10位 532 pt - Erlang/OTP SSH Server の認証不要のリモートコマンド実行の脆弱性（CVE-2025-32433）

リポジトリ数: **32** / スター数: **192**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>Erlang/OTP は Erlang プログラミング言語向けのライブラリ群</li>
<li>SSH サーバーに脆弱性が存在し、攻撃者が認証なしで遠隔からコードを実行される可能性があります</li>
<li>SSH プロトコルメッセージ処理の脆弱性を悪用されると、攻撃者が認証なしに対象システムに不正アクセスし、有効な認証情報なしで任意のコマンドを実行できる可能性</li>
<li>一時的な回避策として、SSH サーバーを無効化するか、ファイアウォールのルールを設定してアクセスを防止する方法があります</li>
</ul>
</div>


<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】  

- サイバーリーズン | 【脅威分析レポート】CVE-2025-32433 〜Erlang/OTPのSSH実装における脆弱性、悪用されると認証なしでリモートコード実行（RCE）が可能に〜
https://www.cybereason.co.jp/blog/threat-analysis-report/13152/
- Security NEXT | 「Erlang/OTP」に深刻なRCE脆弱性 - 概念実証コードも公開済み
https://www.security-next.com/169684
- PoC | platsecurity/CVE-2025-32433  
https://github.com/platsecurity/CVE-2025-32433
</section>

###  9位 535 pt - Vite で任意ファイルの読み取りが可能な脆弱性（CVE-2025-30208）

リポジトリ数: **24** / スター数: **295**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>任意のファイル読み取りの脆弱性</li>
<li>影響を受けるのは、"--host"または"server.host"設定オプションを使用して Vite 開発サーバーをネットワークに明示的に公開しているアプリケーション</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- IoT OT Security News | Vite の脆弱性 CVE-2025-30208 が FIX：任意のファイル読み取りの PoC も登場  
https://iototsecnews.jp/2025/03/27/millions-at-risk-poc-exploit-releases-for-vite-arbitrary-file-read-flaw-cve-2025-30208/
- Offsec | CVE-2025-30208 – Vite Arbitrary File Read via @fs Path Traversal Bypass  
https://www.offsec.com/blog/cve-2025-30208/
- PoC | ThumpBo/CVE-2025-30208-EXP  
https://github.com/ThumpBo/CVE-2025-30208-EXP
</section>

###  8位 563 pt - Microsoft Windows 製品のLDAPサービスにおけるサービス運用妨害の脆弱性（CVE-2024-49113 / LDAPNightmare）

リポジトリ数: **4** / スター数: **523**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>通称 : LDAPNightmare</li>
<li>LDAP サービスのコンテキスト内で任意コード実行が可能</li>
<li>認証されていない攻撃者が、標的とする Windows マシンに細工された IPv6 パケットを繰り返し送信することで発生させることができる</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- トレンドマイクロ (JP) | CVE-2024-49112およびCVE-2024-49113に関する最新情報
https://www.trendmicro.com/ja_jp/research/25/a/what-we-know-about-cve-2024-49112-and-cve-2024-49113.html
- SafeBreach  | LDAPNightmare: SafeBreach Publishes First PoC Exploit (CVE-2024-49113)
https://www.safebreach.com/blog/ldapnightmare-safebreach-labs-publishes-first-proof-of-concept-exploit-for-cve-2024-49113/
- PoC | SafeBreach-Labs/CVE-2024-49113  
https://github.com/SafeBreach-Labs/CVE-2024-49113
</section>

###  7位 592 pt - RARLAB の WinRAR におけるパストラバーサルの脆弱性（CVE-2025-8088）

リポジトリ数: **27** / スター数: **322**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>Windows版WinRARに存在するパストラバーサル脆弱性を悪用されると、攻撃者が悪意のあるアーカイブファイルを作成される
</li>
<li>任意のコードを実行される可能性</li>
<li>脆弱性は実際に攻撃に利用されている</li>
<li>ESET 社によって発見された</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- ESET | 今すぐアップデートを！WinRARのゼロデイ脆弱性をRomComなどのサイバー攻撃グループが悪用中  
https://www.eset.com/jp/blog/welivesecurity/update-winrar-tools-now-romcom-and-others-exploiting-zero-day-vulnerability-jp/  
- SOC Prime | CVE-2025-8088 検出: WinRARゼロデイがロムコムマルウェアをインストールするために野放しに活動している  
https://socprime.com/ja/blog/detect-cve-2025-8088-exploitation-for-romcom-delivery/
</section>

###  6位 665 pt - Microsoft Windows File Explorer のスプーフィングの脆弱性（CVE-2025-24071）

リポジトリ数: **19** / スター数: **475**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>権限のない第三者に機密情報が漏洩する脆弱性</li>
<li>ネットワーク経由で不正ななりすまし攻撃が行われる可能性</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- ITmedia エンタープライズ | Windows File Explorerの脆弱性に関するPoCが公開  
https://www.itmedia.co.jp/enterprise/articles/2503/24/news077.html
- PoC | 0x6rss/CVE-2025-24071_PoC  
https://github.com/0x6rss/CVE-2025-24071_PoC
</section>

###  5位 766 pt -  Windows SMB クライアントの権限昇格の脆弱性（CVE-2025-33073）

リポジトリ数: **9** / スター数: **676**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>Windows SMBにおける不適切なアクセス制御機構により、認証済みの攻撃者がネットワーク経由で権限を昇格させる可能性</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- SOC Prime | CVE-2025-33073: Windows SMBクライアントのゼロデイ脆弱性により攻撃者がSYSTEM権限を取得可能  
https://socprime.com/ja/blog/cve-2025-33073-zero-day-vulnerability/
- ITmedia エンタープライズ | Windows SMBに特権昇格リスク「CVE-2025-33073」　PoC公開  
https://www.itmedia.co.jp/enterprise/articles/2506/17/news039.html
- PoC | mverschu/CVE-2025-33073  
https://github.com/mverschu/CVE-2025-33073
</section>

###  4位 888 pt - Apache Tomcat partial PUT におけるリモートコード実行や改ざんの脆弱性（CVE-2025-24813）

リポジトリ数: **49** / スター数: **398**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>セキュリティ上重要なファイルの表示やコンテンツ挿入の可能性</li>
<li>さらに条件によっては、リモートコード実行の可能性</li>
<li>明確な有害なコンテンツを含まないため従来のWAFによる検出が困難</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- Akamai | Apache Tomcat の脆弱性 CVE-2025-24813 の検知と緩和  
https://www.akamai.com/ja/blog/security-research/march-apache-tomcat-path-equivalence-traffic-detections-mitigations
- Security NEXT | 「Apache Tomcat」の脆弱性攻撃が発生 - 「WAF」回避のおそれも  
https://www.security-next.com/168301
- PoC | absholi7ly/POC-CVE-2025-24813  
https://github.com/absholi7ly/POC-CVE-2025-24813
</section>

###  3位 938 pt -  Microsoft SharePoint Server におけるリモートコード実行の脆弱性（CVE-2025-53770）

リポジトリ数: **42** / スター数: **518**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>オンプレミス環境の Microsoft SharePoint Server</li>
<li>Microsoft のクラウドベースサービスである SharePoint Online および Microsoft 365 はこの影響を受けない</li>
<li>Microsoft 社は CVE-2025-53770 に対する攻撃コードが実環境で確認</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- TeamT5 | Microsoft SharePoint ゼロデイ脆弱性（CVE-2025-53770）【TeamT5の提案】  
https://teamt5.org/jp/posts/microsoft-sharepoint-vulnerability-cve-2025-53770/
- Ars Technica | SharePoint vulnerability with 9.8 severity rating under exploit across globe  
https://arstechnica.com/security/2025/07/sharepoint-vulnerability-with-9-8-severity-rating-is-under-exploit-across-the-globe/
- soltanali0/CVE-2025-53770-Exploit  
https://github.com/soltanali0/CVE-2025-53770-Exploit
</section>

###  2位 1,473 pt - Next.js の Middleware 認証バイパスの脆弱性（CVE-2025-29927）

リポジトリ数: **111** / スター数: **363**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>Next.js アプリケーション内でミドルウェアによる認証チェックが行われる場合、そのチェックを回避することが可能</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- Zenn | Next.jsの脆弱性CVE-2025-29927まとめ  
https://zenn.dev/t3tra/articles/c293410c7daf63
- Akamai | Next.js の認証バイパス脆弱性を検知・緩和する  
https://www.akamai.com/ja/blog/security-research/march-authorization-bypass-critical-nextjs-detections-mitigations
- PoC | aydinnyunus/CVE-2025-29927  
https://github.com/aydinnyunus/CVE-2025-29927
</section>

###  1位 1,713 pt - Sudo のローカル権限昇格の脆弱性（CVE-2025-32463）

<div style="text-align:center;">[f:id:motikan2010:20241230115058p:plain:h100]</div>

リポジトリ数: **58** / スター数: **3,942**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>Sudo バージョン 1.9.17p1 以前では、ローカルユーザーがルート権限を取得できてしまう脆弱性</li>
<li>chroot オプション (-R) を悪用する NSS (Name Service Switch) システムの操作により、悪意のライブラリを root 権限で読み込ませことで攻撃が行われる</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- SOC Prime | CVE-2025-32463およびCVE-2025-32462の検出：Sudoのローカル権限昇格の脆弱性がLinux環境を脅かす  
https://socprime.com/ja/blog/cve-2025-32463-and-cve-2025-32462-vulnerabilities/
- Security NEXT | 特権コマンド実行ツール「sudo」に重要度「クリティカル」の脆弱性  
https://www.security-next.com/171873
- PoC | pr0v3rbs/CVE-2025-32463_chwoot  
https://github.com/pr0v3rbs/CVE-2025-32463_chwoot
</section>

## 【番外編】CVE-2025-55182 - React2Shell

　脆弱性公開日が2025年12月以降ということで本ランキングの対象外となった話題となった脆弱性です。  

### 11,005 pt - React Server Components の認証不要のリモートコード実行の脆弱性 （CVE-2025-55182 / React2Shell）

リポジトリ数: **328** / スター数: **7,725**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>通称 React2Shell</li>
<li>攻撃者が細工したHTTPリクエスト（HTTP経由の不正なFlightペイロード）をReact Server Components（RSC）を処理するサーバーに送信することで、認証不要のリモートコード実行につながる可能性</li>
<li>2025年12月3日に公開された</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- JPCERT/CC | React Server Componentsの脆弱性（CVE-2025-55182）について  
https://www.jpcert.or.jp/newsflash/2025120501.html
- piyolog | React Server Componentsの脆弱性 CVE-2025-55182（React2Shell）についてまとめてみた。  
https://piyolog.hatenadiary.jp/entry/2025/12/08/113316
</section>

## まとめ

2025年の人気脆弱性は以下のようになりました。  
<section style="overflow-x: scroll;white-space: nowrap;">

|  | CVE ID | スコア |  | 公表日 | リポ数 | スター合計 |
| -: | :- | -: | - | - | -: | -: |
|  1位 | CVE-2025-32463 | 1,713 | Sudo のローカル権限昇格の脆弱性 | 2025-06-30 | 62 | 1093 |
|  2位 | CVE-2025-29927 | 1,473 | Next.js の Middleware 認証バイパスの脆弱性 | 2025-03-21 | 111 | 363 |
|  3位 | CVE-2025-53770 | 938 | Microsoft SharePoint Server におけるリモートコード実行の脆弱性 | 2025-07-20 | 42 | 518 |
|  4位 | CVE-2025-24813 | 888 | Apache Tomcat partial PUT におけるリモートコード実行や改ざんの脆弱性 | 2025-03-10 | 49 | 398 |
|  5位 | CVE-2025-33073 | 766 | Windows SMB クライアントの権限昇格の脆弱性 | 2025-06-10 | 9 | 676 |
|  6位 | CVE-2025-24071 | 665 | Microsoft Windows File Explorer のスプーフィングの脆弱性 | 2025-03-11 | 19 | 475 |
|  7位 | CVE-2025-8088 | 592 | RARLAB の WinRAR におけるパストラバーサルの脆弱性 | 2025-08-08 | 27 | 322 |
|  8位 | CVE-2024-49113 | 563 | Microsoft Windows 製品のLDAPサービスにおけるサービス運用妨害の脆弱性 | 2024-12-10 | 4 | 523 |
|  9位 | CVE-2025-30208 | 535 | Vite で任意ファイルの読み取りが可能な脆弱性 | 2025-03-24 | 24 | 295 |
|  10位 | CVE-2025-32433 | 532 | Erlang/OTP SSH Server の認証不要のリモートコマンド実行の脆弱性 | 2025-04-16 | 34 | 192 |

</section>

[blog:g:12921228815726579926:banner]
