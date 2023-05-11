<div style="text-align: center;">[f:id:motikan2010:20200129012730p:plain]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　まず「依存ライブラリのセキュリティチェックツール」とは？ということですが、このようなツールの呼び方が定まっていないため本記事ではこのような呼び方にしています。  
（強いて言えば、Software Composition Analysツールと呼ばれているツール群）  
  
これらのツールの特徴として、動作しているアプリケーションが<span class="m-y">依存しているライブラリやパッケージに既知の脆弱性が存在していないかをバージョン情報などをもとに検査</span>します。脆弱性があるバージョンのライブラリがあるとアラートを出す挙動をします。  

　従来から利用されている Burp Suite や OWASP ZAP などのWebセキュリティ診断ツールは実際にアプリケーションを動作させて、その挙動から脆弱性を検出します。それに対して「依存ライブラリのセキュリティチェックツール」は<span class="m-y">アプリケーションを動作させずに検査</span>します。ツールの動作が全く異なるので、検査内容も大きく違います。  

　どちらかといったらWordPressの検査に利用される WPScan に検査内容は近いです。どちらも利用しているプラグイン(ライブラリ)のバージョン情報で検出するのがメインとなっているからです。  
それでも検査を実施できる範囲は異なります。  
　WPScan はネット越しに公開されているサイト全てに検査することが可能です。対して「依存ライブラリのセキュリティチェックツール」はアプリケーションが<span class="m-y">依存しているライブラリが記述されているファイル（Gemfile.lock や composer.lock）が検査対象</span>になりますので、それらのファイルを閲覧できる権限者のみが検査を実施できます。それらのファイルに依存関係はほぼ全て記述されているので、依存関係を全て取得するのが難しいネット越しでの検査に比べてより厳密な検査ができます。  
  
  
　そして近頃、Webアプリケーションなどのソフトウェアに依存しているライブラリに既知の脆弱性が潜んでいないかを検査するために、セキュリティチェックツールを提供・利用するサービスが増えてきている気がしています。

