<div style="text-align:center;">
[f:id:motikan2010:20200909213042p:plain]
</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## GRR Rapid Response とは？

　「GRR Rapid Response」（以下、GRR）はGoogleが開発しているインシデントレスポンス時にするツールであり、主に**フォレンジック作業を支援するツール**です。  

　このツールの特徴として、<span class="m-y">フォレンジック対象の端末にログインすることなく、フォレンジック作業を行うことができます。</span>

　そのため、事前にフォレンジック対象端末にはGRRエージェントがインストールされている必要があります。  

▼ イメージ図
<div style="text-align:center;">[f:id:motikan2010:20200911202437p:plain:w600]</div>

[https://grr-doc.readthedocs.io/en/v3.4.0/what-is-grr.html:title]

### GRR クライアントの機能

- クライアントは `Linux`、`OS X`、`Windows` に対応
- `YARAライブラリ`を使用したライブリモートメモリフォレンジック
- 「ファイル」と「Windowsレジストリ」の強力な検索やダウンロード機能
- `SleuthKit（TSK）`を使用したOSレベルおよびrawファイルシステムアクセス
- インターネット展開用に設計された安全な通信インフラストラクチャ
- クライアントのCPU、メモリ、IOの使用状況、および自分で課した制限の詳細な監視

### GRR サーバの機能

- ほとんどのインシデント対応およびフォレンジックタスクを処理する、完全な対応機能
- エンタープライズハンティング（一連のマシン全体の検索）のサポート
- 何百ものデジタルフォレンジックアーティファクトの高速でシンプルなコレクション
- AngularJS Web UIとRESTful JSON API、Python、PowerShell、Goのクライアントライブラリ
- さまざまなフォーマットや出力プラグインをサポートするデータエクスポート機能
- 大規模なデプロイメントを処理できる完全にスケーラブルなバックエンド
- スケジューリングによるタスクの自動実行
- 多数のラップトップで動作するように設計された、クライアントの将来のタスクスケジューリングを可能にする非同期設計

### GRRの開発リポジトリ

　GRR Rapid Response はOSSであり、コードはGItHubで管理されています。  

[https://github.com/google/grr:embed:cite]

## 環境構築

　では実際に動作確認を行うためにGRRの環境構築を行っていきます。

　以下のように役割が異なる２つのサーバ（仮想環境）を構築します。

- フォレンジック対象である「<b>GRR クライアント</b>」
- フォレンジックに必要な情報を取得する「<b>GRR サーバ</b>」
  
　今回はGRRクライアントは１台で作業を進めていきます。  

<div style="text-align:center;">[f:id:motikan2010:20200909233612p:plain]</div>

### 環境の概要

　GRRサーバはUbuntu上に構築してきます。

　GRRクライアントも同様にUbuntu上に導入します。  
GRRクライアントでサポートされているOSは以下の３です。

- Windows
- Mac OS
- Linux OS

　今回の検証で利用する各ソフトウェアのバージョンは以下の通りです。  

| | |
| - | - |
| VirtualBox | 6.1.12 |
| Ubuntu | 18.04.3 |
| GRR Rapid Response | 3.4.2 |

　「VirtualBox」と「Ubuntu」のバージョンに関しては最近のものであればなんでもいいと思います。

### GRR サーバの構築

▼debパッケージを使用したインストールマニュアル  
[https://grr-doc.readthedocs.io/en/latest/installing-grr-server/from-release-deb.html:embed:cite]

#### MySQL インストール

　GRRサーバを動作させるためにはMySQLが必要です。  

　インストールと一緒に「専用ユーザの作成」と「DBの作成」を行います。

<div class="md-code" style="width:100%">
```
# MySQLのインストール
$ sudo apt install -y mysql-server

# ユーザ・DBを作成
$ sudo mysql -u root -p
> SET GLOBAL max_allowed_packet=41943040;
> CREATE USER 'grr'@'localhost' IDENTIFIED BY 'grr_pass';
> CREATE DATABASE grr;
> GRANT ALL ON grr.* TO 'grr'@'localhost';

# 確認
$ sudo mysql -u'grr' -p'grr_pass' -e 'show databases'
```
</div>

#### GRR Rapid Response インストール

　debパッケージをダウンロードして、インストールします。  
<div class="md-code" style="width:100%">
```
$ wget https://storage.googleapis.com/releases.grr-response.com/grr-server_3.4.2-0_amd64.deb
$ sudo apt install -y ./grr-server_3.4.2-0_amd64.deb
```
</div>


　インストール中にDBの認証情報やGRRサーバの情報などの設定項目が聞かれます。  
今回はメールについての機能は使用しない想定ですので、メール関連の設定はスキップ（デフォルト設定）しています。  

　設定内容は設定ファイルに保存されますので、後で変更することも容易です。  

<div class="md-code" style="width:100%">
```
Step 0: Importing Configuration from previous installation.
No old config file found.

Step 1: Setting Basic Configuration Parameters
We are now going to configure the server using a bunch of questions.
Use Fleetspeak (next generation communication framework)? [yN]:  [N]:【✅ Enter】


-=GRR Datastore=-
For GRR to work each GRR server has to be able to communicate with
the datastore. To do this we need to configure a datastore.

GRR will use MySQL as its database backend. Enter connection details:
MySQL Host [localhost]:【✅ Enter】
MySQL Port (0 for local socket) [0]: 3306  ✅ MySQLのポート番号
MySQL Database [grr]:【✅ Enter】
MySQL Username [root]: grr  ✅ 先ほど作成したDBユーザ
Please enter password for database user grr:【✅ Enter】
Configure SSL connections for MySQL? [yN]:  [N]:【✅ Enter】
Successfully connected to MySQL with the provided details.


-=GRR URLs=-
For GRR to work each client has to be able to communicate with the
server. To do this we normally need a public dns name or IP address
to communicate with. In the standard configuration this will be used
to host both the client facing server and the admin user interface.

Please enter your hostname e.g. grr.example.com [user1-VirtualBox]: 192.168.56.15  ✅ ホストOSからもアクセスするのでゲストOSのIP


-=Server URL=-
The Server URL specifies the URL that the clients will connect to
communicate with the server. For best results this should be publicly
accessible. By default this will be port 8080 with the URL ending in /control.

Frontend URL [http://192.168.56.15:8080/]:【✅ Enter】


-=AdminUI URL=-:
The UI URL specifies where the Administrative Web Interface can be found.

AdminUI URL [http://192.168.56.15:8000]:【✅ Enter】


-=GRR Emails=-
GRR needs to be able to send emails for various logging and
alerting functions. The email domain will be appended to GRR
usernames when sending emails to users.



-=Monitoring/Email Domain=-
Emails concerning alerts or updates must be sent to this domain.

Email Domain e.g example.com [localhost]:【✅ Enter】


-=Alert Email Address=-
Address where monitoring events get sent, e.g. crashed clients,
broken server, etc.

Alert Email Address [grr-monitoring@localhost]:【✅ Enter】


-=Emergency Email Address=-
Address where high priority events such as an emergency ACL bypass are sent.

Emergency Access Email Address [grr-emergency@localhost]:【✅ Enter】

Step 2: Key Generation
All keys will have a bit length of 2048.
Generating executable signing key
Generating CA keys
Generating Server keys
Generating secret key for csrf protection.

Writing configuration to /etc/grr//server.local.yaml.
I0909 13:16:27.637845 139789273700160 config_lib.py:470] Writing back configuration to file /etc/grr//server.local.yaml
Initializing the datastore.
I0909 13:16:27.812390 139789273700160 server_logging.py:186] Writing log file to /usr/share/grr-server/lib/python3.6/site-packages/grr_response_core/var/log//GRRlog.txt

Step 3: Adding GRR Admin User
Please enter password for user 'admin':【✅ 管理者パスワートの登録（Webダッシュボードにログインするために必要）】

Step 4: Repackaging clients with new configuration.
Server debs include client templates. Re-download templates? [yN]:  [N]:【✅ Enter】
Repack client templates? [Yn]:  [Y]:【✅ Enter】

（略）

GRR Initialization complete! You can edit the new configuration in /etc/grr//server.local.yaml.

Please restart the service for the new configuration to take effect.

#################################################################
Install complete.
If upgrading, make sure you read the release notes:
https://grr-doc.readthedocs.io/en/latest/release-notes.html
#################################################################
Processing triggers for libc-bin (2.27-3ubuntu1) ...
```
</div>

　これでGRRサーバの構築が完了しました。  

　ちなみに設定ファイルは「`/etc/grr/server.local.yaml`」にあります。  
このファイルを編集することで先ほど設定した内容を変更することができます。  

　以下が設定ファイルの中身です。  
<div class="md-code" style="width:100%">
```
$ sudo cat /etc/grr/server.local.yaml
Database.implementation: MysqlDB
Blobstore.implementation: DbBlobStore
Mysql.host: localhost
Mysql.port: 3306
Mysql.database: grr
Mysql.database_name: grr
Mysql.username: grr
Mysql.database_username: grr
Mysql.password: grr_pass
Mysql.database_password: grr_pass
Client.server_urls:
- http://192.168.56.15:8080/
Frontend.bind_port: 8080
AdminUI.url: http://192.168.56.15:8000
AdminUI.port: 8000
Logging.domain: localhost
Monitoring.alert_email: grr-monitoring@localhost
Monitoring.emergency_access_email: grr-emergency@localhost
PrivateKeys.executable_signing_private_key: '-----BEGIN RSA PRIVATE KEY-----

（以下略）
```
</div>

#### GRRサーバの起動の確認

　念のためGRRサーバが起動していることの確認。  

<div class="md-code" style="width:100%">
```
$ sudo systemctl status grr-server
● grr-server.service - GRR Service
   Loaded: loaded (/lib/systemd/system/grr-server.service; enabled; vendor preset: enabled)
   Active: active (exited) since Wed 2020-09-09 13:14:50 JST; 7min ago
     Docs: https://github.com/google/grr
  Process: 30748 ExecStart=/bin/systemctl --no-block start grr-server@admin_ui.service grr-server@frontend.service grr-server@worker.service grr-server@worker2.service fleetspeak-server.service (code=exit
 Main PID: 30748 (code=exited, status=0/SUCCESS)

 9月 09 13:14:50 user1-VirtualBox systemd[1]: Starting GRR Service...
 9月 09 13:14:50 user1-VirtualBox systemd[1]: Started GRR Service.
```
</div>

　動いているっぽい。  

### GRRクライアントの導入

　次はフォレンジック対象端末にGRRクライアントを導入します。

　GRRサーバに比べるとGRRクライアントの導入は簡単です。

▼ debパッケージを使用したエージェント導入マニュアル  
[https://grr-doc.readthedocs.io/en/latest/deploying-grr-clients/on-linux.html:embed:cite]  

#### エージェント（debパッケージ）の取得・インストール

　まずはエージェントをダウンロードするために、GRRサーバのダッシュボート(`http://192.168.56.15:8000`)にアクセスします。  

　ダッシュボードにログインするための認証情報はGRRサーバのインストール中に設定したものです。  

　トップページは以下のようになります。  
[f:id:motikan2010:20200909205231p:plain:w700]

　左サイドバーの「Binaries」を押下することで、各環境のエージェントが表示されます。  
debパッケージからインストールするので、deb拡張子のファイルを押下してダウンロードします。  

[f:id:motikan2010:20200909205236p:plain:w700]

　 あとは以下の手順をするだけです。  

<div class="md-code" style="width:100%">
```
# ゲストOS側へdebファイルをコピー
$ scp grr_3.4.2.0_amd64.deb user2@192.168.56.16:/tmp/

# エージェントをインストール
$ cd /tmp/
$ sudo dpkg -i grr_3.4.2.0_amd64.deb
```
</div>

　これでクライアント側へのエージェントの導入は完了です。  
特に設定も必要ありません。

#### エージェントの起動の確認

　念のためエージェントが起動していることの確認。 

<div class="md-code" style="width:100%">
```
$ sudo systemctl status grr
● grr.service - grr linux amd64
   Loaded: loaded (/lib/systemd/system/grr.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-09-09 13:57:41 JST; 1min 52s ago
 Main PID: 5501 (grrd)
    Tasks: 6 (limit: 2332)
   CGroup: /system.slice/grr.service
           ├─5501 /usr/lib/grr/grr_3.4.2.0_amd64/grrd --config=/usr/lib/grr/grr_3.4.2.0_amd64/grrd.ya
           └─5502 /usr/lib/grr/grr_3.4.2.0_amd64/grrd --config=/usr/lib/grr/grr_3.4.2.0_amd64/grrd.ya

 9月 09 13:57:41 user2-VirtualBox systemd[1]: Started grr linux amd64.
 9月 09 13:57:43 user2-VirtualBox grrd[5501]: I0909 13:57:43.106781 140121068840768 client_logging.py
```
</div>

　こちらも動いているっぽいです。

## 動作確認

　GRRに馴染みのない方が多いと思われるので、キャプチャ画像を使って説明していきます。  

　まずは、先ほどエージェントをインストールしたGRRクライアント情報を見てみます。  

　ダッシュボート上部の「虫眼鏡ボタン」を押下します。左のテキストエリアは空白でよいです。  
[f:id:motikan2010:20200909205241p:plain:w700]

　GRRクライアント一覧が表示されます。  
今回は１台のサーバにしか導入していないので、１行のみ表示されています。  

　該当のサーバの行を押下すると、そのサーバの詳細ページに遷移します。  
[f:id:motikan2010:20200909205245p:plain:w700]

　OS、NWインタフェースの情報などが表示されました。  
[f:id:motikan2010:20200909205248p:plain:w700]

　もちろんGRRこちらに表示された情報よりも有意義な情報を取得することができます。  
  
　今回は例として「該当サーバでListenしているプロセス一覧」と「指定ユーザのWebブラウザ閲覧履歴」の２パターンを見てみます。  

### Listenしているプロセス一覧の取得

　左サイドバーの「Start new flows」→ 「Processes - ListProcesses」を開きます。  

この画面ではクライアントのプロセス情報を取得することができます。  

　今回は Listenしているプロセス一覧 を取得するので、 Connection states に「LISTEN」を選択します。  
選択後は、「Launch」ボタンを押下してください。  

[f:id:motikan2010:20200909213448p:plain:w700]  

　下の画面でしばらく待ちます...。上部の通知ボタンが赤くなったら、取得処理が完了したことを表します。  
赤くなったボタンを押下します。
[f:id:motikan2010:20200909213451p:plain:w700]  
  
　そうすると今までの取得処理の一覧が表示されいます。  
私の画面では何回か取得処理を動かしたので複数表示されています。  

　一番上のものが最新の取得処理なので先ほど動作させたものなります。  
「4 results」と記載されているため、プロセス情報の取得処理は成功していることが分かります。  

　プロセスの詳細も確認したいので、取得処理の部分（赤枠）を押下します。  
[f:id:motikan2010:20200909213455p:plain:w700]    

　プロセスの詳細が表示されました。  
画面には１つのプロセス情報しか収めていませんが、４つプロセス情報が表示されています。  

　対象のサーバでは80番ポートが動作しているようです。  
記述していませんが、裏でncコマンドで80番ポートをListenしていました。  

　Pidも取得できたので、メモリダンプも取得できそうです。  
ですが、今回はさわりだけですので、そこまでは行いません。  
[f:id:motikan2010:20200909213500p:plain:w700]

　念のためGRRクライアントに入ってListenしているプロセス見てみます。  

４つプロセスが表示されており、Web上で確認した情報は正しそうです。  
[f:id:motikan2010:20200909213506p:plain:w700]  

### ブラウザの閲覧履歴を取得

　次は趣向を変えてユーザのブラウザの閲覧履歴を取得してみることにします。  

　左サイドバーの「Start new flows」 → 「Browser - FirefoxHistory」を開きます。  

　この画面ではFirefoxブラウザの閲覧履歴を取得することができます。  

　試しに「user2」のブラウザの閲覧履歴を取得してみます。  

[f:id:motikan2010:20200909213509p:plain]  

　取得結果は以下のようになっています。  
「cat+image」のような文字列があることから、user2は猫の画像をググったことが確認されました。  
[f:id:motikan2010:20200909213513p:plain]  

　このように特定のサーバだけではなく、人が利用するシステム（今回はWebブラウザ）の痕跡なども確認できるようになっています。  

本記事では２種類の機能しか紹介できませんでしたが、他にもさまざまな項目を遠隔地から確認できるようになっています。  

## まとめ

　今回初めて「GRR Rapid Response」をさわってみました。  
クライアントを導入したサーバが１台ということもあり、そこまでこのツールの恩恵を授かることはできていなかったと思っております。  

　導入することが本記事のメインであったため、様々な機能にさわることができていないのが残念ですが、機能項目を見てみたところまだまだ楽しそうな機能がたくさんありました。  
　次はクライアントの台数を増やしたり、別のOSのクライアントで試したりしたいと思っております。  

本格的なフォレンジック機能もあると思いますので、その辺りもさわっていく予定です。

　<s>とはいっても『〇〇編』と銘打って続きを書いたことが(ry</s>  

　<span style="color: #ff0000"><b>続きを書きました。</b></span>YARAが使えるとのこと。  
[https://blog.motikan2010.com/entry/2020/09/11/%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3_GRR_Rapid_Response_YARA:embed:cite]  

## 更新履歴

- 2020年9月10日 新規作成
- 2020年9月11日 YARA編のリンク追加
