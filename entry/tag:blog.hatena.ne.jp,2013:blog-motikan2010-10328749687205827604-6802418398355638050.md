<div style="text-align:center;">[f:id:motikan2010:20250420234906p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　最近「**セキュリティ診断AIエージェント**」というものを見て楽しそうと思ったので、調査/開発に着手し始めましたのでそれのメモです。  

▼ ソースコード（Dockerで動作します）  
[https://github.com/motikan2010/Niwaka/tree/0.1.0:embed:cite]

▼ Slack上で開発に関する様々なタスクを依頼することができます。  
[f:id:motikan2010:20250419224857p:plain:w700]

▼ 処理の流れ  
[f:id:motikan2010:20250421112753p:plain:w700]

## 環境準備

### 1. アカウントの作成

Niwaka を動作させるには２つのアカウントを作成します。  

- **Slack アカウント**
  - Slackbot を作成するのに利用します。
  - ▼ Slackbot の作成はコチラを参考しました。  
[https://nuco.co.jp/blog/article/Nt6tPV78:title]

- **OpenAI API アカウント**
  - 生成AIには OpenAI を利用します。
  - OpenAI API 利用は有償ですが $10 チャージ程あれば十分です。  
（私の場合は $2 分ほどしか消費しませんでした）  
[f:id:motikan2010:20250420233333p:plain:w500]

### 2. 設定値の指定 （Slack トークンやOpenAI API キー など）

#### app/.env

`.env.example` をコピーして設定してください。

- **BOT_TOKEN** : Slack API の"OAuth Tokens"
- **APP_TOKEN** : Slack API の"App-Level Tokens"
- **CHANNEL_ID** : Slack のチャンネルID

#### .llm/config.json

`config.json.example` をコピーして設定してください。

- **OPENAI_API_KEY** : OpenAI API キー
- **REPO_NAME** : 検査対象リポジトリ（リポジトリは手動で clone する必要があります）
- **GITHUB_PERSONAL_ACCESS_TOKEN** : Github の Personal access tokens（アクセストークン）
  - アクセストークンのパーミッションはこんな感じ  
[f:id:motikan2010:20250424190320p:plain:w400]

### 3. ソースコードの配置

　`repos`配下にAIが参照するソースコードのリポジトリを配置する。

### 4. 起動

<div class="md-code" style="width:50%">

```
$ make up
 or 
$ docker compose up
```

</div>

## 動作検証

### 脆弱性の修正プルリク作成

[f:id:motikan2010:20250419224857p:plain:w700]

[f:id:motikan2010:20250419224908p:plain:w700]

### SECURITY.md ファイルの作成

[f:id:motikan2010:20250420042227p:plain:w700]  

[blog:g:12921228815726579926:banner][blog:g:11696248318754550880:banner]