▼ Snyk もセキュリティチェックツールを開発/提供している企業であり資金調達が上手くいっている模様。
[https://jp.techcrunch.com/2020/01/22/2020-01-21-snyk-snags-150m-investment-as-its-valuation-surpasses-1b/:embed:cite]

　まずは、手軽に導入することができるOSSのセキュリティチェックツールを使用してみて、どのような検査結果を得ることができるのかなどを確認してみます。  
ついでに検出された脆弱性の情報はどこから取得しているのかなども確認してきます。  

<!-- more -->

## 言語別 ツール一覧

　昨今では様々な言語に対応したセキュリティツールが存在しています。  
本記事では「<span class="m-y">PHP</span>」「<span class="m-y">Ruby</span>」「<span class="m-y">Python</span>」「<span class="m-y">Java</span>」「<span class="m-y">JavaScript</span>」の5つの言語に対応したツールを各言語一つづつ紹介していきます。   
  
　今回紹介するツールはどれもOSSなのでセキュリティ関連のツールに興味のある人はコードを読んでみても良いかと。  

### PHP

　まず初めにPHPのセキュリティチェックツールを使ってみます。  
  
有名なものに <span class="m-y">security-checker</span> がありますので、これを紹介します。

#### sensiolabs / security-checker

　検査時に`composer.lock`ファイルを指定することから、<span class="m-y">`composer.lock`に記述されたPHPの依存ライブラリのバージョン情報から脆弱性を検出している</span>ことが分かります。

[https://github.com/sensiolabs/security-checker:embed:cite]

[f:id:motikan2010:20200128223937p:plain]  


##### 使い方

　検査用のスクリプトを取得したら、１つのコマンドでセキュリティの検査が可能です。  

<div class="sm-code">
```
# security-checker のスクリプトを取得
$ wget https://get.sensiolabs.org/security-checker.phar

# 検査の開始
$ php security-checker.phar security:check composer.lock
Symfony Security Check Report
=============================

2 packages have known vulnerabilities.

symfony/http-foundation (v3.4.32)
---------------------------------

 * [CVE-2019-18888][]: Prevent argument injection in a MimeTypeGuesser

symfony/http-kernel (v3.4.32)
-----------------------------

 * [CVE-2019-18887][]: Use constant time comparison in UriSigner

[CVE-2019-18888]: https://symfony.com/cve-2019-18888
[CVE-2019-18887]: https://symfony.com/cve-2019-18887

Note that this checker can only detect vulnerabilities that are referenced in the SensioLabs security advisories database.
Execute this command regularly to check the newly discovered vulnerabilities.
```
</div>
　検査の結果、「<span><a href="https://symfony.com/cve-2019-18887" target="_blank">CVE-2019-18888</a></span>」と「<span><a href="https://symfony.com/cve-2019-18888" target="_blank">CVE-2019-18887</a></span>」の脆弱性が「symfony/http-foundation」と「symfony/http-kernel」から検出されました。  

##### 脆弱性データベース

[https://github.com/FriendsOfPHP/security-advisories:embed:cite]

　プルリクベースで新しい脆弱性の追加などの更新が行われており、今後も引き続き更新は行われていきそうです。  

他の言語のセキュリティチェックツールも同様に、ほとんどの脆弱性データベースは様々に人によって頻繁に更新されています。  


##### Web版

　Web版も用意されており、インストールなども必要なく利用することができます。  

検査対象となる`composer.lock`ファイルをアップロードすることで検査が可能です。  

[https://security.symfony.com/:embed:cite]  

▼ 検査結果  
[f:id:motikan2010:20200128224042p:plain:w400]  

### Ruby

#### rubysec / bundler-audit

[https://github.com/rubysec/bundler-audit:embed:cite]

[f:id:motikan2010:20200128224105p:plain]  

##### 使い方

　試しに適当にRailsプロジェクトを作成し、そのプロジェクトに対して検査を実施していきます。  

<div class="sm-code">
```
# bundler-audit をインストール
$ gem install bundler-audit

# 検査対象のディレクトリに移動
$ cd ~/RubymineProjects/railsTestApp/

# 検査を実施
$ bundle-audit
bundle-audit
Name: actionview
Version: 5.1.6
Advisory: CVE-2019-5419
Criticality: Unknown
URL: https://groups.google.com/forum/#!topic/rubyonrails-security/GN7w9fFAQeI
Title: Denial of Service Vulnerability in Action View
Solution: upgrade to >= 6.0.0.beta3, >= 5.2.2.1, ~> 5.2.2, >= 5.1.6.2, ~> 5.1.6, >= 5.0.7.2, ~> 5.0.7, >= 4.2.11.1, ~> 4.2.11

Name: actionview
Version: 5.1.6
Advisory: CVE-2019-5418
Criticality: Unknown
URL: https://groups.google.com/forum/#!topic/rubyonrails-security/pFRKI96Sm8Q
Title: File Content Disclosure in Action View
Solution: upgrade to >= 4.2.11.1, ~> 4.2.11, >= 5.0.7.2, ~> 5.0.7, >= 5.1.6.2, ~> 5.1.6, >= 5.2.2.1, ~> 5.2.2, >= 6.0.0.beta3
```
</div>

<div onclick="obj=document.getElementById('20200128_1').style; obj.display=(obj.display=='none')?'block':'none';">
<a style="cursor:pointer;">クリックで展開</a>
</div>
<div id="20200128_1" style="display:none;clear:both;">
<div class="sm-code">
```
Name: activejob
Version: 5.1.6
Advisory: CVE-2018-16476
Criticality: Unknown
URL: https://groups.google.com/forum/#!topic/rubyonrails-security/FL4dSdzr2zw
Title: Broken Access Control vulnerability in Active Job
Solution: upgrade to ~> 4.2.11, ~> 5.0.7.1, ~> 5.1.6.1, ~> 5.1.7, >= 5.2.1.1

Name: devise
Version: 4.4.3
Advisory: CVE-2019-5421
Criticality: High
URL: https://github.com/plataformatec/devise/issues/4981
Title: Devise Gem for Ruby Time-of-check Time-of-use race condition with lockable module
Solution: upgrade to >= 4.6.0

Name: devise
Version: 4.4.3
Advisory: CVE-2019-16109
Criticality: Unknown
URL: https://github.com/plataformatec/devise/issues/5071
Title: Devise Gem for Ruby confirmation token validation with a blank string
Solution: upgrade to >= 4.7.1

Name: loofah
Version: 2.2.2
Advisory: CVE-2019-15587
Criticality: Unknown
URL: https://github.com/flavorjones/loofah/issues/171
Title: Loofah XSS Vulnerability
Solution: upgrade to >= 2.3.1

Name: loofah
Version: 2.2.2
Advisory: CVE-2018-16468
Criticality: Unknown
URL: https://github.com/flavorjones/loofah/issues/154
Title: Loofah XSS Vulnerability
Solution: upgrade to >= 2.2.3

Name: nokogiri
Version: 1.8.3
Advisory: CVE-2019-5477
Criticality: High
URL: https://github.com/sparklemotion/nokogiri/issues/1915
Title: Nokogiri Command Injection Vulnerability via Nokogiri::CSS::Tokenizer#load_file
Solution: upgrade to >= 1.10.4

Name: nokogiri
Version: 1.8.3
Advisory: CVE-2019-11068
Criticality: Unknown
URL: https://github.com/sparklemotion/nokogiri/issues/1892
Title: Nokogiri gem, via libxslt, is affected by improper access control vulnerability
Solution: upgrade to >= 1.10.3

Name: nokogiri
Version: 1.8.3
Advisory: CVE-2018-14404
Criticality: Unknown
URL: https://github.com/sparklemotion/nokogiri/issues/1785
Title: Nokogiri gem, via libxml2, is affected by multiple vulnerabilities
Solution: upgrade to >= 1.8.5

Name: nokogiri
Version: 1.8.3
Advisory: CVE-2019-13117
Criticality: Unknown
URL: https://github.com/sparklemotion/nokogiri/issues/1943
Title: Nokogiri gem, via libxslt, is affected by multiple vulnerabilities
Solution: upgrade to >= 1.10.5

Name: omniauth
Version: 1.8.1
Advisory: CVE-2015-9284
Criticality: High
URL: https://github.com/omniauth/omniauth/pull/809
Title: CSRF vulnerability in OmniAuth's request phase
Solution: remove or disable this gem until a patch is available!

Name: puma
Version: 3.11.4
Advisory: CVE-2019-16770
Criticality: High
URL: https://github.com/puma/puma/security/advisories/GHSA-7xx3-m584-x994
Title: Keepalive thread overload/DoS in puma
Solution: upgrade to ~> 3.12.2, >= 4.3.1

Name: rack
Version: 2.0.5
Advisory: CVE-2018-16470
Criticality: Unknown
URL: https://groups.google.com/forum/#!topic/ruby-security-ann/Dz4sRl-ktKk
Title: Possible DoS vulnerability in Rack
Solution: upgrade to >= 2.0.6

Name: rack
Version: 2.0.5
Advisory: CVE-2018-16471
Criticality: Unknown
URL: https://groups.google.com/forum/#!topic/ruby-security-ann/NAalCee8n6o
Title: Possible XSS vulnerability in Rack
Solution: upgrade to ~> 1.6.11, >= 2.0.6

Name: rack
Version: 2.0.5
Advisory: CVE-2019-16782
Criticality: Unknown
URL: https://github.com/rack/rack/security/advisories/GHSA-hrqr-hxpp-chr3
Title: Possible information leak / session hijack vulnerability
Solution: upgrade to ~> 1.6.12, >= 2.0.8

Name: rubyzip
Version: 1.2.1
Advisory: CVE-2019-16892
Criticality: Unknown
URL: https://github.com/rubyzip/rubyzip/pull/403
Title: Denial of Service in rubyzip ("zip bombs")
Solution: upgrade to >= 1.3.0

Name: rubyzip
Version: 1.2.1
Advisory: CVE-2018-1000544
Criticality: Unknown
URL: https://github.com/rubyzip/rubyzip/issues/369
Title: Directory Traversal in rubyzip
Solution: upgrade to >= 1.2.2

Vulnerabilities found!
```
</div>
</div>

　検出した脆弱性は17個あり列挙するとこんな感じです。  
<div class="sm-code">
```
CVE-2019-5419    CVE-2019-5418  CVE-2018-16476  CVE-2019-5421  CVE-2019-16109
CVE-2019-15587  CVE-2018-16468  CVE-2019-5477  CVE-2019-11068  CVE-2018-14404
CVE-2019-13117  CVE-2015-9284  CVE-2019-16770  CVE-2018-16470  CVE-2019-16782
CVE-2019-16892  CVE-2018-1000544
```
</div>

　昨年発見された rack(CVE-2019-16782)の脆弱性も検出されており、検出に用いられる脆弱性データベースも更新されていることが分かります。  
  
　次は脆弱性データベースを見てみることにします。

#### 脆弱性データベース - rubysec / ruby-advisory-db

　検出のために用いられている脆弱性データベースは 「rubysec / ruby-advisory-db」リポジトリにあります。  
[https://github.com/rubysec/ruby-advisory-db:embed:cite]  

### Python

#### pyupio / safety

[https://github.com/pyupio/safety:embed:cite]  

[f:id:motikan2010:20200128224129p:plain]

##### 脆弱性データベース

- [pyupio/safety-db: A curated database of insecure Python packages](https://github.com/pyupio/safety-db)

##### 使い方

<div class="sm-code">  
```
# 脆弱性データベースを取得
$ pip install safety-db

# safetyのインストール
$ pip install safety

# 依存ライブラリの確認。古いDjangoがあります。
$ cat requirements.txt
Django==1.7.5

# 検査の開始
$ safety check -r requirements.txt
╒══════════════════════════════════════════════════════════════════════════════╕
│                                                                              │
│                               /$$$$$$            /$$                         │
│                              /$$__  $$          | $$                         │
│           /$$$$$$$  /$$$$$$ | $$  \__//$$$$$$  /$$$$$$   /$$   /$$           │
│          /$$_____/ |____  $$| $$$$   /$$__  $$|_  $$_/  | $$  | $$           │
│         |  $$$$$$   /$$$$$$$| $$_/  | $$$$$$$$  | $$    | $$  | $$           │
│          \____  $$ /$$__  $$| $$    | $$_____/  | $$ /$$| $$  | $$           │
│          /$$$$$$$/|  $$$$$$$| $$    |  $$$$$$$  |  $$$$/|  $$$$$$$           │
│         |_______/  \_______/|__/     \_______/   \___/   \____  $$           │
│                                                          /$$  | $$           │
│                                                         |  $$$$$$/           │
│  by pyup.io                                              \______/            │
│                                                                              │
╞══════════════════════════════════════════════════════════════════════════════╡
│ REPORT                                                                       │
│ checked 5 packages, using default DB                                         │
╞════════════════════════════╤═══════════╤══════════════════════════╤══════════╡
│ package                    │ installed │ affected                 │ ID       │
╞════════════════════════════╧═══════════╧══════════════════════════╧══════════╡
│ django                     │ 1.7.5     │ <1.7.11                  │ 25714    │
│ django                     │ 1.7.5     │ <1.7.6                   │ 25715    │
│ django                     │ 1.7.5     │ <1.8.10                  │ 33074    │
│ django                     │ 1.7.5     │ <1.8.10                  │ 33073    │
│ django                     │ 1.7.5     │ <1.8.15                  │ 25718    │
│ django                     │ 1.7.5     │ >=1.7,<1.7.10            │ 25728    │
│ django                     │ 1.7.5     │ >=1.7,<1.7.10            │ 25727    │
│ django                     │ 1.7.5     │ >=1.7,<1.7.7             │ 25731    │
│ django                     │ 1.7.5     │ >=1.7,<1.7.7             │ 25713    │
│ django                     │ 1.7.5     │ >=1.7,<1.7.9             │ 25725    │
│ django                     │ 1.7.5     │ >=1.7,<1.7.9             │ 25726    │
╘══════════════════════════════════════════════════════════════════════════════╛
```
</div>  

　いくつかの脆弱性が検出されました。しかし脆弱性の詳細情報は出力されていない状態です。  
`--full-report`オプションを付与することで、脆弱性の詳細情報を出力することができます。  

##### 詳細情報レポートの取得。 --full-report オプション

　`--full-report`オプションを付与することで、検出した脆弱性の詳細を表示することが可能です。

<div class="sm-code">  
```
$ safety check -r requirements.txt --full-report
╒══════════════════════════════════════════════════════════════════════════════╕
│                                                                              │
│                               /$$$$$$            /$$                         │
│                              /$$__  $$          | $$                         │
│           /$$$$$$$  /$$$$$$ | $$  \__//$$$$$$  /$$$$$$   /$$   /$$           │
│          /$$_____/ |____  $$| $$$$   /$$__  $$|_  $$_/  | $$  | $$           │
│         |  $$$$$$   /$$$$$$$| $$_/  | $$$$$$$$  | $$    | $$  | $$           │
│          \____  $$ /$$__  $$| $$    | $$_____/  | $$ /$$| $$  | $$           │
│          /$$$$$$$/|  $$$$$$$| $$    |  $$$$$$$  |  $$$$/|  $$$$$$$           │
│         |_______/  \_______/|__/     \_______/   \___/   \____  $$           │
│                                                          /$$  | $$           │
│                                                         |  $$$$$$/           │
│  by pyup.io                                              \______/            │
│                                                                              │
╞══════════════════════════════════════════════════════════════════════════════╡
│ REPORT                                                                       │
│ checked 5 packages, using default DB                                         │
╞════════════════════════════╤═══════════╤══════════════════════════╤══════════╡
│ package                    │ installed │ affected                 │ ID       │
╞════════════════════════════╧═══════════╧══════════════════════════╧══════════╡
│ django                     │ 1.7.5     │ <1.7.11                  │ 25714    │
╞══════════════════════════════════════════════════════════════════════════════╡
│ The get_format function in utils/formats.py in Django before 1.7.x before    │
│ 1.7.11, 1.8.x before 1.8.7, and 1.9.x before 1.9rc2 might allow remote       │
│ attackers to obtain sensitive application secrets via a settings key in      │
│ place of a date/time format setting, as demonstrated by SECRET_KEY.          │
╞══════════════════════════════════════════════════════════════════════════════╡
│ django                     │ 1.7.5     │ <1.7.6                   │ 25715    │
╞══════════════════════════════════════════════════════════════════════════════╡
│ Cross-site scripting (XSS) vulnerability in the contents function in         │
│ admin/helpers.py in Django before 1.7.6 and 1.8 before 1.8b2 allows remote   │
│ attackers to inject arbitrary web script or HTML via a model attribute in    │

（以下省略）
```
</div>  

### Java

#### jeremylong / DependencyCheck

[https://github.com/jeremylong/DependencyCheck:embed:cite]  

　OWASPツールのDependencyCheckですが、MavenやGradleなどの様々なJavaプロジェクトに対応しています。  
  
今回はGradleのプロジェクトを対象にツールを使用しますので、「`dependency-check-gradle`」というライブラリをGradleに追記します。

#### Gradle用 jeremylong/dependency-check-gradle

[https://github.com/jeremylong/dependency-check-gradle:embed:cite]  

　使用方法は簡単で「`./gradlew dependencyCheckAnalyze`」コマンドを実行するだけです。  
  
検査が開始されますので、しばらくすると脆弱性が存在しているライブラリの一覧が表示されます。  

[f:id:motikan2010:20200128224152p:plain]  

　整形されずに出力されるので少々見づらいですが、このツールの良い点としてHTML形式のレポートが自動的に保存されるようになっている。  
私の環境だと「`./build/reports/dependency-check-report.html`」に保存されていました。  
出力内容をブラウザで確認してみます。  

[f:id:motikan2010:20200128224212p:plain:w400]  
[f:id:motikan2010:20200128224225p:plain:w400]  

　ブラウザ上での確認だと非常に見やすく、脆弱性の詳細情報への遷移もラクで便利です。  


### JavaScript

#### Npm標準ツール - npm-audit

[https://github.com/npm/cli/blob/b829d62c98506325d2afb2d85d191a8ff1c49157/docs/content/cli-commands/npm-audit.md:embed:cite]  

[f:id:motikan2010:20200128224238p:plain]

npmコマンドに標準で付属されている`npm audit`サブコマンドです。  

##### 脆弱なパッケージをアップデートする " npm audit fix " コマンド

　`npm audit fix`コマンドを使用することで、脆弱性見つかっているパッケージのアップデートを行うことができます。  
　以下がそのコマンドを使用してパッケージのアップデートが行われるまでの流れです。  
 
 <div class="sm-code">   
```
$ cat package-lock.json | grep '"hot-formula-parser"' -A 1
    "hot-formula-parser": {
      "version": "3.0.0",
```
</div>  

　`npm audit fix`コマンドを実行します。1つの脆弱性が見つかり、修正した旨のメッセージが表示されました。  
<div class="sm-code"> 
```
$ npm audit fix
npm WARN vuln_hot-formula-parser_RCE No repository field.
npm WARN vuln_hot-formula-parser_RCE No license field.

+ hot-formula-parser@3.0.2
updated 1 package in 0.316s
fixed 1 of 1 vulnerability in 5 scanned packages
```
</div>  

　依存パッケージを確認してみると、バージョンが上がっていることが確認できます。  
<div class="sm-code">  
```
$ cat package-lock.json | grep '"hot-formula-parser"' -A 1
    "hot-formula-parser": {
      "version": "3.0.2",
```
</div>  

　確認のために再度`npm audit`コマンドを実行します。  
脆弱性は見つからず、修正されたことが確認できます。
<div class="sm-code">  
```
$ npm audit

                       === npm audit security report ===

found 0 vulnerabilities
 in 5 scanned packages
```
</div>  

## まとめ

　どのツールもリポジトリが依存するライブラリのバージョンを利用して検査しています。  
そのため実際にアプリケーションが動作する環境に影響を及ぼすことなくセキュリティの検査を実施することが可能です。  
 今回紹介したツールは、主に各自の環境にインストールして利用するものでした。しかし、そのような手間をかけるかけることなく「検査対象のリポジトリを登録するだけで脆弱性が存在しているライブラリが存在する場合にアラートを出してくれるサービス」も存在しています。  
  
　一番身近なものはGitHubだと思います。リポジトリが依存しているライブラリに脆弱性が見つかったら下画像のようにアラートが表示されます。
これは脆弱性が放置されている良くない例です・・・。ちなみにリポジトリ所有者のみにアラートが見えるようになっています。  
[f:id:motikan2010:20200129013022p:plain:w400]  

　Github以外にも <span><a href="https://snyk.io/" target="_blank">Snyk</a></span> や <span><a href="https://yamory.io/" target="_blank">yamory</a></span> といったサービスがあります。  
どちらも無料プランがあり、導入も簡単であることから両方利用していますがどちらも使いやすくオススメです。趣味で開発している人も使ってみるとよいかと。

## 更新履歴

- 2019年1月29日 新規作成
