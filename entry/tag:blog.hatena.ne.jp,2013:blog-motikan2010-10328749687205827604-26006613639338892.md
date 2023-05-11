<div style="text-align:center;">[f:id:motikan2010:20201011211756p:plain]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　先日GitHubから Code Scanning が正式リリースされました。  
[https://github.blog/jp/2020-10-06-code-scanning-is-now-available/:embed:cite]

　一言で表すと「<span class="m-y">プログラムを検査し、脆弱性を検出する</span>」ツールです。  

　今までもこのような静的解析ツールは存在していましたが、<span class="m-y">GitHubパブリックリポジトリに対してのスキャンは「「「無料」」」</span>なので早速使ってみました。  

　対応している言語は `C`、`C++`、`C#`、`Java`、`JavaScript`、`TypeScript`、`Python`、`Go`  となっています。  

### 本記事でやること

- 意図的に脆弱性を埋め込んだWebアプリケーションのスキャン
- スキャン結果の確認

▼ 本記事で利用したリポジトリです。プルリクに検査結果があります。  
[https://github.com/motikan2010/GitHub-code-scanning-Test/pulls:embed:cite]

　ちなみにリポジトリメンバー以外は検出された脆弱性の詳細情報を確認することができないようです。  

- メンバー からの見え方  
[f:id:motikan2010:20201011214016p:plain:w500]  
- メンバー以外 からの見え方  
[f:id:motikan2010:20201011214012p:plain:w500]  

### 環境

　本記事で Java で作成したWebアプリケーションをスキャンしていきます。  
Webフレームワークには Spring Boot を利用しています。  

| | |
| - | - |
| Java | AdoptOpenJDK 11.0 |
| Spring Boot | 2.3.4 |

## スキャンの実行準備

　Code Scanning を実行には、GitHub Actions を利用することができます。  

そのため<span class="m-y">Code Scanningのワークフロー用ファイルを作成するだけでスキャンを実行することが可能</span>です。

### ワークフロー用のYAMLファイルの作成

　スキャンを実行するためには `.github/workflows/codeql-analysis.yml` ファイルが必要です。  
このファイルの作成手順は以下の通りです。  

①. パブリックリポジトリ内で、`Security`タブ > `Set up code scanning`ボタン  
[f:id:motikan2010:20201011151525p:plain:w600]  

　ちなみにプライベートリポジトリでは `Set up code scanning` の表示されませんでした。  
[f:id:motikan2010:20201011151535p:plain:w600]  

　（当然ですが）Code Scanning 設定後にプライベートリポジトリに変更してもスキャンできません。  
[f:id:motikan2010:20201011210108p:plain:w600]  

②. `Set up this workflow`ボタン  
[f:id:motikan2010:20201011151540p:plain:w600]  

③. `Start commit`ボタン押下後、`.github/workflows/codeql-analysis.yml` ファイルが作成されます。  

[f:id:motikan2010:20201011151551p:plain:w600]  

<details><summary>`codeql-analysis.yml`ファイルの内容</summary><div>

<div class="sm-code" style="width:100%">
```yaml
# For most projects, this workflow file will not need changing; you simply need
# to commit it to your repository.
#
# You may wish to alter this file to override the set of languages analyzed,
# or to provide custom queries or build logic.
name: "CodeQL"

on:
  push:
    branches: [master]
  pull_request:
    # The branches below must be a subset of the branches above
    branches: [master]
  schedule:
    - cron: '0 4 * * 0'

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false
      matrix:
        # Override automatic language detection by changing the below list
        # Supported options are ['csharp', 'cpp', 'go', 'java', 'javascript', 'python']
        language: ['java']
        # Learn more...
        # https://docs.github.com/en/github/finding-security-vulnerabilities-and-errors-in-your-code/configuring-code-scanning#overriding-automatic-language-detection

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2
      with:
        # We must fetch at least the immediate parents so that if this is
        # a pull request then we can checkout the head.
        fetch-depth: 2

    # If this run was triggered by a pull request event, then checkout
    # the head of the pull request instead of the merge commit.
    - run: git checkout HEAD^2
      if: ${{ github.event_name == 'pull_request' }}

    # Initializes the CodeQL tools for scanning.
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v1
      with:
        languages: ${{ matrix.language }}
        # If you wish to specify custom queries, you can do so here or in a config file.
        # By default, queries listed here will override any specified in a config file. 
        # Prefix the list here with "+" to use these queries and those in the config file.
        # queries: ./path/to/local/query, your-org/your-repo/queries@main

    # Autobuild attempts to build any compiled languages  (C/C++, C#, or Java).
    # If this step fails, then you should remove it and run the build manually (see below)
    - name: Autobuild
      uses: github/codeql-action/autobuild@v1

    # ℹ️ Command-line programs to run using the OS shell.
    # 📚 https://git.io/JvXDl

    # ✏️ If the Autobuild fails above, remove it and uncomment the following three lines
    #    and modify them (or add more) to build your code if your project
    #    uses a compiled language

    #- run: |
    #   make bootstrap
    #   make release

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v1

```
</div>
</details>

### エラー発生

　`codeql-analysis.yml`の作成時にスキャンが実行されましたが、下記のエラーが発生しました。  
<div class="sm-code" style="width:100%">
```
  [2020-10-10 18:13:27] [autobuild] Daemon will be stopped at the end of the build stopping after processing
  [2020-10-10 18:13:41] [autobuild] > Task :compileJava FAILED
  [2020-10-10 18:13:41] [autobuild] FAILURE: Build failed with an exception.
  [2020-10-10 18:13:41] [autobuild] * What went wrong:
  [2020-10-10 18:13:41] [autobuild] Execution failed for task ':compileJava'.
  [2020-10-10 18:13:41] [autobuild] > Could not target platform: 'Java SE 11' using tool chain: 'JDK 8 (1.8)'.
```
</div>

　`Could not target platform: 'Java SE 11' using tool chain: 'JDK 8 (1.8)'.`  
とメッセージが表示され、Java 11 に対応させる必要がありそうです。  

### Java 11 に対応

　エラーを解消するために、`codeql-analysis.yml`内の`steps`に以下を追加します。  
<div class="md-code" style="width:60%">
```yaml
    - uses: actions/setup-java@v1
      with:
        java-version: '11'
```
</div>

　これで問題なくスキャンが実行されるようになりました。  

## スキャン実行

　ここからは実際にアプリケーションに対してスキャンを実行してきます。  

　今回利用するアプリケーションは Code Scanning を試すために作成したもので意図的に脆弱性を埋め込んでいます。  
試した脆弱性は「SQL Injection」「Insecure Deserialization」「Cross-Site Scripting」の３種です。  

　そしてスキャンによって検知された脆弱性が、どのように表示されるのかを確認します。  

### SQL Injection

　スキャンを実行するために <span><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/4" target="_blank">SQL Injectionの脆弱性が含まれたコードのプルリクエスト</a></span> を作成しました。

　スキャンの結果に ❌ が表示されていることから、何か起きたことが分かります。`Details`を押下しスキャンの詳細を確認してみます。  
[f:id:motikan2010:20201011175049p:plain:w600]  

　スキャン結果 1件の脆弱性が検知された旨の記述があり、脆弱性の概要は `Query built from user-controlled sources` となっています。  

　これで意図的に埋め込んだSQL Injection が Code Scanning によって検知されたことが確認できました。  
[f:id:motikan2010:20201011184350p:plain:w600]  

　脆弱性の詳細を確認するためには `Show more details` を押下します。  

そうすると脆弱性が存在しているコード部分が強調された、脆弱性の詳細画面へ遷移します。  
[f:id:motikan2010:20201011175052p:plain:w600]  

　より詳細に脆弱性について知りたいのであれば、`Show more` を押下します。すると脆弱性の概要や修正方法、参考リンクなどの情報が展開されます。  

　下画像は SQL Injection の場合ですが、結構なボリュームがあります。  
[f:id:motikan2010:20201011174009p:plain:w400]

　さらに`Show paths`を押下すると、ユーザが入力した値（ここでは`name`値）がどのような流れでSQLに挿入されるのかを確認できます。（親切すぎる！）  
[f:id:motikan2010:20201011175057p:plain:w600]  

### Insecure Deserialization（安全でないデシリアライゼーション）

　次は <span><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/2" target="_blank">Insecure Deserialization の脆弱性を埋め込んでスキャン</a></span> してみます。  

　スキャン結果の詳細は以下のように表示されました。  
[f:id:motikan2010:20201011190503p:plain:w600]  

　Insecure Deserialization でも同様にユーザが入力した値がどのような流れで脆弱なコードにたどり着くのかを確認することができました。  
[f:id:motikan2010:20201011190459p:plain:w600]  

### Cross-Site Scripting（XSS）

　最後に <span><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/1" target="_blank">Cross-Site Scripting（XSS） の脆弱性を埋め込んでスキャン</a></span> してみます。  

　スキャン結果から先に言うと、Code Scanning では脆弱性が検知されませんでした。  

　脆弱性を含むプルリクエストは下画像です。上部がコントローラ側、下部がビュー側となっています。  
[f:id:motikan2010:20201011202004p:plain:w600]

　XSSの原因となっているコードは `<span th:utext="${msg}"/>` の部分で `th:utext` を使っている点です。
これはユーザから与えられた値はHTMLエスケープせずに表示されるようになっています。  

　ですが、下画像のスキャン結果通り XSS は検知されませんでした。  

[f:id:motikan2010:20201011202350p:plain:w600]

　原因は不明ですが、Javaコード外で発生している脆弱性だからなのではないかなと思っています。  
(テンプレートエンジンは検査対象外？)  
　それか 脆弱性の埋め込み力 が足りていない・・・。

## その他

### リポジトリにアクティビティがないと無効化される

　Code Scanning を有効化しているリポジトリは60日間更新しないと自動的に無効化されるようです。  

　無効化される前に下のようなメールが届きました。  

[f:id:motikan2010:20201207043557p:plain:w600]  

　メール記載のリンク先からすぐに更新することができました。  
[f:id:motikan2010:20201207043828p:plain:w600]

## まとめ

　今回初めて GitHub の Code Scanning を利用してみましたが、導入が簡単であり、検出された脆弱性についても説明や検出理由の記載もあり、有意義な機能だと感じました。  

　パブリックリポジトリでの利用は無料ですので、今後の開発では導入するのが一般的になっていくのではないかと期待していまが、どうしてもプライベートリポジトリでの利用は有償となっているので、その点が導入のハードルになるのではと思っています。  

　プライベートリポジトリで利用するためには、<span><a href="https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/githubs-products#github-one" target="_blank">「GitHub One」ライセンス</a></span>、または<span><a href="https://github.com/features/security#security-prevent" target="_blank">「Enterprise + Code scanning オプション」ライセンス</a></span> を利用する必要があるとのことです。  

　価格は提示されていませんが、十分な機能が提供されていることもありますのでいいお値段しそうです。  
[f:id:motikan2010:20201011203630p:plain:w600]  

　今も様々な言語がサポートしていますが PHP や Ruby もサポートされることに期待しています。  

## 更新履歴

- 2020年10月11日 新規作成
