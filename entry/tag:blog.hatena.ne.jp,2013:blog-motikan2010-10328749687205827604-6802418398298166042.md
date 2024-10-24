<div style="text-align:center;">[f:id:motikan2010:20241023223247p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## GitHub Copilot Autofix とは

　「GitHub Copilot Autofix」は自動的にソースコードから脆弱性を検出して、**脆弱性を修正するためのソースコードを提案**してくれるサービスです。  

　2024年8月14日にリリースされました。  
 
[https://github.blog/jp/2024-08-15-secure-code-more-than-three-times-faster-with-copilot-autofix/:embed:cite]

## Copilot Autofix が利用できるリポジトリ

　Copilot Autofix の利用は GitHub Copilot のサブスクリプションは**不要**です。  

以下２種類のリポジトリで使用ができます。  

- GitHub.com のすべての**パブリックリポジトリ**
- GitHub Advanced Security のライセンスを持つ GitHub Enterprise Cloud Enterprise の**プライベートリポジトリ**

つまり無償で利用するためにはパブリックリポジトリである必要があります。今回はパブリックリポジトリを利用して検証を進めます。  

<span><a href="https://docs.github.com/ja/code-security/code-scanning/managing-code-scanning-alerts/responsible-use-autofix-code-scanning" target="_blank">コード スキャンに対するコパイロットの自動修正の責任ある使用 - GitHub Docs</a></span>

## Copilot Autofix を有効化

Copilot Autofix を動作させるには Code scanning を有効化する必要があります。  
GithHub リポジトリの「`Settings` > `Code security` > `Code scanning`」から有効化することができます。  

[f:id:motikan2010:20241022235417p:plain:w700]  

## Copilot Autofix の検証

### 検証内容

　意図的に脆弱性を埋め込んだJavaのソースコードを Copilot Autofix で検査し、修正ソースコードの有無や修正ソースコードの内容を確認していきます。  

　前提として Copilot Autofix は Github が提供している CodeQL（静的解析）で検出した脆弱性の修正提案を出力するようになっています。そのため CodeQL で脆弱性を検出できた以下６つの脆弱性に対して Copilot Autofix がどのような検査結果を出力するのかを調べていきます。    

- Insecure Deserialization (安全ではないデシリアライゼーション)
- SQL インジェクション
- XSS (クロスサイトスクリプティング)
- ディレクトリ・トラバーサル
- XXE 攻撃 (XML eXternal Entity attack)
- RCE (Remote Code Execution)

### 結果

　各脆弱性の検証結果を以下の表に記載します。  

| 脆弱性 | Autofix による修正ソースコード提案 |
| - | - |
| Insecure Deserialization <br>(安全ではないデシリアライゼーション) | なし（脆弱性がサポートされていない） |
| SQL インジェクション | なし（提案できるソースコードを生成できない） |
| XSS (クロスサイトスクリプティング) | なし（Copilot Autofix が起動していない） |
| ディレクトリ・トラバーサル | **あり** |
| XXE 攻撃 (XML eXternal Entity attack) | **あり** |
| RCE (Remote Code Execution) | **あり** |

　見て分かる通り Copilot Autofix が修正ソースコードを提案しない場合もあります。

　そもそも Copilot Autofix に対応していない脆弱性が存在しており、その一覧は以下のサイトから確認できるようになっていました。  
<span><a href="https://docs.github.com/ja/code-security/code-scanning/managing-your-code-scanning-configuration/java-kotlin-built-in-queries" target="_blank">CodeQL 分析のための Java クエリと Kotlin クエリ - GitHub Docs</a></span>
[f:id:motikan2010:20241023005719p:plain:w600]  

#### Insecure Deserialization (安全ではないデシリアライゼーション)

　Copilot Autofix による修正ソースコードの出力はありませんでした。  

　CodeQL の結果に「Rule java/unsafe-deserialization is not supported by autofix （ルール java/unsafe-deserialization は **autofix ではサポートされていません**）」とあることから、Copilot Autofix が実行されることはありませんでした。  

[f:id:motikan2010:20241022234516p:plain]  
<span style="font-size: 80%"><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/2" target="_blank">https://github.com/motikan2010/GitHub-code-scanning-Test/pull/2</a></span>

　こちらの脆弱性は先述した「Copilot Autofix に対応していない脆弱性」のようです。  

<span style="font-size: 140%">**➡ 「 前提として Copilot Autofix に対応していない脆弱性がある 」**</span>

---

#### SQL インジェクション

Copilot Autofix による修正ソースコードの出力はありませんでした。  

　CodeQL の結果に「Copilot Autofix cannot generate an appropriate fix for this alert at this time （Copilot Autofix は現時点では**このアラートに対する適切な修正プログラムを生成できません**）」とあります。  

　Copilot Autofix は実行されたようですが、修正ソースコードの作成まではできなかったようです。

[f:id:motikan2010:20241022234513p:plain]  
<span style="font-size: 80%"><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/4" target="_blank">https://github.com/motikan2010/GitHub-code-scanning-Test/pull/4</a></span>  

<span style="font-size: 140%">**➡ 「 脆弱性の埋め込まれ方によっては修正ソースコードを生成することができない場合がある 」**</span>

---

#### XSS (クロスサイトスクリプティング)

　Copilot Autofix による修正ソースコードの出力はありませんでした。  

[f:id:motikan2010:20241022234510p:plain]  
<span style="font-size: 80%"><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/5" target="_blank">https://github.com/motikan2010/GitHub-code-scanning-Test/pull/5</a></span>

　ドキュメント上では Copilot Autofix に対応している脆弱性ではありますが、実行はされていないようで、実行されない理由についての出力もありませんでした。  （severity に "Error" と出てますがこれが関係している?）  
[f:id:motikan2010:20241023180353p:plain:w600]

<span style="font-size: 140%">**➡ 「 原因不明で Copilot Autofix が実行されない場合がある 」**</span><span style="font-size: 90%">？</span>

---

#### ディレクトリ・トラバーサル

　Copilot Autofix による修正ソースコードの提案がありました。  

　修正内容について細かくは紹介しませんが「パスの指定に用いられる特殊文字の禁止」や「開くファイルのディレクトリの固定化」など実際に利用できる修正ソースコードを提案されています。  

　提案されたソースコードの差分の説明は生成AIによって作成されているようです。そのため修正内容の説明は実際のソースコードを基に作成されているためか、固定の修正文言を出力するツールに比べて非常に分かりやすい印象がありました。  

Copilot Autofix による修正内容の提案：
> <span style="font-size: 80%">この問題を解決するには、`fileName`パラメータを検証して、パストラバーサルシーケンスや無効な文字が含まれていないことを確認する必要があります。</span>  
<span style="font-size: 80%">・`fileName` に「..」、「/」、「\」が含まれていないかチェックするバリデーションを追加しました。</span>  
<span style="font-size: 80%">・解決されたパスが `src/main/resources/static/` ディレクトリ内にあることを確認します。</span>  

[f:id:motikan2010:20241022234457p:plain]  
<span><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/8" target="_blank">https://github.com/motikan2010/GitHub-code-scanning-Test/pull/8</a></span>

<span style="font-size: 140%">**➡ 「 修正ソースコードの提案が分かりやすい 」**</span><span style="font-size: 90%">　・・・例外はあるかも。</span> 

---

#### XXE 攻撃 (XML eXternal Entity attack)

Copilot Autofix による修正ソースコードの提案がありました。XXE 攻撃の対応策は XML パーサー設定を変更するというのが有効であり、その設定を行うような修正ソースコードが生成されていました。  
（この辺りは今までの SAST でも十分にやってくれる印象）

[f:id:motikan2010:20241022234501p:plain]  
<span style="font-size: 80%"><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/7" target="_blank">https://github.com/motikan2010/GitHub-code-scanning-Test/pull/7</a></span>

<span style="font-size: 140%">➡ **「 そのまま反映してよい修正ソースコードを提案する場合がある 」**</span>

---

#### RCE (Remote Code Execution)

Copilot Autofix による修正ソースコードの提案がありました。今回の中では一番興味深い Copilot Autofix の動作をしていました。  

　まず、今回検証した脆弱性があるソースコードは変数名が「`ip`」という変数がOSコマンドとしてそのまま渡しているという点が脆弱となっています。  

　Copilot Autofix が提案した修正ソースコードは変数`ip`がIPアドレスのフォーマットに合致しているかを確認して、そうでない場合にエラーを発生させるようにして RCE の対策を行なっています。  

　つまり Copilot Autofix は<span style="color: #ff5252">**変数名から変数の目的を推測**</span>して修正ソースコードを提案するようになっており、汎用的な修正方法ではなく検査対象のソースコードに合わせた修正内容を提案しています。  
　この挙動は生成AIを活用しているから実現できているものだと思います。  

[f:id:motikan2010:20241022234505p:plain]  
<span style="font-size: 80%"><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/6" target="_blank">https://github.com/motikan2010/GitHub-code-scanning-Test/pull/6</a></span>



　本当に「変数名から変数の目的を推測」しているのかを確認するために、処理は同じだが変数名を `ip` から「`iq`」に変えて検査を実行してみます。  
　脆弱性の検出後、Copilot Autofix は**全く異なる修正コードを提案しました。**   
[f:id:motikan2010:20241022234453p:plain]  
<span style="font-size: 80%"><a href="https://github.com/motikan2010/GitHub-code-scanning-Test/pull/21" target="_blank">https://github.com/motikan2010/GitHub-code-scanning-Test/pull/21</a></span>

　**Copilot Autofix は変数名も考慮して脆弱性の修正ソースコードを提案しているようです。**  
このような仕組みを用いた脆弱性の提案方法が普及するとセキュリティ向上のためのネーミングルールも普及していきそうです。 

<span style="font-size: 140%">➡ **「 Copilot Autofix は変数名などのシステム特有の部分も考慮して修正ソースコードを生成している 」**</span>

## まとめ

　Copilot Autofix を検証して分かったことは以下の通りです。

- Copilot Autofix に対応していない脆弱性がある（対応有無はドキュメントに記載されている）
- 脆弱性の埋め込まれ方によっては修正ソースコードを生成することができない場合がある
- 原因は不明だが Copilot Autofix が実行されない場合がある
- 提案された修正ソースコードに生成AIによって作成された説明が記載されており分かりやすい
- すぐに反映できる修正ソースコードが提案される場合もある
- Copilot Autofix は変数名などのシステム特有の部分も考慮して修正ソースコードを生成している

　脆弱性の検出ロジックに関しては既にあった CodeQL に依存しているようで、Copilot Autofix は検出結果を基に修正ソースコードや修正内容の説明を生成するシステムでした。  

　そのため脆弱性の検出能力が生成AI活用前と比べて格段に上がったというわけではありませんので、従来のDASTといった脆弱性診断の代わりにはならなさそうです。  

　ですが、ソースコードのコンテクストも考慮しての脆弱性修正のための提案内容が出力される仕組みは、ソースコードを書いている本人には脆弱性の内容や修正方法がとても分かりやすくSASTとの相性は良い印象です。  

　Copilot Autofix を利用するためのハードルは金銭面で高いと思われますが、Copilot Autofix 以外でも生成AIを用いて SAST の結果を分かりやすい形式で出力する仕組みは普及していきそうです。  

[blog:g:12921228815726579926:banner]　[blog:g:11696248318754550880:banner]
