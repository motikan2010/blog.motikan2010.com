<div style="text-align:center;">[f:id:motikan2010:20251217223201g:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　AWS から「AWS Security Agent」というセキュリティサービスが発表され、「**Code review**」という機能のSAST(静的アプリケーションセキュリティテスト)が提供されましたので、脆弱なソースコードをセキュリティ検査してみました。  (**※プレビュー版**)

▼ AWS Security Agent についてはコチラ  
[https://aws.amazon.com/jp/blogs/aws/new-aws-security-agent-secures-applications-proactively-from-design-to-deployment-preview/:embed:cite]

　Security Agent には主に下記３つの機能がありますが、本記事では Code review を検証してみます。 

- Design review （設計ドキュメント有りのSAST）
- **Code review** （SAST）
- Penetration testing （DAST）

<div style="text-align:center;">[f:id:motikan2010:20251217213248p:plain:w700]</div>

## 検証

　SASTを検証するためには脆弱性を含んだソースコードのプルリクエストが必要です。そのために大量の脆弱性を含んだ GitHub リポジトリで検証を実施しています。    
▼ リポジトリはコチラ
[https://github.com/motikan2010/Test-AWS_Security_Agent-Code_Review:title]  

### 巨大なプルリクエストのSAST

　ソースコード全体のSASTが可能であれば、簡易にソースコード診断に活用できると思ったので確認してみました。  

　"Code review"の動作として脆弱性が検出された場合にはソースコードの該当行に対して"コメントが記載"されるようになっているようです。
<div style="text-align:center;">[f:id:motikan2010:20251216230234p:plain:w600]</div>

　脆弱性学習アプリであるDVWAのソースコード全体をプルリクエストにあげているため、巨大なプルリクエストとなっています。  
<div style="text-align:center;">[f:id:motikan2010:20251216225740p:plain:w600]</div>

　そのプルリクエストに対して７つのコメントしか追加されていないため、**ソースコードの全てに対してセキュリティ検査が行われるわけではない**ことが確認できました。    
<div style="text-align:center;">[f:id:motikan2010:20251216230002p:plain:w600]</div>

 ➡︎ "７つ"の脆弱性が検出された後は検査が行われないような動作でした。（以降の各脆弱性でも同様に最大７つでした）  

### 各脆弱性のSAST

　ソースコード全体を検査することができないということが分かったので各脆弱性の検査を検証していきます。今回は「DVWA(PHP)」と「Java」の脆弱性について確認を行っています。  

<div style="text-align:center;">[f:id:motikan2010:20251217010128p:plain:w600]</div>

　脆弱性が検出された場合には以下の３項目が主に出力されるようになっていました。  

- What is the issue? （どのような問題ですか？）
- Why is this important? ）なぜこの問題が重要なのか？）
- What is the recommendation? （推奨される対策は？）

#### 結果一覧

　検査を行った脆弱性と検査結果は以下の通りです。  

| 脆弱性 | 脆弱性の検出 |
| - |: - :|
| Java - RCE | ❌ |
| Java - Directory Traversa | ✅ |
| Java - XML External Entity  | ✅ |
| Java - XSS (Tag Type)  | ✅ |
| Java - XSS (JSON Type)  | ✅ |
| Java - SQL Injection  | ✅ |
| Java - Insecure Deserialization  | ✅ |
| DVWA - Authorisation Bypass  | ❌ |
| DVWA - Command Injection  | ❌ |
| DVWA - File Inclusion  | ❌ |
| DVWA - Cross Site Scripting (Reflected)  | ❌ |
| DVWA - Cross Site Scripting (DOM Based) | ❌ |
| DVWA - Cross Site Scripting (Stored)  | ❌ |
| DVWA - API Security | ❓ |
| DVWA - Weak Session IDs | ❌ |
| DVWA - File Upload | ✅ |
| DVWA - SQL Injection (Blind) | ❌ |
| DVWA - SQL Injection | ❌ |
| DVWA - Open HTTP Redirect | ❌ |
| DVWA - Client Side JavaScript | ❌ |
| DVWA - Cross Site Request Forgery (CSRF) | ❌ |
| DVWA - Content Security Policy (CSP) Bypass | ❌ |
| DVWA - Cryptographic Problems  | ❌ |
| DVWA - Insecure CAPTCHA  | ❌ |
| DVWA - Brute Force (Login)  | ❌ |
| DVWA - Broken Access Contro | ❌ |

## まとめ

- 検出される脆弱性を存在しますが、見逃される脆弱性をいくつかあるようでした
  - LLM に頼りすぎているような印象で、既存の SAST の検出ロジックを活用できていない印象。
- まだ プレビュー版 なので今後のアップデートに期待です
  - 本検証段階では無償で利用できるため LLM に制限がかかっているかもしれないですね
- コメントの出力に関しても再現が容易な不具合が多少見られました
<div style="text-align:center;">[f:id:motikan2010:20251217005834p:plain:w400]</div>
- GA版(一般提供)に期待です！！
