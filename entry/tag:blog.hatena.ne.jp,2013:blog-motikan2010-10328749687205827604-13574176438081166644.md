<div style="text-align:center;">[f:id:motikan2010:20220409053609p:plain]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　2022年4月6日、AWS Lambdaに大きなアップデートがありました。  
　アップデート内容を簡単に説明すると「AWS Lambda Function URLs」というもので、AWS Lambdaに**直接**HTTPSエンドポイントを設定できるようになりました。

[AWS Lambda Function URLs: built-in HTTPS endpoints for your Lambda functions](https://aws.amazon.com/jp/about-aws/whats-new/2022/04/aws-lambda-function-urls-built-in-https-endpoints/)

　図で表すとこんな感じです。
[f:id:motikan2010:20220409235415p:plain]

　AWS LambdaにHTTPSエンドポイントを設定できるのは便利でありますが、<span class="m-y">API Gatewayを介さないのでAWS WAFを設定することができなくなり、セキュリティ機構を持たせることが難しくなりました。</span>

[f:id:motikan2010:20220409235632p:plain]

　<span class="m-y">本記事ではRASP(Runtime application self-protection)を用いてAWS Lambdaにセキュリティ機構を持たせる方法を説明していきます。</span>  

[f:id:motikan2010:20220411182335p:plain]

　RASPにはDatadog社(元Sqreen社)のRASPを利用して検証していきます。**無料枠も提供されているの**で、興味がある方は実際に試してみるのもいいと思います。

▼ Sqreen社のサイト
[Application Security Management Platform | Sqreen](https://www.sqreen.com/)

## 準備

- AWS Lambda - Python 3.8
  - ※最新バージョンである 3.9 はSqreen RASPで対応していないので 3.8 を使います
- RASP (Sqreen)
  - Sqreen アカウント
  - 無料枠

### AWS Lambda

　AWS Lambda Function URLsを設定したAWS Lambdaを用意します。

▼ AWS Lambda Function URLs の設定方法はこのサイトが参考になります。
[[アップデート]LambdaがHTTPSエンドポイントから実行可能になる、AWS Lambda Function URLsの機能が追加されました！ | DevelopersIO](https://dev.classmethod.jp/articles/aws-lambda-function-urls-built-in-https-endpoints/)


　以下は本記事の検証環境の設定値です。

- 関数名：`testPublicFuntion`
- ランタイム：`Python 3.8`・・・ 3.9は現在RASP側で対応していませんでした。
- 関数 URL を有効化：チェック
  - 認証タイプ：NONE

[f:id:motikan2010:20220409053557p:plain]

#### コードの内容

　Lambdaのコードの処理内容は以下のようにしています。

- `/tmp/date.txt`ファイルに現在時刻を保存する
- `filename`クエリストリングに指定したファイルの内容を画面に返す

<div class="md-code" style="width:100%">
```python
import datetime

def lambda_handler(event, context):
    with open('/tmp/date.txt', mode='w') as f:
        f.write(datetime.datetime.now().strftime('%Y年%m月%d日 %H:%M:%S'))

    body = None
    with open('/tmp/' + event['queryStringParameters']['filename'], "r") as f:
        body = f.read()

    return {
        'statusCode': 200,
        'body': body
    }
```
</div>

### RASP

　Sqreenのドキュメントに従ってLambda レイヤーを作成します。

[Sqreen | Install Sqreen for AWS Lambda Python functions](https://docs.sqreen.com/python/installation/aws-lambda/)


　作成が完了すると、ダッシュボード上で「`sqreen-for-python`」というLambda レイヤーが確認できます。
[f:id:motikan2010:20220409054831p:plain]

　これで準備は完了となります。

## 動作確認

#### 正常系

　`filename`クエリストリングに`date.txt`を指定すると、ファイルの内容が返されました。
[f:id:motikan2010:20220409054318p:plain]

#### 攻撃系

　Lambdaのコードを見れば分かる通り、`filename`クエリストリングのバリデーション（値の検証）が実施されていないので、ディレクトリトラバーサルの脆弱性があります。  

　試しに「`../../../etc/passwd`」を入力すると、`/etc/passwd`ファイルの内容が返されました。  
（RASPのデフォルト動作として攻撃リクエストのブロックは行いません。ログ保存が行われます。）

[f:id:motikan2010:20220409054315p:plain]

### 攻撃ログの確認

　Sqreen RASPの管理画面で攻撃ログが保存されていることを確認することができます。

　先ほど攻撃リクエストを送信したので、攻撃ログが保存されています。
[f:id:motikan2010:20220409053627p:plain]

　攻撃元IPや攻撃種別などを確認することができます。
[f:id:motikan2010:20220409053612p:plain]

　攻撃詳細を画面では実際のリクエスト内容の詳細情報を確認することができます。
[f:id:motikan2010:20220409053602p:plain]

#### 攻撃のブロック設定

　デフォルトでは攻撃リクエストをブロックせずログを保存する設定になっています。

　攻撃リクエストをブロックするように設定を変更します。
[f:id:motikan2010:20220409053615p:plain]

　再度攻撃リクエストを送信すると、攻撃が成功しないことが確認できます。
[f:id:motikan2010:20220409053618p:plain]

## 更新履歴

- 2022年04月09日 新規作成
