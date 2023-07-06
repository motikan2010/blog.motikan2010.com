<div style="text-align:center;">[f:id:motikan2010:20230706142618p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## TL;DR



- <span style="font-size: 120%">**「Ultimate Member」が導入されている 1,168 サイトを調査対象**</span>
- <span style="font-size: 120%">**脆弱性が修正されているバージョン(2.6.7)を利用しているサイトは 約40%**</span>
- <span style="font-size: 120%">**攻撃を受けた痕跡は確認できず**</span>

## 脆弱性の概要

<div style="text-align:center;">[f:id:motikan2010:20230706163302p:plain:w500]</div>

　WordPress プラグインである「**Ultimate Member – User Profile, Registration, Login, Member Directory, Content Restriction & Membership Plugin**」の脆弱性です。

　アクティブインストール数は 20万以上であり人気のあるプラグインということもあり本脆弱性は警戒されています。  

　脆弱性に関する情報は以下のサイトが分かりやすいです。2023年7月6日時点でWPScanからのPoCは公開されておらずで2023年8月1日に公開する予定らしい。  

[https://ja.wordpress.org/plugins/ultimate-member/:embed:cite]  

[https://wpscan.com/vulnerability/694235c7-4469-4ffd-a722-9225b19e98d7:title]  

> 　このプラグインは、訪問者が任意の機能を持つユーザーアカウントを作成することを防止しないため、事実上、攻撃者が自由に管理者アカウントを作成することを許してしまう。
　この脆弱性は実際に悪用されています。

　CVE IDは「CVE-2023-3460」が採番されている。

### PoC

　WPScan からのPoCの公開はありませんが、GitHubでは既に公開されています。  

[https://github.com/gbrsh/CVE-2023-3460:embed:cite]

　ちなみに後述する調査はこのPoCが公開される前に実施しています。  

### 脆弱性の流れ

- 2023-06-04 : 本脆弱性を悪用した攻撃成功を観測 （Pressable.com / WP.cloud の監視システム）
- 2023-06-26 : Slavic Dragovtev が Ultimate Member に特権昇格の脆弱性を報告
- 2023-06-27 : バージョン 2.6.4 リリース  （まだ脆弱性有り）
- 2023-06-27 : 一部のプラグインユーザが自分のサイトに攻撃を受けたとフォーラムに投稿
- 2023-06-28 : バージョン 2.6.5 リリース （まだ脆弱性有り）
- 2023-06-29 : WPScan がブログを投稿 → **脆弱性がパブリック**になる
- 2023-07-01 : バージョン 2.6.7 リリース （脆弱性修正済み）

「Hacking Campaign Actively Exploiting Ultimate Member Plugin - WPScan WordPress Security」からの抜粋。  
[https://blog.wpscan.com/hacking-campaign-actively-exploiting-ultimate-member-plugin/:embed:cite]  


◆ 攻撃を受けた旨のフォーラム  

[https://wordpress.org/support/topic/register-role-ignored-and-user-became-an-admin/:title]

## 調査

　日本国内でのこのプラグインの普及度や利用しているバージョン情報などを確認していきます。  

### 利用バージョンの確認

#### 約80万(80,3632)の WordPress サイトを確認

　まずは、約80万の WordPress サイトに対して Ultimate Member が導入されているサイトを絞り込みます。  
（プラグインの有効・無効は関係なし）

<span style="color: #ff0000">結果： 1,168 サイトで Ultimate Member が導入されていることを確認</span>

#### 1,168 サイトの Ultimate Member のバージョンを確認

　調査対象で利用されていた Ultimate Member のバージョンの割合は以下の結果なりました。  

[f:id:motikan2010:20230706203320p:plain:w600]  

　<span style="color: #ff0000">本脆弱性が修正されているバージョンは 2.6.7 であり、約40%の環境で導入されており最も多い結果となっています。</span>  

　しかし、残りの約60%は脆弱性が未修正のバージョンを利用している結果となっています。  
（前提として バージョン2.6.7 未満には脆弱性あるとしています。古すぎるバージョンには脆弱性がない場合もあります。）  

##### 導入数とバージョンの全体

<div class="md-code" style="width:50%">

```
導入数	バージョン
476	2.6.7
5	2.6.6
75	2.6.5
1	2.6.4
3	2.6.3
4	2.6.2
16	2.6.0
3	2.5.4
6	2.5.3
4	2.5.1
4	2.5.0
18	2.4.2
17	2.4.1
3	2.4.0
19	2.3.2
36	2.3.1
42	2.3.0
42	2.2.5
19	2.2.4
3	2.2.3
8	2.2.2
4	2.2.0
3	2.1.9
7	2.1.8
13	2.1.7
33	2.1.6
32	2.1.5
6	2.1.4
10	2.1.3
15	2.1.21
19	2.1.20
8	2.1.2
10	2.1.19
5	2.1.17
28	2.1.16
47	2.1.15
10	2.1.13
12	2.1.12
10	2.1.11
5	2.1.10
3	2.1.1
2	2.1.0
13	2.0.56
1	2.0.55
1	2.0.54
1	2.0.53
1	2.0.52
1	2.0.51
9	2.0.49
3	2.0.48
1	2.0.47
5	2.0.43
1	2.0.41
1	2.0.40
5	2.0.39
9	2.0.38
1	2.0.37
4	2.0.35
3	2.0.33
4	2.0.29
3	2.0.25
3	2.0.21
2	2.0.17
1	1.3.89
7	1.3.88
1	1.3.87
1	1.3.86
```

</div>

### バックドア用アカウントの確認

　本脆弱性は権限不要で攻撃対象サイトに管理者権限アカウントを追加できるというものです。  

　被害を受けた際には以下の名前のアカウントが作成されることが確認されているようです。  

- apadmin
- wpenginer
- wpadmins
- wpengine_backup
- se_brutal
- segs_brutal

　そこで Ultimate Member のバージョン関係なくこのプラグインを利用しているサイトのユーザ一覧を取得し、これらのアカウントが存在するのかを確認します。  

　もしこれらのアカウントが存在する場合はサイトが被害を受けている可能性があるということになります。  
（あくまで調査時のアカウント状態であり、被害後にアカウントを削除している可能もあります。）  

　以下のように 2,948 アカウントを取得することができました。

　しかし、その中に上記6つの<span style="color: #ff0000">バックドア用アカウントは存在していませんでした。</span>  

[f:id:motikan2010:20230706203408p:plain:w600]  

## まとめ

- 脆弱性が修正されているバージョンを利用しているサイトは 約40% と少な目
- 確認できた範囲では攻撃を受けた被害はない（不正なアカウントやサイトの改ざん など）
- とはいえ、まだ PoC は公開されていない状態のため、公開後に攻撃が増える可能性がある

　執筆時のPoCが公開されたので、後日また調査したいと思います。  

## 参考

- NVD - CVE-2023-3460  
https://nvd.nist.gov/vuln/detail/CVE-2023-3460
- timate Member < 2.6.7 - Unauthenticated Privilege Escalation WordPress Security Vulnerability  
https://wpscan.com/vulnerability/694235c7-4469-4ffd-a722-9225b19e98d7
- Hacking Campaign Actively Exploiting Ultimate Member Plugin - WPScan WordPress Security  
https://blog.wpscan.com/hacking-campaign-actively-exploiting-ultimate-member-plugin/
