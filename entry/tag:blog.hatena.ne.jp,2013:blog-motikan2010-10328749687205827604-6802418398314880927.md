<div style="text-align:center;">[f:id:motikan2010:20250106215806p:plain:w500]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

2024年も終わりが近づいてきたということで、毎年恒例の「人気脆弱性トップ10（2024年版）」を発表します。  

過去のランキングはコチラから

- [2023年](https://blog.motikan2010.com/entry/2023/12/26/2023%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)
- [2022年](https://blog.motikan2010.com/entry/2023/01/05/2022%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)
- [2021年](https://blog.motikan2010.com/entry/2022/01/19/2021%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)
- [2020年](https://blog.motikan2010.com/entry/2021/01/01/2020%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)
- [2019年](https://blog.motikan2010.com/entry/2019/12/31/2019%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)
- [2018年](https://blog.motikan2010.com/entry/2020/01/06/2018%E5%B9%B4_%E4%BA%BA%E6%B0%97%E8%84%86%E5%BC%B1%E6%80%A7_TOP_10_in_GitHub)


### スコアの計算方法

ランキングに利用するスコアの算出方法です。  

- リポジトリ毎に 10pt … ①
- スター毎に 1pt … ②

「①＋② の合計pt」でランキングを付けています。


### 対象となる脆弱性

  - 「CVE-2023-*」かつ「公表日が 2023/12/01 以降」
  - 「CVE-2024-*」かつ「公表日が 2024/11/30 以前」


## 2024年 人気脆弱性 TOP 10 in GitHub

### 10位 777 pt - Windows カーネルにおける権限昇格される脆弱性（CVE-2024-30088）

<div style="text-align:center;">[f:id:motikan2010:20241230121554j:plain:h100]</div>

リポジトリ数: **6** / スター数: **717**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>権限昇格が生じる脆弱性</li>
<li>「Windows カーネル」において、「TOCTOU（Time-of-check Time-of-use）」による競合状態に起因</li>
<li>CISA KEV に追加</li>
</ul>
</div>

[https://www.youtube.com/watch?v=y5LnlHjzA64:embed:cite]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- 【セキュリティ ニュース】米当局、「SolarWinds WHD」や「Windows」など脆弱性3件の悪用に注意喚起：Security NEXT  
https://www.security-next.com/162995
- [PoC] exploits-forsale/collateral-damage  
https://github.com/exploits-forsale/collateral-damage
</section>

###  9位 897 pt - Jenkins の任意のファイル読み取りの脆弱性（CVE-2024-23897）

<div style="text-align:center;">[f:id:motikan2010:20241230121740p:plain:h100]</div>

リポジトリ数: **34** / スター数: **557**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>任意のファイル読み取りの脆弱性</li>
<li>Jenkins にはスクリプトまたはシェル環境からJenkinsへアクセスするコマンドラインインターフェースの機能 (Jenkins CLI)が組み込まれている。このCLIを介して任意のファイル読み取りが行える</li>
<li>特定の条件下においてリモートコード実行が可能となる恐れがある</li>
<li>CISA KEV に追加</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- Jenkinsの脆弱性 CVE-2024-23897 についてまとめてみた - piyolog  
https://piyolog.hatenadiary.jp/entry/2024/01/30/031702
- [PoC] h4x0r-dz/CVE-2024-23897  
https://github.com/h4x0r-dz/CVE-2024-23897
</section>

###  8位 1,094 pt -  Windows TCP/IP におけるリモートでコードが実行される脆弱性（CVE-2024-38063）

<div style="text-align:center;">[f:id:motikan2010:20241230121554j:plain:h100]</div>

リポジトリ数: **29** / スター数: **804**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>リモートコード実行の脆弱性</li>
<li>認証されていない攻撃者が、標的とする Windows マシンに細工された IPv6 パケットを繰り返し送信することで発生させることができる</li>
<li>IPv6 が無効にされたシステムは、この脆弱性の影響を受けない</li>
<li>Cisco Talos が発見した</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- システム権限を取得される可能性のある Microsoft カーネルモードドライバの脆弱性を Talos が発見、その他に公開された重大な問題は 7 件 - Cisco Japan Blog
https://gblogs.cisco.com/jp/2024/08/talos-microsoft-patch-tuesday-august-2024/
- [PoC] ynwarcs/CVE-2024-38063  
https://github.com/ynwarcs/CVE-2024-38063
</section>


###  7位 1,120 pt -  Microsoft Office Outlook におけるリモートでコードが実行される脆弱性（CVE-2024-21413）

<div style="text-align:center;">[f:id:motikan2010:20241230122234p:plain:h100]</div>

リポジトリ数: **16** / スター数: **960**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>リモートコード実行の脆弱性</li>
<li>脆弱なエディションを用いて悪意のリンクを含む電子メールを開くと発生する</li>
<li>Check Point の脆弱性研究者である Haifei Li によって発見</li>
</ul>
</div>

![CVE-2024-21413-RCE](https://github.com/xaitax/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability/assets/5014849/cd0dbae7-aaec-4532-9114-b58239fe5775)

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- Microsoft Outlook の脆弱性 CVE-2024-21413：PoC エクスプロイトが GitHub で公開 – IoT OT Security News  
https://iototsecnews.jp/2024/02/16/poc-exploit-released-for-microsoft-outlook-rce-flaw-cve-2024-21413/
- [PoC] xaitax/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability  
https://github.com/xaitax/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability
</section>


###  6位 1,204 pt -  PHP におけるリモートでコードが実行される脆弱性（CVE-2024-4577）

<div style="text-align:center;">[f:id:motikan2010:20241230122312p:plain]</div>

リポジトリ数: **54** / スター数: **664**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>リモートコード実行の脆弱性</li>
<li>特定のロケール設定（中国語繁体字(Code Page 950)/簡体字(Code Page 936)、日本語(Code Page 932））で動作するWindows 環境では攻撃が可能であることが確認</li>
<li>IPAによると、国内の複数組織においてこの脆弱性が悪用され、当該製品が稼働する Web サービスに webshell が設置されていたとの指摘があり</li>
<li>セキュリティコンサルティング会社 DEVCORE の研究者によって発見</li>
<li>CISA KEV に追加</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- PHPの脆弱性CVE-2024-4577 について｜テクニカルブログ｜日本情報通信株式会社  
https://www.niandc.co.jp/tech/20240729_52897/
- PHPの脆弱性（CVE-2024-4577）を狙う攻撃について | 情報セキュリティ | IPA 独立行政法人 情報処理推進機構  
https://www.ipa.go.jp/security/security-alert/2024/alert_20240705.html
- [PoC] watchtowrlabs/CVE-2024-4577  
https://github.com/watchtowrlabs/CVE-2024-4577
</section>

###  5位 1,319 pt -  Git におけるリモートでコードが実行される脆弱性（CVE-2024-32002）

<div style="text-align:center;">[f:id:motikan2010:20241230122449p:plain]</div>

リポジトリ数: **64** / スター数: **679**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>セキュリティ研究者の Amar Murali が発見した</li>
<li>リモートコード実行の脆弱性</li>
<li>クローン作成中の Git のサブモジュールの扱い方のに起因するものであり、悪用に成功した攻撃者は、リモートから任意のコードを実行する</li>
</ul>
</div>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- Git の深刻な脆弱性 CVE-2024-32002 などが FIX：ただちにパッチを！ – IoT OT Security News  
https://iototsecnews.jp/2024/05/15/git-patches-critical-rce-vulnerabilities-cve-2024-32002-cve-2024-32004/
- Gitの重大な脆弱性CVE-2024-32002：RCE用PoCエクスプロイトを研究者らが公開 | Codebook｜Security News  
https://codebook.machinarecord.com/threatreport/32948/
- [PoC] amalmurali47/git_rce  
https://github.com/amalmurali47/git_rce
</section>

###  4位 1,405 pt -  Bluetooth スタックに認証バイパスの脆弱性（CVE-2023-45866）

<div style="text-align:center;">[f:id:motikan2010:20241230122537p:plain]</div>

リポジトリ数: **8** / スター数: **1,325**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>攻撃者はユーザー確認なしに検出可能なホストに接続し、キー入力を行うことが可能</li>
</ul>
</div>

本脆弱性のユースケース例：  
<figure class="figure-image figure-image-fotolife" title="引用: 永島さんの家に攻撃してみた in 明星大学 #初心者 - Qiita">[f:id:motikan2010:20241230123206p:plain:w600]<figcaption>引用: 永島さんの家に攻撃してみた in 明星大学 #初心者 - Qiita</figcaption></figure>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- reblog/cve-2023-45866 at main · skysafe/reblog  
https://github.com/skysafe/reblog/tree/main/cve-2023-45866
- 永島さんの家に攻撃してみた in 明星大学 #初心者 - Qiita  
https://qiita.com/Perplex/items/5549b7c961ab5e6ec301
- [PoC] pentestfunctions/BlueDucky  
https://github.com/pentestfunctions/BlueDucky
</section>

###  3位 2,390 pt -  Linux Kernel における権限昇格の脆弱性（CVE-2024-1086）

<div style="text-align:center;">[f:id:motikan2010:20241230115058p:plain:h100]</div>

リポジトリ数: **7** / スター数: **2,320**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>ローカル環境において脆弱性を悪用すると権限の昇格が可能で、root権限を取得することが可能</li>
<li>コンポーネント「nf_tables」に解放後のメモリを使用するいわゆる「Use After Free」の脆弱性</li>
<li>カーネルをクラッシュさせたり、カーネル内で任意のコードを実行したりする恐れがある</li>
<li>CISA KEV に追加</li>
</ul>
</div>

Wikipedia のページが作成されるほど有名な脆弱性  
[f:id:motikan2010:20241230123001p:plain:w600]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- 米CISA、Linuxカーネルの脆弱性が悪用されていると警告：CVE-2024-1086 | Codebook｜Security News  
https://codebook.machinarecord.com/threatreport/33285/
- 【セキュリティ ニュース】Linuxカーネルに判明した権限昇格の脆弱性 - 詳細やPoCが公開：Security NEXT  
https://www.security-next.com/155568
- [PoC] Notselwyn/CVE-2024-1086  
https://github.com/Notselwyn/CVE-2024-1086
</section>

###  2位 2,800 pt - OpenSSH サーバ（sshd）におけるリモートでコードが実行される脆弱性（CVE-2024-6387：通称「regreSSHion」）

<div style="text-align:center;">[f:id:motikan2010:20241230122633p:plain:h150]</div>

リポジトリ数: **90** / スター数: **1,900**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>リモートコード実行の脆弱性</li>
<li>OpenSSHサーバーのシグナルハンドラの競合により発生します。攻撃者は、設定された時間内に認証に失敗することで、この競合状態を引き起こす可能性
</li>
<li>悪用には長時間の接続と大量の試行が必要であり、実際の攻撃における悪用は現在まで確認されていない</li>
<li>米国のセキュリティベンダー Qualys が発見した</li>
</ul>
</div>

<figure class="figure-image figure-image-fotolife" title="引用: OpenSSHの脆弱性「regreSSHion」（CVE-2024-6387）について | サービス&amp;セキュリティ株式会社">攻撃の流れ  
[f:id:motikan2010:20241230124144p:plain:w600]<figcaption>引用: OpenSSHの脆弱性「regreSSHion」（CVE-2024-6387）について | サービス&amp;セキュリティ株式会社</figcaption></figure>

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- OpenSSHの脆弱性 CVE-2024-6387についてまとめてみた - piyolog  
https://piyolog.hatenadiary.jp/entry/2024/07/02/032122
- OpenSSHの脆弱性「regreSSHion」（CVE-2024-6387）について | サービス&セキュリティ株式会社  
https://www.ssk-kan.co.jp/topics/?p=14546
- [PoC] zgzhang/cve-2024-6387-poc  
https://github.com/zgzhang/cve-2024-6387-poc
</section>

###  1位 4,522 pt -  XZ Utilsに悪意のあるコードが挿入された問題（CVE-2024-3094）

<div style="text-align:center;">[f:id:motikan2010:20241230122712p:plain:h150]</div>

リポジトリ数: **58** / スター数: **3,942**

<div style=" border: 2px solid #90caf3; border-radius: 10px;margin:0.2em;padding:10px;">
<ul>
<li>複数の Linux ディストリビューションなどで利用されているファイル可逆圧縮ツールである XZ Utils に悪意のあるコードが挿入された問題</li>
<li>影響を受けるバージョン：XZ Utils 5.6.0 および 5.6.1</li>
<li>同ツールの共同開発者により2024年2月24日頃に挿入された</li>
<li>特定条件下でSSHポート経由で外部から攻撃者が接続できる可能性がある</li>
</ul>
</div>

日本語の Wikipedia のページが作成されるほど有名な脆弱性  
[f:id:motikan2010:20241230123131p:plain:w600]

<section style=" border: 1px solid gray; border-radius: 10px;margin:0.2em;padding:0.2em;0.5em;">
【参考】

- XZ Utilsに悪意のあるコードが挿入された問題（CVE-2024-3094）について  
https://www.jpcert.or.jp/newsflash/2024040101.html
- XZ Utilsのバックドア - Wikipedia  
https://ja.wikipedia.org/wiki/XZ_Utils%E3%81%AE%E3%83%90%E3%83%83%E3%82%AF%E3%83%89%E3%82%A2
- [PoC] amlweems/xzbot  
https://github.com/amlweems/xzbot
</section>

## まとめ

2024年の人気脆弱性は以下のようになりました。  
<section style="overflow-x: scroll;white-space: nowrap;">

|| スコア | 公表日 | CVE ID | リポ数 | スター合計 |  |
| -: | -: | - | - | -: | -: | - |
|  1位 | 4,522 | 2024/03/29 | CVE-2024-3094 | 58 | 3,942 | XZ Utilsに悪意のあるコードが挿入された問題 |
|  2位 | 2,800 | 2024/07/01 | CVE-2024-6387 | 90 | 1,900 | OpenSSH サーバ（sshd）におけるリモートでコードが実行される脆弱性 |
|  3位 | 2,390 | 2024/01/31 | CVE-2024-1086 | 7 | 2,320 | Linux Kernel における権限昇格の脆弱性 |
|  4位 | 1,405 | 2023/12/08 | CVE-2023-45866 | 8 | 1,325 | Bluetooth スタックに認証バイパスの脆弱性 |
|  5位 | 1,319 | 2024/05/14 | CVE-2024-32002 | 64 | 679 | Git におけるリモートでコードが実行される脆弱性 |
|  6位 | 1,204 | 2024/06/09 | CVE-2024-4577 | 54 | 664 | PHP におけるリモートでコードが実行される脆弱性 |
|  7位 | 1,120 | 2024/02/13 | CVE-2024-21413 | 16 | 960 | Microsoft Office Outlook におけるリモートでコードが実行される脆弱性 |
|  8位 | 1,094 | 2024/08/13 | CVE-2024-38063 | 29 | 804 | Windows TCP/IP におけるリモートでコードが実行される脆弱性 |
|  9位 | 897 | 2024/01/24 | CVE-2024-23897 | 34 | 557 | Jenkins の任意のファイル読み取りの脆弱性 |
| 10位 | 777 | 2024/06/11 | CVE-2024-30088 | 6 | 717 | Windows カーネルにおける権限昇格される脆弱性 |

</section>


[blog:g:12921228815726579926:banner]
