<div style="text-align:center;">
[f:id:motikan2010:20230408202451p:plain:w600]
</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　国内のWordPressで構築されている 約26万サイトを対象に調査しました。(※ 厳密には 260,135 サイト分)

　今回は第二弾は「脆弱なプラグイン 編」ということで、WordPressに導入されている既知の脆弱性があるプラグインに着目して調査を行いました。

　前回の記事は以下のものです。
[https://blog.motikan2010.com/entry/2023/02/20/26%E4%B8%87%E3%82%B5%E3%82%A4%E3%83%88%E5%88%86%E3%81%AEWordPress%E3%82%92%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E8%AA%BF%E6%9F%BB%E3%80%8C%E3%83%87%E3%83%95%E3%82%A9%E3%83%AB%E3%83%88:embed:cite]


### 調査内容

　調査対象としたプラグインは、私が運用しているハニーポットに対してプラグインの有無を確認するスキャンの多さ上位のものを対象にしています。  

　そして調査対象の脆弱性は、当プラグインで特に目立つ脆弱性の有無を確認しています。  

　具体的には以下4種類のプラグインと脆弱性が調査対象です。

| プラグイン | 脆弱性 |
|-|-|
| File Manager | Arbitrary File Upload/Remote Code Execution (CVE-2020-25213) |
| InfiniteWP Client | Authentication Bypass (CVE-2020-8772) |
| bbPress | Unauthenticated Privilege Escalation (CVE-2020-13693) |
| Fancy Product Designer | Unauthenticated Arbitrary File Upload (CVE-2021-24370) |

　**最終的には 260,135サイト中の「プラグイン導入数」と「脆弱性有りプラグイン導入数」を出力しています。**

## 調査結果

### File Manager <= 6.8 - Arbitrary File Upload/Remote Code Execution (CVE-2020-25213)

|| サイト数 |
|-|-:|
| プラグイン導入数 | 2,688 |
| 脆弱性有りプラグイン導入数 | 11 |

<figure class="figure-image figure-image-fotolife" title="File Manager &lt;= 6.8 - Arbitrary File Upload/Remote Code Execution - Wordfence">[f:id:motikan2010:20230408153445p:plain:w600]<figcaption>File Manager &lt;= 6.8 - Arbitrary File Upload/Remote Code Execution - Wordfence</figcaption></figure>

#### プラグインのバージョン

[f:id:motikan2010:20230408151737p:plain:w600]

| 導入数 | バージョン | 脆弱 | | 導入数 | バージョン | 脆弱 | | 導入数 | バージョン | 脆弱 |
|-:|-|:-:|-|-:|-|:-:|-:|-:|-|:-:|
| 413 | 7.1.6 |  | | 14 | 5.4 |  | | 1 | 6.7 | ○ |
| 405 | 7.1.8 |  | | 13 | 1.9 |  | | 1 | 6.2 | ○ |
| 369 | 7.1.7 |  | | 12 | 7 |  | | 1 | 4 |  |
| 273 | 7.1.2 |  | | 12 | 4.1 |  | | 1 | 3.4 |  |
| 181 | 7.1.1 |  | | 12 | 3.2 |  | | 1 | 2.9 |  |
| 176 | 7.1.5 |  | | 12 | 2.8 |  | | 1 | 2.7 |  |
| 173 | 不明 |  | | 10 | 1.7 |  | | 1 | 2.2 |  |
| 149 | 7.1.4 |  | | 9 | 1.8 |  | | 1 | 1.1 |  |
| 54 | 7.1 |  | | 8 | 3.1 |  | | 1 | 1 |  |
| 48 | 6.9 |  | | 7 | 3.8 |  | |  |  |  |
| 42 | 7.1.3 |  | | 7 | 2.4 |  | |  |  |  |
| 41 | 5.3 |  | | 7 | 1.6 |  | |  |  |  |
| 38 | 5.7 |  | | 6 | 3.7 |  | |  |  |  |
| 29 | 4.4 |  | | 5 | 6.4 | ○ | |  |  |  |
| 28 | 5.2 |  | | 4 | 6.5 | ○ | |  |  |  |
| 26 | 5.9 |  | | 4 | 3 |  | |  |  |  |
| 26 | 4.8 |  | | 4 | 2 |  | |  |  |  |
| 24 | 5.5 |  | | 4 | 1.5 |  | |  |  |  |
| 15 | 4.6 |  | | 3 | 2.1 |  | |  |  |  |
| 14 | 5.8 |  | | 2 | 2.6 |  | |  |  |  |

#### 脆弱性の割合

[f:id:motikan2010:20230408151944p:plain:w600]

### InfiniteWP Client <= 1.9.4.4 - Authentication Bypass (CVE-2020-8772)

|| サイト数 |
|-|-:|
| プラグイン導入数 | 233 |
| 脆弱性有りプラグイン導入数 | 8 |

<figure class="figure-image figure-image-fotolife" title="InfiniteWP Client <= 1.9.4.4 - Authentication Bypass - Wordfence">[f:id:motikan2010:20230408154806p:plain:w600]<figcaption>InfiniteWP Client &lt;= 1.9.4.4 - Authentication Bypass - Wordfence</figcaption></figure>

#### プラグインのバージョン

[f:id:motikan2010:20230408154149p:plain:w600]

| 導入数 | バージョン | 脆弱 |
|-|-|-|
| 115 | 1.9.6 |  |
| 63 | 1.11.0 |  |
| 29 | 1.9.4.8.2 |  |
| 7 | 1.9.8 |  |
| 5 | 1.9.4.5 |  |
| 4 | 1.8.5 | ○ |
| 3 | 1.9.9 |  |
| 2 | 1.9.4.11 |  |
| 2 | 1.6.4.2 | ○ |
| 1 | 1.6.6.3 | ○ |
| 1 | 1.6.3.2 | ○ |
| 1 | 不明 |  |

### 脆弱性の割合

[f:id:motikan2010:20230408154313p:plain:w600]

### bbPress <= 2.6.4 - Unauthenticated Privilege Escalation (CVE-2020-13693)

|| サイト数 |
|-|-:|
| プラグイン導入数 | 29 |
| 脆弱性有りプラグイン導入数 | 8 |

<figure class="figure-image figure-image-fotolife" title="bbPress <= 2.6.4 - Unauthenticated Privilege Escalation - Wordfence">[f:id:motikan2010:20230408154632p:plain:w600]<figcaption>bbPress &lt;= 2.6.4 - Unauthenticated Privilege Escalation - Wordfence</figcaption></figure>

#### プラグインのバージョン

[f:id:motikan2010:20230408155045p:plain:w600]

| 導入数 | バージョン | 脆弱 |
|-|-|-|
| 13 | 2.6.9 |  |
| 5 | 2.6.6 |  |
| 3 | 2.6.5 |  |
| 1 | 2.6.4 | ○ |
| 1 | 2.6.3 | ○ |
| 2 | 2.5.14 | ○ |
| 2 | 2.5.12 | ○ |
| 1 | 2.5.11 | ○ |
| 1 | 2.5.8 | ○ |

#### 脆弱性の割合

[f:id:motikan2010:20230408155255p:plain:w600]

### Fancy Product Designer <= 4.6.8 - Unauthenticated Arbitrary File Upload (CVE-2021-24370)

|| サイト数 |
|-|-:|
| プラグイン導入数 | 5 |
| 脆弱性有りプラグイン導入数 | ? |

#### プラグインのバージョン

<figure class="figure-image figure-image-fotolife" title="Fancy Product Designer <= 4.6.8 - Unauthenticated Arbitrary File Upload - Wordfence">[f:id:motikan2010:20230408160435p:plain:w600]<figcaption>Fancy Product Designer &lt;= 4.6.8 - Unauthenticated Arbitrary File Upload - Wordfence</figcaption></figure>

| 導入数 | バージョン | 脆弱 |
|-|:-|-|
| 5 | 不明 |  |

## まとめ

　WordPressプラグインの脆弱性を悪用されてサイトが改ざんされるような事例が度々上がっており、今回このような調査を行いましたが、結構プラグインがアップデートされているサイトが多いという印象でした。

　調査前は、利用者の 10% 程はプラグインのアップデートをしていないのではと考えていました。  
　ですが、実際は 0.00 数パーセントが対応できていない状況でした。

　本調査ではその点が定量化できたので、良かった点かと。

## 参考

- [WordPress Vulnerability Database](https://www.wordfence.com/threat-intel/vulnerabilities/)


[blog:g:12921228815726579926:banner]
