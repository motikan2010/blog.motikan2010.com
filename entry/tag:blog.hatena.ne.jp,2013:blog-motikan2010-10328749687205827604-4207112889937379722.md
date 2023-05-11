<div style="text-align:center;">[f:id:motikan2010:20221117224937p:plain]</div>

<div class="contents-box"><p>[:contents]</p></div>

## CSPM とは

　CSPM（シーエスピーエム）は「<span class="m-y">Cloud Security Posture Management</span>」の略称でクラウド環境の"ポスチャ"を管理するツールです。  

　Posture という単語にあまり馴染みがないですが、とりあえず「Posture ≒ 状態」と捉えればよさそうです。  
[f:id:motikan2010:20221117155556p:plain:w500]

　日本語では「クラウドセキュリティ動態管理」や「クラウドセキュリティ態勢管理」、「クラウドセキュリティポスチャマネジメント」の表記をよく見かけます。

　CSPM はクラウドの状態を検査するツールであり、検査対象のクラウド環境の以下のようなものを検出します。

- 「<span class="m-y">バッドプラクティスな利用</span>」 → ルートユーザを利用している　など
- 「<span class="m-y">脆弱な設定</span>」 → 読み込み権限、書き込み権限がパブリックになっているS3バケットが存在している　など
- 「<span class="m-y">提供されているセキュリティ機能やサービスの不使用</span>」 → VPC フローログが有効になっていない など

## 動作検証

　本検証では AWS環境に対してCSPMを使います。

　CSPM には AWS から提供されている「AWS Config」などもありますが、<span class="m-y">本記事では AquaSecurity 社からOSSで提供されている「CloudSploit」を使います。</span>
[https://github.com/aquasecurity/cloudsploit:embed:cite]  

　Dockerfile が提供されているので Docker で動作させます。

### 検査用のIAMユーザを追加

　検査を実行するには検査用のIAMユーザが必要なので、AWSコンソールにアクセスし、IAMユーザを追加します。  

「ユーザ名」は任意のものでよいので、ここでは「`test_cloudsploit`」にしています。  
APIキーが必要なので「アクセスキー - プログラムによるアクセス」にチェックします。
[f:id:motikan2010:20221117194017p:plain:w600]

　ユーザに対して「`SecurityAudit`」をアタッチします。
[f:id:motikan2010:20221117194014p:plain:w600]

　後は特に編集せずにユーザ追加を完了させます。

### CloudSploit の導入

#### コードの取得

　リポジトリ上では最終リリースが 2020年8月26日 であり、バージョン 2.0.0 が最新のものとなっています。
これは少し古いため `master`ブランチのコードを使って検証を行なっていきます。

　コードのクローン後にチェックアウト(checkout)していますが、執筆時点(2022/11/17)の最新コードにするためです。

<div class="md-code" style="width:100%">
```
$ git clone https://github.com/aquasecurity/cloudsploit.git
$ cd cloudsploit
$ git checkout c19bf228dd04585609a432c5e4c9adbcb3c7f25a
```
</div>

#### コードの修正

　取得したコードをそのまま実行してもエラーが発生することが確認されています。

　解消するために「`Dockerfile`」を以下のように編集します。

<div class="md-code" style="width:100%">
```diff
$ git diff
diff --git a/Dockerfile b/Dockerfile
index d65d367d..5de252ce 100644
--- a/Dockerfile
+++ b/Dockerfile
@@ -14,7 +14,8 @@ COPY . /var/scan/cloudsploit/
 # Install cloudsploit/scan into the container using npm from NPM
 RUN cd /var/scan \
 && npm init --yes \
-&& npm install ${PACKAGENAME}
+&& npm install ${PACKAGENAME} \
+&& npm link /var/scan/cloudsploit

 # Setup the container's path so that you can run cloudsploit directly
 # in case someone wants to customize it when running the container.
```
</div>

参考：fix-broken-node-bin-links by pasanchamikara · Pull Request #1119 · aquasecurity/cloudsploit
https://github.com/aquasecurity/cloudsploit/pull/1119

#### Dockerfile からイメージを 構築 (build)

　Dockerfileの修正が完了したらDockerイメージを作成するためにビルドをします。

<div class="md-code" style="width:100%">
```
$ docker build . -t cloudsploit:0.0.1
```
</div>

#### 動作テスト

　動作確認のためにヘルプを表示します。

<div class="md-code" style="width:100%">
```
$ docker run cloudsploit:0.0.1 --help

   _____ _                 _  _____       _       _ _
  / ____| |               | |/ ____|     | |     (_) |
 | |    | | ___  _   _  __| | (___  _ __ | | ___  _| |_
 | |    | |/ _ \| | | |/ _` |\___ \| '_ \| |/ _ \| | __|
 | |____| | (_) | |_| | (_| |____) | |_) | | (_) | | |_
  \_____|_|\___/ \__,_|\__,_|_____/| .__/|_|\___/|_|\__|
                                   | |
                                   |_|

  CloudSploit by Aqua Security, Ltd.
  Cloud security auditing for AWS, Azure, GCP, Oracle, and GitHub

