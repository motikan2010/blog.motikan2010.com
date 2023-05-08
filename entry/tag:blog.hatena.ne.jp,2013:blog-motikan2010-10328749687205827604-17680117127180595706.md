<div align="center">[f:id:motikan2010:20190617024001p:plain]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに
　Webサイトの管理画面が推測されることにより、サイトの改ざん等の被害が増えてきているようで、先月(5/9)にEC-CUBEでは下記のような注意喚起がありました。  

　以下、注意喚起の一部抜粋。
> 2. 管理画⾯の URL が /admin/ など<span style="color: #ff0000">推測されやすい URL</span> になっていないか
管理画⾯の URL を変更せず /admin/ のままで運⽤していた場合、攻撃者が管理画⾯にア
クセスしやすい状況になっておりますので、 早急に変更をお願いします。

[【重要】サイト改ざんによるクレジットカード流出被害が増加しています。](https://www.ec-cube.net/user_data/news/201905/security_notice.pdf)

　ここで述べられている推測されやすいURLの代表例として、<span style="color: #ff0000">ツールによって発見されるURLであること</span>があります。  
　なので今回は各ツールがどのようなURL（管理画面へのパス）を検索しているのかを調べてみました。

## 確認したツール

　今回確認したツールは以下の6つです。  

- OWASP ZAP
- Arachni
- Nikto
- skipfish
- Wfuzz
- WAScan

## 最悪な管理画面URLランキング

### 1位 - 全てのツールで検索対象

||
|-|
| admin |
| administrator |

　「admin」「administrator」パスはツール全てで検索されていました。

　EC-CUBEでは、インストール時に管理画面のパス名を指定することができるようになっていますが、「admin」というパス名は指定できないようになっており「admin」がパスとして利用される危険性の配慮がなされていることが分かります。  

　実際に下画像のように「admin」を管理画面のパスに指定すると、エラーメッセージが表示されることが確認できます。  
[f:id:motikan2010:20190616204123p:plain:w500]  

<!-- more -->

### 2位 - 5つのツールで検索対象

||
|-|
| webadmin |
| users |
| user |
| staff |
| portal |
| phpmyadmin |
| manager |
| manage |
| console |

### 3位 - 4つのツールで検索対象

||
|-|
| iisadmin |
| error |
| backend |
| administration |
| administracion |
| admin_ |
| admin.php3 |
| admin.html |
| admin.aspx |
| adm |
| _admin |

### 4位以下 - 1~3つのツールで検索対象

　3つ以下のツールで検索されるパスは多すぎるので、ここには記載せずに下記のところに記載しました。  
[ツールで確認されるパス一覧](https://gist.github.com/motikan2010/5d38e15ee0e20e92748557324c722150)  

## まとめ

　たとえ管理画面にアクセスできても操作するには認証する必要があるから安心だと思われて、外部からのアクセスも許容しているというところもあると思います。しかし、冒頭で紹介したように実際に被害が出てきている以上、管理画面には特定のユーザのみがアクセスできるように制御する必要があります。  
　具体的には、管理画面にはIP制限を設定し、特定のアクセス元以外からはアクセスさせないようにしましょう。  
利用しているアプリケーションによっては、設定できるようなものもあります。

- 各ツールの検索ディレクトリが記述されている調査リポジトリ
[https://github.com/motikan2010/AdminDirectory:title]

## 更新履歴

- 2019年6月19日 新規作成