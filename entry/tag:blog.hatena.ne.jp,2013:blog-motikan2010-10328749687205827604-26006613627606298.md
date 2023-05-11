<div style="text-align:center;">[f:id:motikan2010:20200915214802p:plain]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　先日、こちらのリポジトリのスター数が1,000個を迎えていました。thanks🎉
[https://github.com/nomi-sec/PoC-in-GitHub:embed:cite]  

　そこで、どのようなアカウントがスターを付けているのかという点が気になったので、スターを付けたアカウントのbio(プロフィール)を集めて分析し、最終的にWordCloud(ワードクラウド)を生成してみることにしました。  

　ちなみに WordCloud とは下画像のような、文章から頻出単語を選出し、頻度に応じての目立つように表示された図のことです。  
[f:id:motikan2010:20200915194741p:plain]  

　上記画像は下の文章から生成されています。  
[https://github.com/amueller/word_cloud/blob/master/examples/constitution.txt:title]  

### やること

[f:id:motikan2010:20200915215005p:plain]

1. スターをつけた<b>1004人分のアカウント名を取得</b>
2. 1004人分の<b>プロフィール情報を取得</b>
3. WordCloud生成ライブラリにプロフィール情報を入力
4. 画像の<span><a href="#生成された画像">完成</a></span>

## 実装

　実装で一番難しい点は WordCloud の生成だと思われます。  

　なので今回は WordCloud の生成はPythonのライブラリとして提供されていましたので、それに頼って実装を行なっていきます。  

　それに合わせて実装も Python となります。  

▼ WordCloud の生成は以下のライブラリを利用します。文字列を渡すだけで WordCloud が生成されます。  
[https://github.com/amueller/word_cloud:embed:cite]

### 事前準備 - OAuth token

　リポジトリに「スターを付けたアカウント一覧の取得」と「アカウント名からプロフィール情報の取得」といったGitHubから情報を取得する処理は、GitHubのREST APIを活用することにします。  

ここで重要なのが API を活用するにおいて、事前に OAuth token を取得しておくことです。  

　OAuth token がなくても API を活用することは可能ですが、リクエスト数に制限があります。（2020年9月現在）

| | |
| - | - |
| トークンなし | 60 リクエスト / 時間 |
| トークンあり | 5000 リクエスト / 時間 |

　今回の実装では、1アカウントのプロフィールを取得毎に1リクエスト送信します。  
(スターが5,000以上あるリポジトリについては、今回の実装では動きません。sleepを掛ける、複数のトークンを活用するなどの修正を入れてやってください・・・。)

### 実装コード

　本記事では重要な部分を抜粋したコードのみを説明します。  　

▼ コードの全体・使い方はこちらから 
[https://github.com/motikan2010/GitHub-Stargazer-Analyzer:embed:cite]

#### 1. スターを付けたアカウント名一覧を取得

　アカウント名一覧の取得には、以下のAPIを活用します。  
[https://developer.github.com/v3/activity/starring/#list-stargazers:title]


　`GET /repos/:owner/:repo/stargazers` からスターを付けたアカウント一覧が取得できます。  
1ページに30アカウントが返却されるので、最終ページまで取得し続けます。  

<div class="md-code" style="width:100%">
```python
def get_stargazer_list(token, repository_name):
    stargazer_username_list = [] # アカウント名を格納するリスト
    page = 1
    while True:
        url = '%s/repos/%s/stargazers' % (GITHUB_API_URL, repository_name)
        print('%s?page=%i' % (url, page))
        r = requests.get(url=url, # https://api.github.com/repos/:owner/:repo/stargazers?page=X にアクセス
                         headers={'Accept': 'application/vnd.github.v3.star+json',
                                  'Authorization': 'token %s' % token}, # トークンの設定
                         params={'page': page})
        stargazer_list = json.loads(r.text)
        for stargazer in stargazer_list:
            stargazer_username_list.append(stargazer['user']['login']) # 取得したアカウント名をひたすら格納

        if len(stargazer_list) != 30: # 一覧の最終ページにたどり着いたら終わり
            break
        page = page + 1 # ページ進める
    return stargazer_username_list # アカウント一覧を返却
```
</div>

#### 2. アカウント名 から プロフィール情報 の取得

[https://developer.github.com/v3/users/#get-a-user:title]

　`GET /users/:username` からアカウント詳細が取得できます。  
bio(プロフィール)とlocation(所在地)の情報を保存しておきます。  

<div class="md-code" style="width:100%">
```python
def get_userinfo_list(username_list, token):
    userinfo_list = [] # プロフィール情報を格納するリスト
    size = len(username_list)
    for i, username in enumerate(username_list):
        url = '%s/users/%s' % (GITHUB_API_URL, username)
        print('%d/%d %s' % (i + 1, size, url))
        r = requests.get(url='%s' % url, # https://api.github.com/users/:username にアクセス
                         headers={'Authorization': 'token %s' % token})
        r_body = json.loads(r.text)
        bio = r_body['bio']
        if bio is not None:
            bio = bio.replace('\n', ' ')
        userinfo_list.append({'username': username, 'location': r_body['location'], 'bio': bio}) # bio と location を格納
    return userinfo_list
```
</div>

#### 3. 集めたプロフィール から ワードクラウド画像 を生成

　Mac環境には`/System/Library/Fonts/HelveticaNeue.ttc`フォントが元々ありました。  
特に指定しなくても動きます。  

<div class="md-code" style="width:100%">
```python
def create_word_cloud(userinfo_list, image_file_name, target='bio'):
    word_list = ''
    for userinfo in userinfo_list:
        if userinfo[target] is not None:
            word_list += userinfo[target] + '\n' # 各アカウントのプロフィールを1つの文章に格納

    stop_words = [u'am', u'is', u'of', u'and', u'the', u'to', u'it', u'for', u'in', u'as', u'or', u'are', u'be',
                  u'this', u'that', u'will', u'there', u'was'] # 頻出しすぎる単語は除外
    word_cloud = WordCloud(background_color="white",
                           font_path='/System/Library/Fonts/HelveticaNeue.ttc',
                           width=900, height=500, stopwords=set(stop_words)).generate(word_list) # 画像の生成
    word_cloud.to_file(image_file_name) # ファイルとして保存
```
</div>

### 生成された画像

　アカウントのbioを分析した結果、以下の画像が生成されました。  
[f:id:motikan2010:20200915211008p:plain]  

　「Security」という文字が目立っていることから<span class="m-y">セキュリティに関心のある人がスターを付けている</span>ことが分かりました。（まぁ予想はできていましたが、、）
他にも「Developer」だったり「Python」という文字が目立っていますね。  

　今回は脆弱性のPoC系のリポジトリが対象ということで、セキュリティに関心ある人がスターを付けているだろうなという予想があったので、今回は特に驚きはありませんでした。  

　しかし、そういった予想ができないリポジトリに対して分析してみると驚きがありそうです。  

## まとめ

　はじめて WordCloud を生成するライブラリをさわってみましたが、導入も含めて簡単に行うことができました。  
おもちゃ感覚でいろんなことに利用できそうです。

　ちなみに上記コードでは、プロフィール情報と一緒に location(所在地) を取得しましたが、locationで WordCloud を生成した結果は下の画像です。  
　中国が多いようですね。  
[f:id:motikan2010:20200915212119p:plain]

## 参考

- <span><a href="https://github.com/amueller/word_cloud" target="_blank">amueller/word_cloud: A little word cloud generator in Python</a></span>  
本記事で大活躍したリポジトリ
- <span><a href="https://qiita.com/seigot/items/c6093fa1e1a7bf72acaa" target="_blank">WordcloudでMr.Talkboxの英語歌詞のメッセージを可視化する - Qiita</a></span>  
word_cloudに関してはこちらを丸パクリ

## 更新履歴

- 2020年9月15日 新規作成
