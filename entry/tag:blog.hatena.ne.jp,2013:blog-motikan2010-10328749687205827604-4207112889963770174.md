<div style="text-align:center;">[f:id:motikan2010:20230220015743p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　国内のWordPressで構築されている 約26万サイトを対象に調査しました。  
(※ 厳密には 260,135 サイト分)


　**今回調査した内容は2つあります。**

- 調査①：「初期ログイン画面へのアクセス」が可能か
- 調査②：「アカウントの列挙」が可能か

　**調査前は特に以下の2点が気になっていました。**  

- 調査①と調査②の割合に差があるか  
→ （予想）セキュリティプラグインを導入すると両方とも問題なくなる場合が多いので、割合の差は少なそう。
- トップドメイン別で割合に差があるか  
→ （予想）政府が管理している「go.jp」や企業のみが取得できる「co.jp」は問題が少なそう。

## 調査①：初期ログイン画面へのアクセス（/wp-login.php）

### 調査内容

　WordPressの管理画面へアクセスするためのURLがデフォルト（`/wp-login.php`）になっているかを確認しています。

　管理画面がデフォルトのURLである場合は下画像のように「`<WordPressインストール先>/wp-login.php`」でログイン画面にアクセスできます。  

　**この状態だと誰でもがログイン画面にアクセスできることにより、不正なログイン試行が行われるといった可能性があります。**

<div style="text-align:center;">[f:id:motikan2010:20230216211846p:plain:w600]</div>

（※ログイン画面にアクセスできるが、認証のリクエストは拒否されるサイトも含まれます。）

### 調査結果

　最初に全体の数を見てみます。

　調査対象である**<span style="color: #ff0000">全260,135サイト中、204,695サイト</span>**がデフォルトのURLで管理者のログイン画面にアクセスすることができました。

<figure class="figure-image figure-image-fotolife" title="ログイン画面にアクセス可能か">[f:id:motikan2010:20230218215712p:plain:w500]<figcaption>ログイン画面にアクセス可能か</figcaption></figure>

　次にドメイン別で見てみます。検査数が20以上のものを取り上げています。  
（黄色に塗りつぶされているドメインは属性JPドメインであり、取得するには特定に機関に必要なドメインです。）

　割合の差が最大は2倍ほどありますが、ドメインの特徴での差はあまり確認できませんでした。  

　「go.jp」ドメインが一番対応されているドメインであることがわかりますが、それでも半数近くが対応していないのは意外でした。  

<div style="text-align:center;">[f:id:motikan2010:20230220013024p:plain:w700]</div>

## 調査②：アカウントの列挙（/wp-json/wp/v2/users）

### 調査内容

　「`<WordPressインストール先>/wp-json/wp/v2/users`」にアクセスしてアカウント列挙ができるかを確認してみます。

　アカウント列挙の機能が有効になっていると下画像のように容易にアカウント情報（認証情報の一部）が取得できます。  

<div style="text-align:center;">[f:id:motikan2010:20230216211850p:w600]</div>

　アカウントが列挙できるという状態は、調査①の初期ログイン画面へのアクセスが可能よりも深刻であり、Tenable社のスキャナーでは脆弱性として検出されるものになっています。  

[https://jp.tenable.com/plugins/was/98203:title]

### 調査結果

　調査対象である**<span style="color: #ff0000">全260,135サイト中、134,373サイト</span>**がアカウントの列挙が可能になっていました。  

　調査①よりも問題のあるサイトの割合が少なくなっていることを確認できます。  

<figure class="figure-image figure-image-fotolife" title="アカウントの列挙が有効か">[f:id:motikan2010:20230218215709p:plain:w500]<figcaption>アカウントの列挙が有効か</figcaption></figure>

　次にドメイン別で見てみます。検査数が20以上のものを取り上げています。

　<span style="color: #ff0000">属性JPドメインのサイトの対応割合が多い点は興味深い結果となりました。</span>

<div style="text-align:center;">[f:id:motikan2010:20230220013015p:plain:w700]</div>

## まとめ

### 予想と結果

　最初に記載した予想の答え合わせをしていきます。  

- 調査①と調査②の割合に差があるか  
→ （予想）セキュリティプラグインを導入すると両方とも問題なくなる場合が多いので、割合の差は少なそう。
→ （結果）『調査①は約4/3、調査②は約1/2』 
　　　　　アカウント列挙は不要な場合が多いが、ログイン画面に関しては管理者以外も利用するサイトがあるのでその差が出たのでしょうか。

- トップドメイン別で割合に差があるか  
→ （予想）政府が管理している「go.jp」や企業のみが取得できる「co.jp」は問題が少なそう。  
→ （結果）『調査①の差は小さい、調査②の差は大きい』  
　　　　　調査②の問題はより脆弱性として扱われることが多いことが原因なのでしょうか。

### 次回

　次は実際に問題なっているプラグイン関連について調査をしていこうと考えています。

追記：第二弾書きました。⬇︎
[https://blog.motikan2010.com/entry/2023/04/08/26%E4%B8%87%E3%82%B5%E3%82%A4%E3%83%88%E5%88%86%E3%81%AEWordPress%E3%82%92%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E8%AA%BF%E6%9F%BB%E3%80%8C%E8%84%86%E5%BC%B1%E3%81%AA%E3%83%97%E3%83%A9:embed:cite]


[blog:g:12921228815726579926:banner]
