先日、『生つべ』というWebサービス開発しました。リリースをしたのはよいが「完成!」と言えるような状態になるのは、まだまだ時間が掛かりそう。   
どのようなサービスなのかを簡単に説明をすると「YouTubeの動画を皆で見ながらチャットしようよ」というもの。

[http://namatube.motikan2010.com/:embed:cite]



ソースコードも公開してある。  


[https://github.com/motikan2010/NamaTube:embed:cite]



## 利用している技術・API群の紹介
### 認証
このサービスでは、動画の登録にはログインすることが必要です。  
　「Twitter」と「GitHub」アカウントを用いた**OAuth認証方式のみ**をサポートしています。
 
 サービス内だけの認証(ログインIDとパスワードを登録)も用意しようか迷いましたが、最近のWebサービスでは、認証方法にOAuth認証のみというのを目にするようになっているので、今回はOAuth認証のみとしました。（2つのサービスのOAuth認証は少ないと思われるが・・・）

この記事通りにやったらできました。  


[https://qiita.com/Hassan/items/176bc2c6fd75a3e00111:title]



◆ なぜGitHubアカウント？  
　Twitterアカウントは多くの人が持っているので、認証方法として適していると思うが、なぜこのサービスに**持っている人が限られているであろうGitHubアカウント**を用いているかというと、こんな経緯があったりします。  
　サービス開発の当初は、技術者向けのサービスとして開発しようと考えいました。具体的には、YouTubeをはじめとするニコニコ動画などといった動画配信サービスに登録されている技術系の動画を１つのサイトにまとめ、検索・評価できるようなサービス。  の予定だったが、途中で微妙...と感じ断念\_(:3 」∠ )_ 。

もともとは「Tech TV」というサービス名だったという名残があります。  


[https://github.com/motikan2010/NamaTube/commit/714a89f5de7c3e63d3fa7d242dbb203034857c46#diff-9599427925097c3c66f26ac1e0de5cad:title]



### チャット
[f:id:motikan2010:20180712213852p:plain]  

Rails5になって「**Action Cable**」というWebSocketを簡単に扱えるようになる機能が追加されたので利用しました。



[https://railsguides.jp/action_cable_overview.html:title]



モデルの構成など大枠の作成こちらを参考にしました。


[https://qiita.com/jnchito/items/aec75fab42804287d71b:title]



今回は動画毎にチャットルームが異なるようになっているので、ルーム毎にメッセージを送る方法は下記を参考にしました。


[https://qiita.com/kohei1228/items/7aed5aad9c63e834c0e1:title]




### タグ生成
[f:id:motikan2010:20180712214102p:plain]
　動画新規登録時のタグ生成の処理は「**Google Natural Language API**」を利用して実現しています。
 その中のエンティティ分析を活用することにより、動画タイトルから**固有名詞のみを抽出**し、タグとして登録しています。


[https://qiita.com/howdy39/items/a1aef86fef1ce1b6d778#entities%E3%82%A8%E3%83%B3%E3%83%86%E3%82%A3%E3%83%86%E3%82%A3%E5%88%86%E6%9E%90:title]



▼ APIを利用しているのは、この辺り  

[https://github.com/motikan2010/NamaTube/blob/20180710/app/controllers/concerns/util/analyze_entity_util.rb:title]




### 動画の再生位置の制御
[f:id:motikan2010:20180712214135p:plain]
動画の埋め込みは「**YouTube Player API**」を利用しています。このAPIを活用することによって、動画ページを開いた時の再生位置を制御できています。


[https://developers.google.com/youtube/iframe_api_reference?hl=ja:title]



動画再生ページを開いた時の時間によって再生位置を決めていますので、このサービスの肝である「みんなが同じ動画を見ている」というはこの部分で実現されています。

▼ この辺りで再生位置の計算・制御を行っている

[https://github.com/motikan2010/NamaTube/blob/20180710/app/assets/javascripts/videos.js:title]



### 動画のタイトル・再生時間の取得
動画のタイトルや長さを取得は「**YouTube Data API**」を利用しています。


[https://developers.google.com/youtube/v3/docs/?hl=ja:title]



▼ APIを利用しているのは、この辺り

[https://github.com/motikan2010/NamaTube/blob/20180710/app/controllers/concerns/util/youtube_api_util.rb:title]



### 動的なフロント部分
　動的なUIを実現するために「jQuery」と「React」で迷ったのだが、保守的なことを考えてReactを採用することにしました。JSXの方がタグ構造を直感的に把握しやすく、書き直しが簡単だと感じているので。
　フロントに関しては、ほぼ知識がないようなものなので、とりあえず動くのを目指して実装。
 
#### 新規登録画面
[f:id:motikan2010:20180712215519p:plain]  

まずはこっちの方をReactで実装した。  
▼ソースコード
[https://github.com/motikan2010/NamaTube/blob/20180710/front/src/videos/new.js:title]


#### 編集画面
[f:id:motikan2010:20180712215534p:plain]  
　最初は「React DnD」というライブラリを利用して、ドラッグ&ドロップで上下移動させることを考えていたのだが、想像よりも実装に時間が掛かりそうだったので断念。  
　結局、ボタン押下で上下移動させるようにした。


[http://react-dnd.github.io/react-dnd/:title]

▼ソースコード  

[https://github.com/motikan2010/NamaTube/blob/master/front/src/videos/edit.js:title]



## 今後(TODO)
▲優先度高

- トップページ(ランディングページ)の作成
- スマホ向けのレイアウト
- OAuth認証の対象を増やす
- HTTPS化
- マイページの拡充
- アプリ固有のエラー画面の作成
- React(フロント)側のリファクタリング
- MySQLへの移行

▼優先度低

