<div style="text-align:center;">[f:id:motikan2010:20230814205824p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　前回の記事ではOSSのCSPMツールである「CloudSploit」を用いてAWS環境のセキュリティ診断を行う方法を紹介しました。  

CSPM（Cloud Security Posture Management）の簡単な説明は、この記事に記載しています。  

[https://blog.motikan2010.com/entry/2022/11/17/%E3%80%90%E5%85%A5%E9%96%80%E3%80%91%E7%84%A1%E6%96%99%E3%81%A7%E5%A7%8B%E3%82%81%E3%82%8BCSPM%EF%BC%88AWS%E7%B7%A8%EF%BC%89:embed:cite]

　今回はCloudSploitを用いて、**Azure環境**に対してセキュリティ診断を実施していきます。  

### 環境

　Cloudsploit を動作させるためには Nodejs が必要です。本検証では以下のバージョンを利用しています。  

||||
| - | - |
| Node.js | 18.16.0 |


## Azureから認証情報の取得手順

　CloudSploit を用いてAzure環境を診断する場合には、Azureにログインして下記４種類の値の取得する必要があります。

- Application ID （アプリケーション ID）
- Directory ID （ディレクトリ ID）
- Client secrets の value （クライアントシークレットの値）
- Subscription ID （サブスクリプション ID）

　それらの値の取得手順を以下に記載します。  

### アプリケーションの登録

1. Azure ポータルにログインし、「**Azure Active Director**」サービスに移動します。
2. 「**App registrations**」を選択し、「**New registration**」をクリックする。  
[f:id:motikan2010:20230814124034p:plain:w700]
3. 「**Name**」フィールドに「CloudSploit」などの分かりやすい名前を入力します。
4. 「**Supported account types**」のデフォルトは「**Accounts in this organizational directory only (YOURDIRECTORYNAME)**」のままにしておきます。
[f:id:motikan2010:20230814124134p:plain:w700]
5.  「Register」をクリックします。
[f:id:motikan2010:20230814124231p:plain:w700]
6. 「**Application ID**」をコピーしてメモします。
7. 「**Directory ID**」をコピーしてメモします。

### クライアントシークレットの作成

8. 「**Certificates & secrets**」をクリックする。
9. 「Client secrets」の下にある「**New client secret**」をクリックする。
10. "Description"（例：Cloudsploit-2019）を入力し、"Expiers"に「**Recommended: 180 days (6 months)**」を選択します。  
[f:id:motikan2010:20230814124355p:plain:w700]
11. 「Add」をクリックします。
12. **クライアントシークレットの値**をメモします。  
（※クライアントシークレットの値は一度だけ表示されます。）  
[f:id:motikan2010:20230814124437p:plain:w700]

### 検査用ロールの割り当て

13. 「**Subscriptions**」に移動します。
[f:id:motikan2010:20230814124636p:plain:w700]
14. 関連する「**Subscription ID**」メモして、IDをクリックします。
15. 「**Access Control (IAM)**」をクリックします。
16. 「**Role assignments**」タブに移動します。
17. 「**Add**」をクリックし、「**Add role assignment**」をクリックします。
[f:id:motikan2010:20230814124650p:plain:w700]
18. 「Role」のドロップダウンで「**Security Reader**」を選択します。
19. 「Assign access to」はデフォルト値のままにしておきます。
20. 「Select」ドロップダウンに、作成したアプリ登録名（例："CloudSploit"）を入力し、選択します。
[f:id:motikan2010:20230814124703p:plain:w700]
21. 「**Next**」、そして「**Review + assign**」をクリックする。
22. 同様に「**Log Analytics Reader**」ロールを付与します。
[f:id:motikan2010:20230814124714p:plain:w700]

## セキュリティ診断の実施

　Cloudsploit を実行するための認証情報を取得できたので、次は設定を行っていきます。

### インストールと設定

```
-- ソースコードの取得
$ git clone https://github.com/aquasecurity/cloudsploit.git

-- 執筆時点の最新バージョンに切り替える
$ git checkout 7fce62cac57330019e886ea282d0a57a953b74f8

-- 必要なライブラリのインストール
$ npm install

-- 設定ファイルのコピー
$ cp config_example.js config.js
```

　コピーした「`config.js`」を編集します。編集前は下画像の箇所がコメントアウトされているため、コメントを解除します。  
[f:id:motikan2010:20230814124728p:plain:w700]

　Azure から取得した情報を環境変数に設定します。

```
$ export AZURE_APPLICATION_ID="＜Application ID （アプリケーション ID）＞"
$ export AZURE_DIRECTORY_ID="＜Directory ID （ディレクトリ ID）＞"
$ export AZURE_KEY_VALUE="＜Client secrets の value （クライアントシークレットの値）＞"
$ export AZURE_SUBSCRIPTION_ID="＜Subscription ID （サブスクリプション ID）＞"
```
### 検査の実施

　「`index.js`」ファイルを実行することで検査を実施することができます。

　以下のコマンドで検査を実施できます。

<div class="md-code" style="width:100%">

```
$ ./index.js --config=./config.js --cloud=azure --console=none --json=./results.json --compliance=pci
```

</div>

　コマンドに指定しているオプションの説明は次のとおりです。

| オプション | 説明 |
| - | - |
| --config=./config.js | 先ほど編集した設定ファイルを反映します。 |
| --cloud=azure | Azure 環境に対して検査を実施します。 |
| --console=none | 検査結果をターミナル上に出力しないようにします。 |
| --json=./results.json | 検査結果をJSONファイルに出力するようにします。 |
| --compliance=pci | PCI DSS に基づいた検査項目のみ検査します。 |

### 検査結果を確認

　検査結果がJSONファイルに出力されていることが確認できます。

<div class="md-code" style="width:100%">

```
$ cat results.json | head -n 20
[
  {
    "plugin": "networkAccessDefaultAction",
    "category": "Storage Accounts",
    "title": "Network Access Default Action",
    "description": "Ensures that Storage Account access is restricted to trusted networks",
    "resource": "N/A",
    "region": "eastus",
    "status": "OK",
    "message": "No storage accounts found",
    "compliance": "PCI: PCI requires data access to be configured to use a firewall. Removing the default network access action enables a more granular level of access controls."
  },
(以下省略)
```

</div>

　検査結果をTSVファイルに変換して「エクセル」や「Google スプレッドシート」などで表示することで検査結果が見やすくなります。  

<div class="md-code" style="width:100%">

```
$ cat results.json | jq -r '.[] | [.plugin, .category, .title, .description, .resource, .region, .status, .message, .compliance] | @tsv' > result.tsv
```

</div>

　例の検査結果では、以下の3つが検出されていることが確認できます。

- ネットワークセキュリティグループの設定で「SSH (22 番ポート)が全ての送信元にオープン状態」
- ネットワークセキュリティグループの設定で「RDP (3389 番ポート)が全ての送信元にオープン状態」
- ネットワークセキュリティグループの設定で「MySQL (3306 番ポート)が全ての送信元にオープン状態」

[f:id:motikan2010:20230814125056p:plain]

## まとめ

CloudSploitを活用することにより簡単にAzure環境のセキュリティ診断を行うことができました。  
これで十分なのかと言われるとそうではないですが、クラウドセキュリティを検討する際の入り口として活用できると思います。  

[blog:g:11696248318754550880:banner][blog:g:12921228815726579926:banner]  