usage: cloudsploit-scan [-h] [--config CONFIG]
                        [--compliance {hipaa,cis,cis1,cis2,pci}]
(...snip...)
```
</div>


### 検査の実行 パターン①

　CloudSploit が正常に動作することが確認できたため、検査を実行してみます。

　デフォルトでは検査結果はコンソールに出力されるようになっていますが、本検証ではJSONファイルに出力するようにします。

<div class="md-code" style="width:100%">
```
$ docker run -e AWS_REGION='ap-northeast-1' -e AWS_ACCESS_KEY_ID='<AWS_ACCESS_KEY_ID>' \
    -v /tmp/result2.json:`pwd`/result2.json \
    cloudsploit:0.0.1 --console=none --json=/tmp/result2.json --compliance=pci

INFO: Found 64 API calls to make for aws plugins
```
</div>

■ オプションの説明

- Docker 側
  - 「`-e AWS_REGION='ap-northeast-1'`」・・・指定しないとエラーになるので指定しています。  
参考：<span><a href="https://github.com/aquasecurity/cloudsploit/issues/1101" target="_blank">Missing region in config Error Message in AWS prevents running the collection/scan · Issue #1101 · aquasecurity/cloudsploit</a></span>
  - 「`-e AWS_ACCESS_KEY_ID`」・・・作成したIAMユーザのアクセスキーを指定します。
  - 「`-e AWS_SECRET_ACCESS_KEY`」・・・作成したIAMユーザのシークレットキーを指定します。
  - 「``-v `pwd`/tmp/:/tmp/``」・・・Docker環境内の /tmp/ をカレントディレクトリの tmp/ にマウントします。  
このディレクトリに検査結果のJSONファイルを出力します。
- CloudSploit 側
  - 「`--console=none`」・・・検査結果をコンソールに出力しない。 大量の結果が表示されるため抑制していおきます。
  - 「`--json=/tmp/result2.json`」・・・検査結果をJSON形式で「`/tmp/result.json` (※Docker環境内)」ファイルに出力します。
  - 「`--compliance=pci`」・・・PCI DSS の項目のみを検査するにします。デフォルトでは全項目を検査するため、時間短縮のために指定しています。

#### 検査結果の確認

　正常に検査が完了した場合、カレンとディレクトリの「tmp/result2.json」に検査結果が出力されます。
[f:id:motikan2010:20221117221334p:plain:w600]  

　デフォルトでは問題有無に関係なく全ての検査項目が出力されるようになっています。

　問題がある項目のみを出力するために、jqコマンドで status が FAIL になっている検査項目を出力するようにします。  

　今回は EC2 と S3 のみを抜粋して検査結果を説明します。

##### EC2 の検出項目

　**EC2 に関しては以下の2点を検出できました。**

- VPC フローログがトラフィックログが有効になっていない
- default のセキュリティグループが「全トラフィックをブロック」になっていない

<div class="sm-code" style="width:100%">
```
$ cat tmp/result2.json | jq '.[] | select(.category == "EC2" and .region == "ap-northeast-1" and .status == "FAIL")'
```
</div>

[f:id:motikan2010:20221117212116p:plain]

##### S3 の検出項目

　**S3 に関しては以下の1点を検出できました。**

- S3バケットのロギングが有効になっていない

<div class="sm-code" style="width:100%">
```
$ cat tmp/result2.json | jq '.[] | select(.category == "S3" and .region == "ap-northeast-1" and .status == "FAIL")'
```
</div>

[f:id:motikan2010:20221117212113p:plain]

### 検査の実行 パターン②

　今度は1項目のみを検査してみることにします。

　CloudSploit ではプラグインという単位で検査項目が作成されています。そのため「`--plugin`」オプションを付与することで1項目の検査をすることができます。

　試しに「`rootAccountInUse`」というプラグインを指定してみることにします。これは「ルートユーザの利用有無」を検査するプラグインです。

<div class="md-code" style="width:100%">
```
$ docker run -e AWS_REGION='ap-northeast-1' -e AWS_ACCESS_KEY_ID='<AWS_ACCESS_KEY_ID>' \
    -v `pwd`/tmp/:/tmp/ \
    cloudsploit:0.0.1 --console=none --json=/tmp/result2.json --compliance=pci --plugin=rootAccountInUse
