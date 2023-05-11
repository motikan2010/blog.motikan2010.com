<div style="text-align:center;">[f:id:motikan2010:20210207233948p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　本記事でやること。

- CodeQL を導入
- 脆弱なアプリケーションに対してのセキュリティテスト(XXEを検出)
- 検出結果を見てみる

### CodeQL とは

　CodeQL は SAST(Static application security testing) というセキュリティテスト手法を実現するためのツールです。  

　本ブログで何度か取りあげた GitHub Code Scanning も SAST に属しているセキュリティテスト手法です。  
<span><a href="https://blog.motikan2010.com/entry/2020/10/11/%E3%80%90%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E3%80%91GitHub%E3%81%AECode_Scanning%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B" target="_blank">GitHubのCode Scanningを使ってみる</a></span>・<span><a href="https://blog.motikan2010.com/entry/GitHub%E3%81%AECode_Scanning%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B_%E3%83%91%E3%83%BC%E3%83%882" target="_blank">GitHubのCode Scanningを使ってみる パート2</a></span>

　そして GitHub Code Scanning は CodeQLの技術が利用されています。  

[https://github.blog/jp/2020-10-06-code-scanning-is-now-available/:embed:cite]

<figure class="figure-image figure-image-fotolife" title="Code Scanningの実行画面。CodeQLが利用されていることが確認できます。">[f:id:motikan2010:20210208004307p:plain]<figcaption>Code Scanningの実行画面。CodeQLが利用されていることが確認できます。</figcaption></figure>

　CodeQL を使うメリットとしてはローカルPCに導入して利用することができるため、<span style="color: #ff0000">GitHubなどで管理していないソースコードに対してもセキュリティテストを実施することができます</span>。  

 　本記事では CodeQL をローカルPC上に導入し、脆弱性を含んだサンプルアプリケーションに対してセキュリティテストを行うまでを紹介します。  

<span style="color: #ff0000">※本記事の情報は2021年2月時点のものです。</span>

### サポート言語

　本記事では Javaで作成したアプリケーションを対象にセキュリティテストを実施しますが、CodeQL でサポートされている言語は以下の通りです。

- C/C++
- C#
- Golang
- Java
- JavaScript
- Python
- TypeScript

　バージョンによってはサポート外のものもありますので詳細は<span><a href="https://codeql.github.com/docs/codeql-overview/supported-languages-and-frameworks/" target="_blank">公式ドキュメント(Supported languages and frameworks — CodeQL)</a></span>をご覧ください。  

## 準備

検証環境

- MacOS 10.15.7
- Java (AdoptOpenJDK) 11

　本記事では以下のツールを導入して検証していきます。  

| ツール・アプリケーション | 簡単な説明 |
| - | - |
| <span><a href="https://github.com/github/codeql-cli-binaries" target="_blank">CodeQL CLI</a></span> | CodeQL のコマンドラインツールです。<br/>バージョン: 2.3.4 |
| <span><a href="https://github.com/github/codeql" target="_blank">CodeQL query</a></span> | セキュリティテストに必要なクエリが保存されています。<br/>バージョン: 1.26.0 |
| <span><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/tree/feature/xml-external-entity" target="_blank">検査対象アプリケーション</a></span> | XXE の脆弱性を含んでいる Spring Boot 製のアプリケーションです。<br/>motikan2010/GitHub-code-scanning-Test |

　私の環境では CodeQL CLI <b>2.4.3</b> を使った場合に "CodeQLデータベースの生成" が上手くいかなかったので、少し古いバージョンを利用しています。

### 検証環境の構築

　検証に使うツールは全て GitHub にありますので、それらをダウンロードしていきます。

<div class="md-code" style="width:100%">

```
■ 作業用ディレクトリを作成
$ mkdir codeql_work
$ cd codeql_work/

■ 検査対象アプリケーションのダウンロード
$ git clone -b feature/xml-external-entity git@github.com:motikan2010/GitHub-code-scanning-Test.git

■ CodeQL CLI のダウンロード (v2.3.4 リリース日：2020/12/17)
$ wget https://github.com/github/codeql-cli-binaries/releases/download/v2.3.4/codeql-osx64.zip
$ unzip codeql-osx64.zip
$ rm -f codeql-osx64.zip

■ CodeQL のダウンロード (v1.26.0 リリース日：2020/12/17)
$ git clone -b v1.26.0 https://github.com/github/codeql.git ql

■ 作業用ディレクトリの状態
~/Desktop/codeql_work
$ ll
total 0
drwxr-xr-x  12 motikan2010  staff  384  2  7 14:58 GitHub-code-scanning-Test
drwxr-xr-x  18 motikan2010  staff  576 12 16 01:15 codeql
drwxr-xr-x  25 motikan2010  staff  800  2  7 15:01 ql
```

</div>

### CodeQL CLI の動作確認 (バージョンの表示)

　CodeQL CLI が正常に動作するかバージョンを表示するコマンドを実行します。  

<div class="md-code" style="width:100%">

```
$ cd codeql
$ ./codeql version --format=json
```
</div>

<div class="md-code" style="width:100%">
```json
{
  "productName" : "CodeQL",
  "vendor" : "GitHub",
  "version" : "2.3.4",
  "sha" : "ff8bc627f2f24adc6ea19c838237d80a8f24a41f",
  "branches" : [
    "codeql-cli-2.3.4"
  ],
  "copyright" : "Copyright (C) 2019-2020 GitHub, Inc.",
  "unpackedLocation" : "/Users/motikan2010/Desktop/codeql_work/codeql"
}
```

</div>

## 動作確認

　スキャンは3ステップで行えます。  

1. スキャン対象のソースコードを元に <b>CodeQLデータベースの生成</b>
2. 「1.」 のデータベースに対して <b>CodeQLクエリの実行</b>
3. スキャン結果の表示

### CodeQLデータベースの生成

　CodeQLデータベースの生成は `codeql database create` コマンドで実行できます。

<span><a href="https://codeql.github.com/docs/codeql-cli/creating-codeql-databases/" target="_blank">Creating CodeQL databases — CodeQL</a></span>

<div class="md-code" style="width:100%">
```
$ ./codeql database create -l=java -s ../GitHub-code-scanning-Test codeql_db
```
</div>

　コマンド実行時の出力です。テスト対象のアプリケーションをビルドしているようです。  
<div class="sm-code" style="width:100%">
```
Initializing database at /Users/motikan2010/Desktop/codeql_work/codeql/codeql_db.
Running command [/Users/motikan2010/Desktop/codeql_work/codeql/java/tools/autobuild.sh] in /Users/motikan2010/Desktop/codeql_work/GitHub-code-scanning-Test.
[2021-02-07 15:19:30] [build] [2021-02-07 15:19:30] Build directory is /Users/motikan2010/Desktop/codeql_work/GitHub-code-scanning-Test/.
[2021-02-07 15:19:30] [build] [2021-02-07 15:19:30] [autobuild] > chmod +x gradlew
[2021-02-07 15:19:30] [build] [2021-02-07 15:19:30] [autobuild] > ./gradlew -Dorg.gradle.caching=false --no-daemon -S clean
[2021-02-07 15:19:34] [build] [2021-02-07 15:19:34] [autobuild] To honour the JVM settings for this build a new JVM will be forked. Please consider using the daemon: https://docs.gradle.org/6.6.1/userguide/gradle_daemon.html.
[2021-02-07 15:19:36] [build] [2021-02-07 15:19:36] [autobuild] Daemon will be stopped at the end of the build stopping after processing
[2021-02-07 15:19:41] [build] [2021-02-07 15:19:41] [autobuild] > Task :clean UP-TO-DATE
[2021-02-07 15:19:41] [build] [2021-02-07 15:19:41] [autobuild] BUILD SUCCESSFUL in 8s
[2021-02-07 15:19:41] [build] [2021-02-07 15:19:41] [autobuild] 1 actionable task: 1 up-to-date
[2021-02-07 15:19:41] [build] [2021-02-07 15:19:41] [autobuild] > ./gradlew -Dorg.gradle.caching=false --no-daemon -S testClasses
[2021-02-07 15:19:43] [build] [2021-02-07 15:19:43] [autobuild] To honour the JVM settings for this build a new JVM will be forked. Please consider using the daemon: https://docs.gradle.org/6.6.1/userguide/gradle_daemon.html.
[2021-02-07 15:19:45] [build] [2021-02-07 15:19:45] [autobuild] Daemon will be stopped at the end of the build stopping after processing
[2021-02-07 15:20:02] [build] [2021-02-07 15:20:02] [autobuild] > Task :compileJava
[2021-02-07 15:20:02] [build] [2021-02-07 15:20:02] [autobuild] > Task :processResources
[2021-02-07 15:20:02] [build] [2021-02-07 15:20:02] [autobuild] > Task :classes
[2021-02-07 15:20:06] [build] [2021-02-07 15:20:06] [autobuild] > Task :compileTestJava
[2021-02-07 15:20:06] [build] [2021-02-07 15:20:06] [autobuild] > Task :processTestResources NO-SOURCE
[2021-02-07 15:20:06] [build] [2021-02-07 15:20:06] [autobuild] > Task :testClasses
[2021-02-07 15:20:07] [build] [2021-02-07 15:20:07] [autobuild] BUILD SUCCESSFUL in 24s
[2021-02-07 15:20:07] [build] [2021-02-07 15:20:07] [autobuild] 3 actionable tasks: 3 executed
Finalizing database at /Users/motikan2010/Desktop/codeql_work/codeql/codeql_db.
[2021-02-07 15:20:09] [build-err] Scanning for files in /Users/motikan2010/Desktop/codeql_work/GitHub-code-scanning-Test...
Successfully created database at /Users/motikan2010/Desktop/codeql_work/codeql/codeql_db.
```
</div>

　コマンド実行後、"codeql_db"というディレクトリが作成されます。
CodeQLデータベースの作成が正常に行われた場合のディレクトリの中身は以下の通りです。    
<div class="md-code" style="width:100%">
```
$ ls -l codeql_db
total 16
-rw-r--r--   1 motikan2010  staff   136  2  7 15:20 codeql-database.yml
drwxr-xr-x   5 motikan2010  staff   160  2  7 15:20 db-java
drwxr-xr-x  15 motikan2010  staff   480  2  7 15:24 log
drwxr-xr-x   3 motikan2010  staff    96  2  7 15:23 results
-rw-------   1 motikan2010  staff  2172  2  7 15:20 src.zip
```
</div>

### 脆弱性の検出

　スキャンは `codeql database analyze` コマンドで実行できます。(<span><a href="https://codeql.github.com/docs/codeql-cli/manual/database-analyze/" target="_blank">database analyze — CLI manual</a></span>)  

　検出対象の脆弱性は以下の通りです。  

<span><a href="https://github.com/github/codeql/tree/main/java/ql/src/Security/CWE" target="_blank">codeql/java/ql/src/Security/CWE at main · github/codeql</a></span>

| CWE 識別子 | 概要 (JVN iPedia) |
| - | - | - |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-020" target="_blank">CWE-20</a></span> | 不適切な入力確認 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-022" target="_blank">CWE-22</a></span> | パス・トラバーサル |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-078" target="_blank">CWE-78</a></span> | OSコマンドインジェクション |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-079" target="_blank">CWE-79</a></span> | クロスサイトスクリプティング |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-089" target="_blank">CWE-89</a></span> | SQLインジェクション |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-090" target="_blank">CWE-90</a></span> | LDAP インジェクション |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-094" target="_blank">CWE-94</a></span> | コード・インジェクション |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-113" target="_blank">CWE-113</a></span> | HTTP レスポンスの分割 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-129" target="_blank">CWE-129</a></span> | 配列インデックスの不適切な検証 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-134" target="_blank">CWE-134</a></span> | 書式文字列の問題 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-190" target="_blank">CWE-190</a></span> | 整数オーバーフローまたはラップアラウンド |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-209" target="_blank">CWE-209</a></span> | エラーメッセージによる情報漏えい |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-312" target="_blank">CWE-312</a></span> | 重要な情報の平文保存 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-319" target="_blank">CWE-319</a></span> | 重要な情報の平文での送信 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-327" target="_blank">CWE-327</a></span> | 不完全、または危険な暗号アルゴリズムの使用 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-335" target="_blank">CWE-335</a></span> | PRNG におけるシードの不正な使用 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-338" target="_blank">CWE-338</a></span> | 暗号における脆弱な PRNG の使用 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-352" target="_blank">CWE-352</a></span> | クロスサイトリクエストフォージェリ |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-367" target="_blank">CWE-367</a></span> | Time-of-check Time-of-use (TOCTOU) 競合状態 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-421" target="_blank">CWE-421</a></span> | Race Condition During Access to Alternate Channel (by MITRE) |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-502" target="_blank">CWE-502</a></span> | 信頼できないデータのデシリアライゼーション |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-601" target="_blank">CWE-601</a></span> | オープンリダイレクト |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-611" target="_blank">CWE-611</a></span> | XML 外部エンティティ参照の不適切な制限 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-614" target="_blank">CWE-614</a></span> | Sensitive Cookie in HTTPS Session Without 'Secure' Attribute (by MITRE) |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-676" target="_blank">CWE-676</a></span> | Use of Potentially Dangerous Function (by MITRE) |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-681" target="_blank">CWE-681</a></span> | 数値型間の変換の誤り |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-732" target="_blank">CWE-732</a></span> | 重要なリソースに対する不適切なパーミッションの割り当て |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-798" target="_blank">CWE-798</a></span>| ハードコードされた認証情報の使用 |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-807" target="_blank">CWE-807</a></span> | Reliance on Untrusted Inputs in a Security Decision (by MITRE) |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-829" target="_blank">CWE-829</a></span> | 信頼性のない制御領域からの機能の組み込み |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-833" target="_blank">CWE-833</a></span> | Deadlock (by MITRE) |
| <span><a href="https://github.com/github/codeql/tree/v1.26.0/java/ql/src/Security/CWE/CWE-835" target="_blank">CWE-835</a></span> | 無限ループ |

#### XXE の検査 ~ 検出されることの確認 ~

　全ての脆弱性を検査対象にすると時間が掛かってしまうため、今回は「XXE攻撃(XML eXternal Entity attack)」のみを対象に検査します。  
（全てを検査対象にした場合、10分ほど時間が掛かりました。）

- [CWE - CWE-611: Improper Restriction of XML External Entity Reference (4.3)](https://cwe.mitre.org/data/definitions/611.html)

##### 検査の実施 ~ CSV形式で出力 ~

　テスト結果はファイルに出力されるようになっています。ファイル形式は `--format` で指定します。  

CSV形式で出力した例です。

<div class="md-code" style="width:100%">
```
■ 検査の実行
$ ./codeql database analyze "codeql_db" \
../ql/java/ql/src/Security/CWE/CWE-611/ \
--format csv --output xxe_result.csv

■ 検査結果の表示
$ cat xxe_result.csv
"Resolving XML external entity in user-controlled data","Parsing user-controlled XML documents and allowing expansion of external entity references may lead to disclosure of confidential data or denial of service.","error","Unsafe parsing of XML file from [[""user input""|""relative:///src/main/java/com/motikan2010/github_code_scanning_test/controller/XxeController.java:26:52:26:92""]].","/src/main/java/com/motikan2010/github_code_scanning_test/controller/XxeController.java","29","41","29","79"
```
</div>

　1行のCSVファイルとして結果が出力されました。  
少々見づらいですが<span style="color: #ff0000">結果内容に脆弱性が見つかったファイルや行数の記載がされている</span>ことが確認できます。  

##### 検査の実施 ~ SARIFで出力 ~

　次は SARIFで出力してみます。  

SARIFという単語は馴染みありませんが「Static Analysis Results Interchange Format」の略称であり、

> SARIF 標準は、静的分析ツールが結果を共有する方法を合理化するために使用されます。

とのことです。  
（当初私は SARIF"形式"と書いていましたが、「Format」の"F"なので明らかな誤りですね...）  

　SARIF の見方は下のドキュメントが参考になります。  
[https://docs.github.com/ja/github/finding-security-vulnerabilities-and-errors-in-your-code/sarif-support-for-code-scanning:embed:cite]

<div class="md-code" style="width:100%">
```
■ 検査の実行
$ ./codeql database analyze "codeql_db" \
../ql/java/ql/src/Security/CWE/CWE-611/ \
--format=sarif-latest --sarif-multicause-markdown --output=xxe_result.sarif --no-sarif-add-snippets
```
</div>

　中身はJSON形式のようです。  
<div class="md-code" style="width:100%">
```
$ cat xxe_result.sarif | jq .
```
</div>

<div onclick="obj=document.getElementById('20210207_folding_text').style; obj.display=(obj.display=='none')?'block':'none';">
<a style="cursor:pointer;">▼ xxe_result.sarif の中身(130行) | クリックで展開</a>
</div>
<div id="20210207_folding_text" style="display:none;clear:both;">

<div class="sm-code" style="width:100%">
```json
{
  "$schema": "https://raw.githubusercontent.com/oasis-tcs/sarif-spec/master/Schemata/sarif-schema-2.1.0.json",
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "CodeQL",
          "organization": "GitHub",
          "semanticVersion": "2.3.4",
          "rules": [
            {
              "id": "java/xxe",
              "name": "java/xxe",
              "shortDescription": {
                "text": "Resolving XML external entity in user-controlled data"
              },
              "fullDescription": {
                "text": "Parsing user-controlled XML documents and allowing expansion of external entity references may lead to disclosure of confidential data or denial of service."
              },
              "defaultConfiguration": {
                "level": "error"
              },
              "properties": {
                "tags": [
                  "security",
                  "external/cwe/cwe-611",
                  "external/cwe/cwe-776",
                  "external/cwe/cwe-827"
                ],
                "kind": "path-problem",
                "precision": "high",
                "name": "Resolving XML external entity in user-controlled data",
                "description": "Parsing user-controlled XML documents and allowing expansion of external entity\n references may lead to disclosure of confidential data or denial of service.",
                "id": "java/xxe",
                "problem.severity": "error"
              }
            }
          ]
        }
      },
      "artifacts": [
        {
          "location": {
            "uri": "src/main/java/com/motikan2010/github_code_scanning_test/controller/XxeController.java",
            "uriBaseId": "%SRCROOT%",
            "index": 0
          }
        }
      ],
      "results": [
        {
          "ruleId": "java/xxe",
          "ruleIndex": 0,
          "message": {
            "text": "Unsafe parsing of XML file from [user input](1)."
          },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "src/main/java/com/motikan2010/github_code_scanning_test/controller/XxeController.java",
                  "uriBaseId": "%SRCROOT%",
                  "index": 0
                },
                "region": {
                  "startLine": 29,
                  "startColumn": 41,
                  "endColumn": 80
                }
              }
            }
          ],
          "partialFingerprints": {
            "primaryLocationLineHash": "2496454f6db6d4d1:1",
            "primaryLocationStartColumnFingerprint": "32"
          },
          "codeFlows": [
            {
              "threadFlows": [
                {
                  "locations": [
                    {
                      "location": {
                        "physicalLocation": {
                          "artifactLocation": {
                            "uri": "src/main/java/com/motikan2010/github_code_scanning_test/controller/XxeController.java",
                            "uriBaseId": "%SRCROOT%",
                            "index": 0
                          },
                          "region": {
                            "startLine": 26,
                            "startColumn": 52,
                            "endColumn": 93
                          }
                        },
                        "message": {
                          "text": "data : String"
                        }
                      }
                    },
                    {
                      "location": {
                        "physicalLocation": {
                          "artifactLocation": {
                            "uri": "src/main/java/com/motikan2010/github_code_scanning_test/controller/XxeController.java",
                            "uriBaseId": "%SRCROOT%",
                            "index": 0
                          },
                          "region": {
                            "startLine": 29,
                            "startColumn": 41,
                            "endColumn": 80
                          }
                        },
                        "message": {
                          "text": "new InputSource(...)"
                        }
                      }
                    }
                  ]
                }
              ]
            }
          ],
          "relatedLocations": [
            {
              "id": 1,
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "src/main/java/com/motikan2010/github_code_scanning_test/controller/XxeController.java",
                  "uriBaseId": "%SRCROOT%",
                  "index": 0
                },
                "region": {
                  "startLine": 26,
                  "startColumn": 52,
                  "endColumn": 93
                }
              },
              "message": {
                "text": "user input"
              }
            }
          ]
        }
      ],
      "columnKind": "utf16CodeUnits",
      "properties": {
        "semmle.formatSpecifier": "sarif-latest"
      }
    }
  ]
}
```
</div>
</div>

　結果を見てみると以下のような記載があります。  
<div class="md-code" style="width:100%">
```json
"locations": [
  {
    "physicalLocation": {
      "artifactLocation": {
        "uri": "src/main/java/com/motikan2010/github_code_scanning_test/controller/XxeController.java",
        "uriBaseId": "%SRCROOT%",
        "index": 0
      },
      "region": {
        "startLine": 29,
        "startColumn": 41,
        "endColumn": 80
      }
    }
  }
],
```
</div>

これはソースコード上で脆弱性を存在している部分を示しています。  

　指摘されたソースコードは下の画像です。
<figure class="figure-image figure-image-fotolife" title="指摘されたソースコード">[f:id:motikan2010:20210208014644p:plain]<figcaption>指摘されたソースコード</figcaption></figure>

　確かに脆弱性となっている部分です。

#### RCE の検査 ~ 検出されないことの確認 ~

　次は RCE を検査し、脆弱性が検出されないことを確認します。  

- [CWE-78: Improper Neutralization of Special Elements used in an OS Command](https://cwe.mitre.org/data/definitions/78.html)

<div class="md-code" style="width:100%">
```
■ 検査の実行
$ ./codeql database analyze "codeql_db" \
../ql/java/ql/src/Security/CWE/CWE-078/ \
--format csv --output rce_result.csv

■ 検査結果の表示
$ cat rce_result.csv
(出力なし)
```
</div>

　脆弱性が検出されなかったので、検査結果が格納されるファイルは空となりました。  
（ツール使用当初は検査が失敗しているのではと疑っていました・・・）

## まとめ

　CodeQL。簡単。無料。 楽しい。

　お金ないけどプライベートリポジトリに対して SAST したいよという方にはオススメです。

## 参考

- <span><a href="https://codeql.github.com/docs/codeql-overview/about-codeql/" target="_blank">About CodeQL — CodeQL</a></span>
- <span><a href="https://jp.techcrunch.com/2019/09/19/2019-09-18-github-acquires-code-analysis-tool-semmle/" target="_blank">GitHubがセキュリティのためのコード分析ツールSemmleを買収 | TechCrunch Japan</a></span>
- <span><a href="https://www.ipa.go.jp/security/vuln/CWE.html" target="_blank">共通脆弱性タイプ一覧CWE概説：IPA 独立行政法人 情報処理推進機構</a></span>

## 更新履歴

- 2021年2月7日 新規作成
- 2021年2月8日 CWE記載など
