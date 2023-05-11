<div style="text-align:center;">[f:id:motikan2010:20220113114159p:plain]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　2021年9月1日、`火线安全平台`社から「DongTai(ドンタイ) IAST ("洞态IAST"の表記もある)」がOSSとして公開されました。  
(IAST = Interactive Application Security Testing)

<figure class="figure-image figure-image-fotolife" title="製品ロゴ">[f:id:motikan2010:20220107192818p:plain]<figcaption>製品ロゴ</figcaption></figure>

　「グローバルプロフェッショナルIAST分野では初のオープンソースプロジェクト」と記載されていますが、Passive IASTでは世界初のOSSだと思います。  

[http://www.anquan419.com/news/18/735.html:embed:cite]

　SaaS版も公開されています。「`快速体验`」ページから環境を構築することもなくDongTai  IASTを試すことができます。

[https://dongtai.io/:embed:cite]

　本記事では DongTai IAST の導入を説明していきます。

　ドキュメントをメモしていたりするので、こちらもどうぞ。  
[https://zenn.dev/motikan2010/articles/b8a36a6e436d01:embed:cite]

## 環境構築

　以下の環境で動作確認を行いました。  

- DongTai **1.2.0**
- EC2 (t2.medium / メモリ4G)
  - Amazon Linux 2 (5.10.75-79.358.amzn2.x86_64)
- Docker 20.10.7
  - Docker Compose 1.29.2

### DongTaiをリポジトリから取得

<div class="md-code" style="width:100%">
```
# git clone https://github.com/HXSecurity/DongTai.git
# cd DongTai/deploy/docker-compose/
```
</div>

### DongTaiを起動

　まず`dtctl`スクリプトが動作することを確認します。以下のコマンドでヘルプが表示されます。
<div class="md-code" style="width:100%">
```
# ./dtctl
[Info] Usage:
[Usage]     ./dtctl -h                                          Display usage message
[Usage]     ./dtctl install -s mysql,redis  -v 1.0.5            Install iast server
[Usage]     ./dtctl remove|rm [-d]                              Uninstall iast server
[Usage]     ./dtctl upgrade -t 1.1.2                            Upgrade iast server
[Usage]     ./dtctl version                                     Get image version
[Usage]     ./dtctl dbhash                                      Get database schema hash
[Usage]     ./dtctl dbschema                                    Export database schema
[Usage]     ./dtctl dbrestore -f FILEPATH                       Restore mysql database
[Usage]     ./dtctl dbbackup  -d FILEPATH                       Backup mysql database
[Usage]     ./dtctl file                                        Export docker-compose.yml
[Usage]     ./dtctl logs webapi|openapi|web|mysql|web|engine    Extract tail logs
```
</div>

　問題なく動作することが確認できたらインストールを実施します。
<div class="md-code" style="width:100%">
```
# ./dtctl install
[Info] check docker servie status.
[Info] docker service is up.
[Info] check port status
[+] please input web service port, default [80]:
[Info] port 80 is ok.
[Info] Starting docker compose ...
Creating network "dongtai-iast_default" with the default driver
Pulling dongtai-mysql (dongtai/dongtai-mysql:1.2.0)...
1.2.0: Pulling from dongtai/dongtai-mysql
72a69066d2fe: Pull complete
93619dbc5b36: Pull complete
(...snip...)
Creating dongtai-iast_dongtai-engine-task_1 ... done
Creating dongtai-iast_dongtai-web_1         ... done
[Important] Installation success!
```
</div>

#### docker-compose.yml の中身

　DongTaiの動作には関係ないですが、起動しているコンテナを確認してみます。

　構成情報が記載されている`docker-compose.yml`は削除されるようになっていますが、`./dtctl file`コマンドを用いることで、`docker-compose.yml`を出力できます。

<div class="md-code" style="width:100%">
```
# ./dtctl file
# cat docker-compose.yml
```
</div>

　出力された`docker-compose.yml`の内容は以下になっています。

<div class="sm-code" style="width:100%">
```yml
version: "2"
services:
  dongtai-mysql:
    image: dongtai/dongtai-mysql:1.2.0
    restart: always
    volumes:
      - "/root/work/DongTai/deploy/docker-compose/data:/var/lib/mysql:rw"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
  dongtai-redis:
    image: dongtai/dongtai-redis:1.2.0
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
  dongtai-webapi:
    image: "dongtai/dongtai-webapi:1.2.0"
    restart: always
    volumes:
      - "/root/work/DongTai/deploy/docker-compose/config-tutorial.ini:/opt/dongtai/webapi/conf/config.ini"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"

  dongtai-web:
    image: "dongtai/dongtai-web:1.2.0"
    restart: always
    ports:
      - "80:80"
    volumes:
      - "/root/work/DongTai/deploy/docker-compose/nginx.conf:/etc/nginx/nginx.conf"
    depends_on:
      - dongtai-webapi
    logging:
      driver: "json-file"
      options:
        max-size: "10m"

  dongtai-openapi:
    image: "dongtai/dongtai-openapi:1.2.0"
    restart: always
    volumes:
       - "/root/work/DongTai/deploy/docker-compose/config-tutorial.ini:/opt/dongtai/openapi/conf/config.ini"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"

  dongtai-engine:
    image: "dongtai/dongtai-engine:1.2.0"
    restart: always
    volumes:
      - "/root/work/DongTai/deploy/docker-compose/config-tutorial.ini:/opt/dongtai/engine/conf/config.ini"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"


  dongtai-engine-task:
    image: "dongtai/dongtai-engine:1.2.0"
    restart: always
    command: ["/opt/dongtai/engine/docker/entrypoint.sh", "task"]
    volumes:
      - "/root/work/DongTai/deploy/docker-compose/config-tutorial.ini:/opt/dongtai/engine/conf/config.ini"
    depends_on:
      - dongtai-engine
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
```
</div>

## 動作確認

　実際に DongTai を使ってみます。

　現在以下の言語に対応しています。

- Java
- Python
- PHP (ベータ版)

　今回はJavaアプリケーションを用いていきます。

### IASTのJavaエージェントの取得

　DongTaiの起動が完了したらアクセスします。  

ログイン画面が表示されるので初期ログイン情報(`admin/admin`)を入力します。
[f:id:motikan2010:20220107075229p:plain]

　ログインするとDongTaiが導入されているアプリケーション一覧画面が表示されます。  

　まだDongTaiを導入しているアプリケーションがないため、空の状態になっています。

　右上の「＋ Add Agent」ボタンを押下すると、エージェントの導入画面へ遷移することができます。

[f:id:motikan2010:20220107075232p:plain]

　「DongTai Java Agent」ボタンを押下してJavaエージェント(`agent.jar`)をダウンロードします。

[f:id:motikan2010:20220107075234p:plain]

### 脆弱アプリケーションの起動

　 脆弱なアプリケーションには以下のSpring Bootアプリケーションを利用します。  
<span><a href="https://github.com/motikan2010/Vulnerability-Spring-Boot" target="_blank">motikan2010/Vulnerability-Spring-Boot</a></span>

#### ビルド

　起動前にビルドをします。

<div class="md-code" style="width:100%">

```
$ mvn clean package -Dmaven.test.skip=true
```

</div>

#### 起動

　起動時のコマンドに「`-javaagent:agent.jar`」オプション含めることで、DongTaiのJavaエージェントをロードします。  

<div class="md-code" style="width:100%">

```
$ java -javaagent:agent.jar -jar target/vuln-0.0.1-SNAPSHOT.jar
```

</div>

　起動が成功すると起動ログに「`[io.dongtai.agent]~`」が表示されます。

<div class="md-code" style="width:100%">

```
$ java -javaagent:agent.jar -jar target/vuln-0.0.1-SNAPSHOT.jar
[io.dongtai.agent] The engine configuration file is initialized successfully. file is /Users/motikan2010/IdeaProjects/Vulnerability-Spring-Boot/config/iast.properties
[io.dongtai.agent] register agent
[io.dongtai.agent] Agent has successfully registered with http://3.xxx.yyy.zzz/openapi
[io.dongtai.agent] engine delay time is 0 s
[io.dongtai.agent] Check if the engine[/var/folders/yw/0wl7x4w56tz9326ncdt107jr0000gn/T//iast-inject.jar] needs to be updated
[io.dongtai.agent] Engine does not exist in local cache, the engine will be downloaded.
[io.dongtai.agent] The remote file http://3.xxx.yyy.zzz/openapi/api/v1/engine/download?engineName=iast-inject was successfully written to the local cache.
[io.dongtai.agent] The remote file http://3.xxx.yyy.zzz/openapi/api/v1/engine/download?engineName=iast-core was successfully written to the local cache.
2022-01-07 06:52:05.870 [cn.huoxian.dongtai.engine] INFO  DongTai Engine is about to be installed, the installation mode is agent
2022-01-07 06:52:05.879 [cn.huoxian.dongtai.engine] INFO  Initialize the core configuration of the engine
2022-01-07 06:52:06.456 [cn.huoxian.dongtai.engine] INFO  The engine's core configuration is initialized successfully.
2022-01-07 06:52:06.456 [cn.huoxian.dongtai.engine] INFO  Start the data reporting submodule
2022-01-07 06:52:06.457 [cn.huoxian.dongtai.engine] INFO  The data reporting submodule started successfully
2022-01-07 06:52:06.458 [cn.huoxian.dongtai.engine] INFO  Register spy submodule
2022-01-07 06:52:06.461 [cn.huoxian.dongtai.engine] INFO  Spy sub-module registered successfully
2022-01-07 06:52:06.461 [cn.huoxian.dongtai.engine] INFO  Install data acquisition and analysis sub-modules
2022-01-07 06:52:07.392 [cn.huoxian.dongtai.engine] INFO  The sub-module of data acquisition and analysis is successfully installed
2022-01-07 06:52:07.394 [cn.huoxian.dongtai.engine] INFO  DongTai Engine is successfully installed to the JVM, and it takes 1 s
2022-01-07 06:52:07.394 [cn.huoxian.dongtai.engine] INFO  Turn on the engine
2022-01-07 06:52:07.395 [cn.huoxian.dongtai.engine] INFO  Engine opened successfully
[io.dongtai.agent] DongTai engine start successfully.
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
(...snip...)
```
</div>

　アプリケーションの`/lfi`にアクセスし、「`sample.txt`」を入力し、送信します。

　<span class="m-y">ここで重要なのは「`../../../../../etc/passwd`」といった攻撃コードではなく、正常値である「`sample.txt`」を入力している点です。</span>

[f:id:motikan2010:20220107075240p:plain]

### 検出した脆弱性の確認

　脆弱性が検出されたことを確認するために DongTai に戻ります。

　DongTai が導入されたアプリケーションが追加されていることが確認できます。

[f:id:motikan2010:20220107075238p:plain]

　「`应用漏洞`」(アプリケーションの脆弱性)から検出された脆弱性を確認することができます。

　検出された脆弱性内に「`/lfi 的GET 出现路径穿越漏洞,位置:GET/HEADER`」というものがあり、パストラバーサルが無事検出されたことが確認できます。  
（パストラバーサルは中国語表記だと"路径穿越"になるそうです。）

　脆弱性を押下することで、脆弱性の詳細情報を見ることできます。
[f:id:motikan2010:20220107075243p:plain]

　脆弱性の詳細情報は以下のように表示されます。

　脆弱性があるパラメータなどの情報があります。
[f:id:motikan2010:20220107075246p:plain]

　少し下に行くと脆弱性が検出されるまでの流れが表示されます。

　<span class="m-y">IASTの特徴である「污点来源 (source)」「传播方法 (propagator)」「危险方法 (sink)」を確認することできます。</span>

[f:id:motikan2010:20220107075250p:plain]

　ここまでが DongTai の導入方法・使い方でした。

## まとめ

- DongTai は OSS の IAST（ソースコードが見れる！）
- アプリケーションに導入し、正常系をテストするだけで脆弱性を検出することができる。
- IASTの実装を見ることができるのは珍しいので、いろいろ分かり次第「【導入篇】」の続きを書いていきたい次第。

## 豆知識

### Active IAST と Passive IAST の違い

　IASTは「Active IAST」と「Passive IAST」に分類されます。

　 2つの違いについては「開発者ファーストのアプリケーションセキュリティ完全ガイド」で説明されています。  
<span><a href="https://resources.github.com/downloads/eBook-GHESAdvancedSecurity.pdf" target="_blank">開発者ファーストのアプリケーションセキュリティ完全ガイド</a></span>

> 　IAST には2つのバリアントがあります。

> 　パッシブ IAST は、テスト環境で実行するアプリケーションに使用します。
アプリケーションでユースケースベースのQAテストを実行すると、エージェントがセキュリティの潜在的な脆弱性を特定します。
このアプローチでは、SAST または DAST を使用して検出できる脆弱性のサブセットも検出します。

> 　アクティブ IAST は、ライブ環境で実行しているアプリケーションに使用します。
これは DAST ツールの拡張機能として機能します。実行中のアプリケーションにエージェントがインストールされ、アプリケーションに対して DAST テストを実行します。
エージェントはスタックトレース情報を確認し、サーバー側で詳細な動作を分析できるため、DASTのプロセスと結果が改善されます。
アクティブ IAST は、スキャン時間を短縮し、DASTの攻撃結果を検証するのに役立ちます。

　<span class="m-y">DongTai IAST は Passive IAST に分類されます。</span>

　Active IAST にもOSSには「OpenRASP-IAST（Baidu社 開発）」があります。  
<span><a href="https://github.com/baidu-security/openrasp-iast" target="_blank">baidu-security/openrasp-iast: IAST 灰盒扫描工具</a></span>

### IASTの実装に関して

　IASTの実装に興味のある人は以下の記事がおすすめです。

- <span><a href="https://su18.org/post/dongtai/" target="_blank">洞态 IAST 试用 | 素十八</a></span>
  - セキュリティリサーチャーがDongTaiを試した感想、実装についてが書かれています。
- <span><a href="https://buaq.net/go-94229.html" target="_blank">浅谈被动式IAST产品与技术实现-代码实现Demo篇</a></span> (パッシブIASTの製品・技術実装の紹介 - コード実装デモ)
  - IASTの重要部分が抽出されたアプリケーションを用いて、IASTの実装について説明されています。
  - このサイトで紹介されているリポジトリ<span><a href="https://github.com/iiiusky/java_iast_example" target="_blank">iiiusky/java_iast_example: JAVA IAST Example</a></span>





