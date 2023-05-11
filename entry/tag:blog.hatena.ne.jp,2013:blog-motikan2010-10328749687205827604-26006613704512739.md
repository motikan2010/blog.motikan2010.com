<div style="text-align:center;">[f:id:motikan2010:20210322225816g:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　まず「OWASP ServerlessGoat」というのはサーバレス環境を想定したアプリケーションセキュリティを学習するためのツールです。  
脆弱性が含まれているアプリケーションのデプロイができ、実際に攻撃することでセキュリティの学習を行うことができます。  

　その「OWASP ServerlessGoat」の Python 版を作成・公開したので共有します。
[https://github.com/motikan2010/Serverless-Goat-Python:embed:cite]

　すでに Java 版は存在していましたが、Python 版は見当たらなかったので作成しました。

↓ Java 版 の OWASP ServerlessGoat
[https://github.com/CodeShield-Security/Serverless-Goat-Java:title]


　OWASP ServerlessGoat の具体的な学習内容は下の記事が参考になります。  
[サーバーレスやられアプリを使った脆弱性診断ハンズオン - Qiita](https://qiita.com/yuuhu04/items/d45fd7acbb4919adbc57)

　また、似たような学習ツールに「OWASP DVSA」というものがあります。  
[https://github.com/OWASP/DVSA:title]


### OWASP ServerlessGoat の注意

　ベースとなっている OWASP ServerlessGoat で学習を進めるにあたってはいくつか注意点があります。 

　リポジトリのサポートが止まっていることが主な原因なのですが、学習を進めるに当たって知っていた方がいいので列挙します。   

[https://github.com/OWASP/Serverless-Goat:title]

#### 1. 更新が止まっている  
 
　PureSec 社がメンテナンスしていますが、2019年5月に買収されたのが原因なのかメンテナンスが行われていません。  

　CloudFormationテンプレート内のLamdaのランタイムが「nodejs8.10」で作成されているので、現在のLambdaでは起動することができません。  

　起動する場合は各自でテンプレートを編集してデプロイする必要があります。  

#### 2. Lambda ランタイムが Node.js
 
　注意点というよりは理想ですが、やられアプリはシェアの高い環境下を想定されているものがいいと考えています。
 
　Datadog 社の調査によると 2019年 Lambda ランタイムは Python のほうが人気とのこと。  
しかし OWASP ServerlessGoat は Node.js のみをサポートしている状態です。

<div style="text-align:center;">[f:id:motikan2010:20210317222554p:plain:w400]</div>

[https://www.datadoghq.com/ja/state-of-serverless/:title]

#### 3. PureSec 社のサイト消失

　ServerlessGoat を利用しての学習を進める上で `https://puresec.io/hubfs/document.doc` の Word ファイルにアクセスして正常動作を確認する場合がありますが、既にこのファイルは存在しておらず正常動作を確認できない状態となっています。正常動作を確認するには適当な Word ファイルを見つけてアクセスする必要があります。  

　**正常動作が確認できないだけで、脆弱性を使った学習に影響はありません。**  

## 使い方

　私は sam コマンドを使ってビルド・デプロイをしました。

<div class="md-code" style="width:100%">
```
$ sam --version
SAM CLI, version 1.20.0
```
</div>

### ビルド

<div class="md-code" style="width:100%">
```
$ sam build --use-container
```
</div>

### デプロイ

<div class="md-code" style="width:100%">
```
$ sam deploy --stack-name "serverless-goat-pyton" --guided
```
</div>
※ `serverless-goat-pyton`の部分は任意の文字列

　下画像がトップ画面です。Word ファイルはリポジトリ内にホスティングするようにしています。  
<div style="text-align:center;">[f:id:motikan2010:20210318010310p:plain:w600]</div>

## 学習

### Lesson 4: Exploiting Over-Privileged IAM Roles

　DynamoDB から全データを取得する項目ですが Python で記述すると以下のようになります。

<div class="sm-code" style="width:100%">
```
https://; python -c 'import os;import boto3;print(boto3.resource("dynamodb").Table(os.getenv("TABLE_NAME")).scan())' #
```
</div>

## まとめ

　以上、「ServerlessGoat for Python」の紹介でした。  

　まだ、README などのドキュメント類の修正はしておらず本家のままとなっていますが、気になる部分を見つけ次第修正してしようかなとは思っています。

　CloudFormation 環境の構築のために作成したポリシー。IAM力が足りないと実感・・・。  
<figure class="figure-image figure-image-fotolife" title="※動くけど悪い例">[f:id:motikan2010:20210318011947p:plain:w300]<figcaption>※動くけど悪い例</figcaption></figure>

## 更新履歴

- 2021年3月17日 新規作成