```
</div>

■ オプションの説明

- 「`--plugin=rootAccountInUse`」・・・`rootAccountInUse`プラグインのみを検査します。
1項目だけを検査するため早く完了します。

#### 検査結果の確認

##### IAM の検出項目

　パターン①の時とは違い status の値が OK となっています。これは問題がないことを表しています。  

　検査対象のAWS環境ではルートユーザを利用していないため、これは正しい結果です。  

<div class="sm-code" style="width:100%">
```
$ cat tmp/result2.json | jq '.[] | select(.category == "IAM" and .plugin == "rootAccountInUse")'
```
</div>

[f:id:motikan2010:20221117215203p:plain]

　次に検査対象に対してルートユーザでログインして、再度検査を実施してみます。

　その結果が以下の画像です。status が FAIL となっており、問題が発生したことが確認できました。

[f:id:motikan2010:20221118080842p:plain]

## 検査を終えて

### CloudSploit の検査の流れ

　検査結果の確認ができたので CloudSploit の処理の流れを軽く見てみることにします。

大きく処理を分けると以下の3つになります。

1. AWS API にアクセスし情報収集
2. 収集情報をプラグインで解析
3. 結果の出力

　AWS API からの収集情報は「`--collection`」オプションで保存できるようになっています。

<div class="sm-code" style="width:100%">
```
$ docker run -e AWS_REGION='ap-northeast-1' -e AWS_ACCESS_KEY_ID='<AWS_ACCESS_KEY_ID>' -e AWS_SECRET_ACCESS_KEY='<AWS_SECRET_ACCESS_KEY>' \
    -v `pwd`/tmp/:/tmp/ \
    cloudsploit:0.0.1 --console=none --json=/tmp/result2.json --compliance=pci --plugin=rootAccountInUse --collection=/tmp/collection.json
```
</div>

■ オプションの説明

- 「`--collection=/tmp/collection.json`」・・・収集した情報(API結果)がJSON形式で出力するようにします。

　上記コマンドを実行した結果の「`collection.json`」ファイルの中身は以下のようになります。  
[f:id:motikan2010:20221117222047p:plain:w400]  

　「`password_last_used`（ルートユーザの最終認証日）」を参照してルートユーザの利用有無を確認しているようでした。

　各種検査項目は以下のディレクトリに記述されています。  
<span><a href="https://github.com/aquasecurity/cloudsploit/tree/master/plugins/aws" target="_blank">Cloud Security Posture Management (CSPM)</a></span>

### CSPM を応用する

　状態の異常を検出するCSPMですが、 ** API から取得した情報（`collection.json`）の差分を取得・監視し続けることにより、監視項目にはない変更も検出可能です**。  
（セキュリティグループが更新された、IAMユーザが新規に作成された など）

## まとめ

　CSPMは、「簡単に導入でき」かつ「効果が大きい」のでもっと普及してほしい。

## 更新履歴

- 2022年11月17日 新規作成


[blog:g:12921228815726579926:banner]
