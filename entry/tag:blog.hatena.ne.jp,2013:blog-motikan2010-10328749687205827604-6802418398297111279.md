<div style="text-align:center;">[f:id:motikan2010:20241019001903p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

Prowler を利用した クラウドセキュリティ設定診断 を紹介します。（※ 本記事はAWSが対象）  

先日は別の診断ツールである「CloudSploit」を活用したクラウドセキュリティ診断を紹介しましたが、** Prowler は 『Compliance（規格） に特化 **』しているようで新しい体験がありましたので紹介します。  
[f:id:motikan2010:20241019002712p:plain:w200]  
[https://blog.motikan2010.com/entry/2022/11/17/%E3%80%90%E5%85%A5%E9%96%80%E3%80%91%E7%84%A1%E6%96%99%E3%81%A7%E5%A7%8B%E3%82%81%E3%82%8BCSPM%EF%BC%88AWS%E7%B7%A8%EF%BC%89:embed:cite]

Prowler には「Prowler **SaaS**」と「Prowler **Open Source**」があり、 SaaS のほうは「検査環境を自分で用意する必要がない」かつ「検査結果の UI がリッチ」という良い面はありますが、検査対象が増えていくと有償となります。

「Prowler **Open Source**」を利用することで手軽に無償でクラウド環境（本記事ではAWS）のセキュリティ設定診断を実施することが可能となるため、その手順を本記事では説明します。

Prowler のソースコード：
<span><a href="https://github.com/prowler-cloud/prowler" target="_blank">prowler-cloud / prowler</a></span>

## 検証環境

本検証では EC2 で診断しています。  
（未検証ですが Python が動作すれば良さそうです）

- イメージ: Ubuntu Server 22.04 LTS 

以下公式情報：  
[f:id:motikan2010:20241018232953p:plain:w600]  
<span><a href="https://docs.prowler.com/projects/prowler-open-source/en/latest/" target="_blank">Overview - Prowler Open Source Documentation</a></span>

## 診断の流れ

### 診断用 IAM ユーザを追加

### IAM ユーザを作成

診断に利用する IAMユーザ「prowler-user」を作成します。  

[f:id:motikan2010:20241018231608p:plain]

引き続き、作成したユーザに４つのポリシーを付与します。  

#### ２つの インラインポリシー を追加

検査を実施するためには AWSの各サービスの読込権限が必要であるため、ポリシーの作成を行います。  
設定内容は以下の AWS CloudFormation を参考にしています。  
<span><a href="https://github.com/prowler-cloud/prowler-permissions-templates-public" target="_blank">prowler-cloud/prowler-permissions-templates-public</a></span>  

- ポリシー名：ProwlerAdditionalViewPrivileges

<div class="md-code" style="width:100%; font-family: 'Courier New';">
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "account:Get*",
                "appstream:Describe*",
                "appstream:List*",
                "backup:List*",
                "cloudtrail:GetInsightSelectors",
                "codeartifact:List*",
                "codebuild:BatchGet*",
                "dlm:Get*",
                "drs:Describe*",
                "ds:Get*",
                "ds:Describe*",
                "ds:List*",
                "ec2:GetEbsEncryptionByDefault",
                "ecr:Describe*",
                "ecr:GetRegistryScanningConfiguration",
                "elasticfilesystem:DescribeBackupPolicy",
                "glue:GetConnections",
                "glue:GetSecurityConfiguration*",
                "glue:SearchTables",
                "lambda:GetFunction*",
                "logs:FilterLogEvents",
                "macie2:GetMacieSession",
                "s3:GetAccountPublicAccessBlock",
                "shield:DescribeProtection",
                "shield:GetSubscriptionState",
                "securityhub:BatchImportFindings",
                "securityhub:GetFindings",
                "ssm:GetDocument",
                "ssm-incidents:List*",
                "support:Describe*",
                "tag:GetTagKeys",
                "wellarchitected:List*"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
```
</div>

- ポリシー名：ProwlerApiGatewayViewPrivileges

<div class="md-code" style="width:100%">
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "apigateway:GET"
            ],
            "Resource": [
                "arn:aws:apigateway:*::/restapis/*",
                "arn:aws:apigateway:*::/apis/*"
            ],
            "Effect": "Allow"
        }
    ]
}
```
</div>

#### ２つの AWS管理ポリシー を追加

作成したユーザに２つのAWS管理ポリシーを付与します。  

- SecurityAudit
- ViewOnlyAccess

最終的には下画像のように４つのポリシーが付与されたユーザになります。
[f:id:motikan2010:20241018224408p:plain]  

#### アクセスキー の発行

Prowler から作成したユーザを利用するためにアクセスキーを発行します。  
アクセスキーを発行することで診断に必要な以下の２つの文字列が発行されます。  

- アクセスキー
- シークレットキー

### Prowler 実行環境の構築

<div class="md-code" style="width:100%">
```
root@ip-172-31-19-66:~# python3 -V
Python 3.10.12
```
</div>

#### pip コマンドのインストール

Prowler をインストールするために必要な pip コマンドをインストールします。

<div class="md-code" style="width:100%">
```
# pip
Command 'pip' not found, but can be installed with:
apt install python3-pip

# apt install python3-pip

# pip -V
pip 22.0.2 from /usr/lib/python3/dist-packages/pip (python 3.10)
```
</div>

#### Prowler のインストール

本記事はバージョンを指定してインストールしています。（2024/10 時点の最新版）

<div class="md-code" style="width:100%">
```
# pip install prowler==4.4.0
```
</div>

環境によっては「AttributeError: module 'lib' has no attribute 'X509_V_FLAG_NOTIFY_POLICY'.」エラーが発生します。  

<div class="md-code" style="width:100%">
```
# prowler
(...省略...)
  File "/usr/lib/python3/dist-packages/urllib3/contrib/pyopenssl.py", line 50, in <module>
    import OpenSSL.SSL
  File "/usr/lib/python3/dist-packages/OpenSSL/__init__.py", line 8, in <module>
    from OpenSSL import crypto, SSL
  File "/usr/lib/python3/dist-packages/OpenSSL/crypto.py", line 1579, in <module>
    class X509StoreFlags(object):
  File "/usr/lib/python3/dist-packages/OpenSSL/crypto.py", line 1598, in X509StoreFlags
    NOTIFY_POLICY = _lib.X509_V_FLAG_NOTIFY_POLICY
AttributeError: module 'lib' has no attribute 'X509_V_FLAG_NOTIFY_POLICY'. Did you mean: 'X509_V_FLAG_EXPLICIT_POLICY'?
```
</div>

以下の GitHub Issue にエラーの解決策がありました。  
<span><a href="https://github.com/conda/conda/issues/13619" target="_blank">AttributeError: module 'lib' has no attribute 'X509_V_FLAG_NOTIFY_POLICY'. Did you mean: 'X509_V_FLAG_EXPLICIT_POLICY'? · Issue #13619 · conda/conda</a></span>

> `pyopenssl >= 23.2.0` can fix the problem.

エラー解決のためにPythonライブラリ「pyOpenSSL」をインストールします。  

<div class="md-code" style="width:100%">
```
# pip freeze | grep pyOpenSSL
pyOpenSSL==21.0.0
```
</div>

pyOpenSSL がインストールされたことを確認します。  
<div class="md-code" style="width:100%">
```
# pip install pyOpenSSL==24.2.1
# pip freeze | grep pyOpenSSL
pyOpenSSL==24.2.1
```
</div>

再度 Prowler をインストールを試してみると成功しました。  
<div class="md-code" style="width:100%">
```
# pip install prowler==4.4.0

--------------------------------------------------
# prowler
                         _
 _ __  _ __ _____      _| | ___ _ __
| '_ \| '__/ _ \ \ /\ / / |/ _ \ '__|
| |_) | | | (_) \ V  V /| |  __/ |
| .__/|_|  \___/ \_/\_/ |_|\___|_|v4.4.0
|_| the handy multi-cloud security tool

Date: 2024-10-18 04:48:12


2024-10-18 04:48:13,958 [File: aws_provider.py:943]     [Module: aws_provider]   CRITICAL: NoCredentialsError[934]: Unable to locate credentials

2024-10-18 04:48:13,958 [File: provider.py:228]         [Module: provider]       CRITICAL: NoCredentialsError[175]: Unable to locate credentials
```
</div>

### AWS 認証情報の設定

Prowaler は「`~/.aws/credentials`」から認証情報を参照しているので、認証情報を記載します。  
（※ AWS CLI の導入は必要なし）

<div class="md-code" style="width:100%">
```
# mkdir ~/.aws/
# vim ~/.aws/credentials
[prowler-user]
aws_access_key_id = ASI(...snip...)2ZF
aws_secret_access_key = 4F(...snip...)HD/
```
</div>


### prowler で診断を実行

先ほど設定したプロファイルを引数に指定( `--profile prowler-user` )して prowler を実行します。  

<div class="md-code" style="width:100%">
```
# prowler aws --profile prowler-user
```
</div>

コンプライアンス（規格）を指定していないため全ての検査項目を実行しますが約5分で完了します。  
検査結果は以下のようにターミナルに出力されます。  

検査の出力：  
<div class="md-code" style="width:100%">
```
(...省略...)

👇 進捗状況が表示されます！
Executing 457 checks, please wait...
-> Scan completed! |▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉| 457/457 [100%] in 5:26.6

Overview Results:
╭─────────────────────┬─────────────────────┬────────────────╮
│ 38.35% (400) Failed │ 61.27% (639) Passed │ 0.0% (0) Muted │
╰─────────────────────┴─────────────────────┴────────────────╯

👇 各AWSサービスの結果が表示されます！
Account 3(...省略...)9 Scan Results (severity columns are for fails only):
╭────────────┬───────────────────┬───────────┬────────────┬────────┬──────────┬───────┬─────────╮
│ Provider   │ Service           │ Status    │   Critical │   High │   Medium │   Low │   Muted │
├────────────┼───────────────────┼───────────┼────────────┼────────┼──────────┼───────┼─────────┤
│ aws        │ accessanalyzer    │ FAIL (17) │          0 │      0 │        0 │    17 │       0 │
├────────────┼───────────────────┼───────────┼────────────┼────────┼──────────┼───────┼─────────┤
│ aws        │ account           │ FAIL (1)  │          0 │      0 │        1 │     0 │       0 │
├────────────┼───────────────────┼───────────┼────────────┼────────┼──────────┼───────┼─────────┤
│ aws        │ apigateway        │ FAIL (6)  │          0 │      0 │        6 │     0 │       0 │
├────────────┼───────────────────┼───────────┼────────────┼────────┼──────────┼───────┼─────────┤
(...省略...)
├────────────┼───────────────────┼───────────┼────────────┼────────┼──────────┼───────┼─────────┤
│ aws        │ vpc               │ FAIL (1)  │          0 │      0 │        1 │     0 │       0 │
├────────────┼───────────────────┼───────────┼────────────┼────────┼──────────┼───────┼─────────┤
│ aws        │ ec2               │ FAIL (1)  │          0 │      0 │        1 │     0 │       0 │
├────────────┼───────────────────┼───────────┼────────────┼────────┼──────────┼───────┼─────────┤
│ aws        │ vpc               │ FAIL (3)  │          0 │      0 │        3 │     0 │       0 │
╰────────────┴───────────────────┴───────────┴────────────┴────────┴──────────┴───────┴─────────╯

👇 各コンプラインス（規格）に沿った簡易結果が表示されます！
(...省略...)

Compliance Status of AWS_ACCOUNT_SECURITY_ONBOARDING_AWS Framework:
╭───────────────────┬─────────────────┬────────────────╮
│ 11.22% (117) FAIL │ 6.62% (69) PASS │ 0.0% (0) MUTED │
╰───────────────────┴─────────────────┴────────────────╯

(...省略...)

Compliance Status of CIS_2.0_AWS Framework:
╭───────────────────┬─────────────────┬────────────────╮
│ 12.46% (130) FAIL │ 8.15% (85) PASS │ 0.0% (0) MUTED │
╰───────────────────┴─────────────────┴────────────────╯

Compliance Status of CIS_3.0_AWS Framework:
╭───────────────────┬─────────────────┬────────────────╮
│ 12.37% (129) FAIL │ 8.05% (84) PASS │ 0.0% (0) MUTED │
╰───────────────────┴─────────────────┴────────────────╯

Compliance Status of CISA_AWS Framework:
╭───────────────────┬───────────────────┬────────────────╮
│ 11.41% (119) FAIL │ 12.85% (134) PASS │ 0.0% (0) MUTED │
╰───────────────────┴───────────────────┴────────────────╯

(...省略...)

👇 詳細結果の保存先！
Detailed compliance results are in /root/output/compliance/
```
</div>

実際のターミナル上では下画像のように色付けされて検査結果が表示されます。  
[f:id:motikan2010:20241019005333p:plain:w700]

コンプライアンス（規格）別にも検査されるので、基準としている規格がある場合には参考にできそうです。  
[f:id:motikan2010:20241018223751p:plain:w700]  


### ブラウザ上で検査結果の確認

検査コマンド実行時に結果が表示されますが、詳細な検査結果は「`＜カレントディレクトリ＞/output/`」に出力されています。  

以下のコマンドを実行することでブラウザ上で検査結果を確認することが可能です。  
<div class="md-code" style="width:70%">
```
# HOST=0.0.0.0 prowler dashboard
```
</div>

Webダッシュボード（`http://<検査環境のIPアドレス>:11666/`）にアクセスすることで検査結果を確認することができます。  
[f:id:motikan2010:20241018223755p:plain:w700]  

「Compliance（規格）」を指定することカバレッジを確認することが可能です。  
[f:id:motikan2010:20241019010811p:plain:w700]

## まとめ

クラウドセキュリティ設定診断はコンプライアンス（規格）を基準に提供されていることが多いので、Prowlerを活用してみるのも良さそうです。

[f:id:motikan2010:20241019001910p:plain:w400]    

不明点などがあれば <span><a href="https://x.com/motikan2010" target="_blank">@motikan2010</a></span> にメッセージください。

## メモ書き

コンプライアンスに準拠するに当たって、ちょっとした疑問がありましたのでSlack上で質問したところ・・・。   
[f:id:motikan2010:20241019012603p:plain]  

3,4日で解決されたので、現状サポート面では不安にならなさそうです。  
（サポートなくともOSSなので修正できるところは良い）  
<span><a href="https://github.com/prowler-cloud/prowler/pull/5399" target="_blank">fix(iam): update AWS Support policy by sergargar · Pull Request #5399 · prowler-cloud/prowler</a></span>


