<div style="text-align:center;">[f:id:motikan2010:20250409212603p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## 調査対象のランタイムセキュリティツール

検知ルールの項目が公開されている以下3つのランタイムセキュリティツールを調査しました。  
OSS として提供されているツールに関しては具体的な検知ルールを確認することが可能です。  

| ベンダー - プロダクト | 提供形式 | シグネチャ確認の可否 |
| - | - |: - :|
| **Falco Rules** - Sysdig | OSS | ✅ |
| **Tracee** - Aqua Security | OSS | ✅ |
| **GuardDuty Runtime Monitoring** - AWS | SaaS | ❌ |

## 検知ルールと MITRE ATT&CK のマッピング

検知ルールの中には MITRE ATT&CK の戦術(Tactics)と紐づいているものがあるので表にまとめてみました。戦術の 「Execution」、「Privilege Escalation」、「Defense Evasion」を検知するルールが多いようです。  
<span style="font-size: 80%">（※ 背景色の意味 青：Sysdig  - Falco Rules , 赤：Tracee - Aqua Security , 橙：GuardDuty Runtime Monitoring - AWS）</span>

<div style="margin:0;padding:0;border: solid #999;">
<style type="text/css">.ritz .waffle a { color: inherit; }.ritz .waffle .s3{background-color:#fce5cd;text-align:left;color:#000000;font-family:Arial;font-size:10pt;vertical-align:bottom;white-space:nowrap;direction:ltr;padding:2px 3px 2px 3px;}.ritz .waffle .s1{background-color:#f4cccc;text-align:left;color:#000000;font-family:Arial;font-size:10pt;vertical-align:bottom;white-space:nowrap;direction:ltr;padding:2px 3px 2px 3px;}.ritz .waffle .s0{background-color:#f3f3f3;text-align:center;color:#000000;font-family:Arial;font-size:10pt;vertical-align:bottom;white-space:nowrap;direction:ltr;padding:2px 3px 2px 3px;}.ritz .waffle .s2{background-color:#cfe2f3;text-align:left;color:#000000;font-family:Arial;font-size:10pt;vertical-align:bottom;white-space:nowrap;direction:ltr;padding:2px 3px 2px 3px;}</style>
<pre style="background-color: #fff;">
<div class="ritz grid-container" dir="ltr"><table class="waffle" cellspacing="0" cellpadding="0"><tbody><tr style="height: 20px"><td class="s0">Reconnaissance<br>偵察</td><td class="s0">Resource Development<br>リソース開発</td><td class="s0">Initial Access<br>初期アクセス</td><td class="s0">Execution<br>悪意あるプログラムの実行</td><td class="s0">Persistence<br>永続性</td><td class="s0">Privilege Escalation<br>特権昇格</td><td class="s0">Defense Evasion<br>防御回避</td><td class="s0">Credential Access<br>認証情報アクセス</td><td class="s0">Discovery<br>探索</td><td class="s0">Lateral Movement<br>水平展開</td><td class="s0">Collection<br>情報収集</td><td class="s0">Command and Control<br>C&amp;C</td><td class="s0">Exfiltration<br>データ流出</td><td class="s0">Impact<br>影響</td></tr><tr style="height: 20px"><td></td><td></td><td class="s1" dir="ltr">Web server spawned a shell</td><td class="s2" dir="ltr">Run shell untrusted</td><td class="s2" dir="ltr">Linux Kernel Module Injection Detected</td><td class="s2" dir="ltr">Debugfs Launched in Privileged Container</td><td class="s2" dir="ltr">Clear Log Activities</td><td class="s2" dir="ltr">Directory traversal monitored file read</td><td class="s2" dir="ltr">Contact K8S API Server From Container</td><td></td><td></td><td></td><td></td><td class="s2" dir="ltr">Remove Bulk Data from Disk</td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s2" dir="ltr">System user interactive</td><td class="s2" dir="ltr">Drop and execute new binary in container</td><td class="s2" dir="ltr">Detect release_agent File Container Escapes</td><td class="s2" dir="ltr">PTRACE anti-debug attempt</td><td class="s2" dir="ltr">Read sensitive file trusted after startup</td><td class="s1" dir="ltr">Kubernetes API server connection detected</td><td></td><td></td><td></td><td></td><td class="s3" dir="ltr">AbusedDomainRequest.Reputation</td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s2" dir="ltr">Terminal shell in container</td><td class="s1" dir="ltr">Kernel module loading detected</td><td class="s2" dir="ltr">PTRACE attached to process</td><td class="s1" dir="ltr">Code injection detected using ptrace</td><td class="s2" dir="ltr">Read sensitive file untrusted</td><td class="s1" dir="ltr">sched_debug CPU file was read</td><td></td><td></td><td></td><td></td><td class="s3" dir="ltr">BitcoinDomainRequest.Reputation</td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s2" dir="ltr">Netcat Remote Code Execution in Container</td><td class="s1" dir="ltr">Scheduled tasks modification detected</td><td class="s2" dir="ltr">Fileless execution via memfd_create</td><td class="s1" dir="ltr">Code injection detected using process_vm_writev syscall</td><td class="s2" dir="ltr">Search Private Keys or Passwords</td><td class="s3" dir="ltr">SuspiciousCommand</td><td></td><td></td><td></td><td></td><td class="s3" dir="ltr">MaliciousDomainRequest.Reputation</td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s2" dir="ltr">Redirect STDOUT/STDIN to Network Connection in Container</td><td class="s1" dir="ltr">Rcd modification detected</td><td class="s1" dir="ltr">Kcore memory file read</td><td class="s1" dir="ltr">File operations hooking on proc filesystem detected</td><td class="s2" dir="ltr">Create Symlink Over Sensitive Files</td><td></td><td></td><td></td><td></td><td></td><td class="s3" dir="ltr">SuspiciousDomainRequest.Reputation</td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s2" dir="ltr">Execution from /dev/shm</td><td class="s1" dir="ltr">LD_PRELOAD code injection detected</td><td class="s1" dir="ltr">Cgroups release agent file modification</td><td class="s1" dir="ltr">Default dynamic loader modification detected</td><td class="s2" dir="ltr">Create Hardlink Over Sensitive Files</td><td></td><td></td><td></td><td></td><td></td><td class="s3" dir="ltr">CryptoMinerExecuted</td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s2" dir="ltr">Disallowed SSH Connection Non Standard Port</td><td class="s3" dir="ltr">SuspiciousCommand</td><td class="s1" dir="ltr">Cgroups notify_on_release file modification</td><td class="s1" dir="ltr">Syscall table hooking detected</td><td class="s2" dir="ltr">Packet socket created in container</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s1" dir="ltr">Process standard input/output over socket detected</td><td></td><td class="s1" dir="ltr">Core dumps configuration file modification detected</td><td class="s1" dir="ltr">Fileless execution detected</td><td class="s2" dir="ltr">Find AWS Credentials</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s3" dir="ltr">NewBinaryExecuted</td><td></td><td class="s1" dir="ltr">Container device mount detected</td><td class="s1" dir="ltr">Hidden executable creation detected</td><td class="s1" dir="ltr">K8s service account token file read</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s3" dir="ltr">ReverseShell</td><td></td><td class="s1" dir="ltr">Docker socket abuse detected</td><td class="s1" dir="ltr">Anti-Debugging detected</td><td class="s1" dir="ltr">K8s TLS certificate theft detected</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s3" dir="ltr">NewLibraryLoaded</td><td></td><td class="s1" dir="ltr">Sudoers file modification detected</td><td class="s1" dir="ltr">Dynamic code loading detected</td><td class="s1" dir="ltr">Process memory access detected</td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s3" dir="ltr">SuspiciousTool</td><td></td><td class="s1" dir="ltr">System request key configuration modification</td><td class="s1" dir="ltr">New executable dropped</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s3" dir="ltr">SuspiciousCommand</td><td></td><td class="s1" dir="ltr">ASLR inspection detected</td><td class="s1" dir="ltr">Code injection detected through /proc/&lt;pid&gt;/mem file</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s3" dir="ltr">MaliciousFileExecuted</td><td></td><td class="s3" dir="ltr">DockerSocketAccessed</td><td class="s3" dir="ltr">ProcessInjection.Proc</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td class="s3" dir="ltr">SuspiciousShellCreated</td><td></td><td class="s3" dir="ltr">RuncContainerEscape</td><td class="s3" dir="ltr">ProcessInjection.Ptrace</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td></td><td></td><td class="s3" dir="ltr">CGroupsReleaseAgentModified</td><td class="s3" dir="ltr">ProcessInjection.VirtualMemoryWrite</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td></td><td></td><td class="s3" dir="ltr">ContainerMountsHostDirectory</td><td class="s3" dir="ltr">FilelessExecution</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td></td><td></td><td class="s3" dir="ltr">UserfaultfdUsage</td><td class="s3" dir="ltr">SuspiciousCommand</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td></td><td></td><td class="s3" dir="ltr">ElevationToRoot</td><td class="s3" dir="ltr">PtraceAntiDebugging</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr><tr style="height: 20px"><td></td><td></td><td></td><td></td><td></td><td class="s3" dir="ltr">SuspiciousCommand</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr></tbody></table></div>
</pre>
</div>

<div>
「GuardDuty Runtime Monitoring」の検知ルールは MITRE ATT&CK 以外にも、以下の4つの項目に分類されている模様。
<style type="text/css">.ritz .waffle a { color: inherit; }.ritz .waffle .s4{background-color:#ffffff;text-align:left;color:#000000;font-family:Arial;font-size:10pt;vertical-align:bottom;white-space:nowrap;direction:ltr;padding:2px 3px 2px 3px;}.ritz .waffle .s0{background-color:#f3f3f3;text-align:center;color:#000000;font-family:Arial;font-size:10pt;vertical-align:bottom;white-space:nowrap;direction:ltr;padding:2px 3px 2px 3px;}</style><div class="ritz grid-container" dir="ltr"><table class="waffle" cellspacing="0" cellpadding="0"><tbody><tr style="height: 20px"><td class="s0" dir="ltr">Backdoor</td><td class="s0" dir="ltr">CryptoCurrency</td><td class="s0" dir="ltr">Trojan</td><td class="s0" dir="ltr">UnauthorizedAccess</td></tr><tr style="height: 20px"><td class="s4" dir="ltr">C&amp;CActivity.B</td><td class="s4" dir="ltr">BitcoinTool.B</td><td class="s4" dir="ltr">BlackholeTraffic</td><td class="s4" dir="ltr">TorRelay</td></tr><tr style="height: 20px"><td class="s4" dir="ltr">C&amp;CActivity.B!DNS</td><td class="s4" dir="ltr">BitcoinTool.B!DNS</td><td class="s4" dir="ltr">DropPoint</td><td class="s4" dir="ltr">TorClient</td></tr><tr style="height: 20px"><td></td><td></td><td class="s4" dir="ltr">BlackholeTraffic!DNS</td><td class="s4" dir="ltr">MetadataDNSRebind</td></tr><tr style="height: 20px"><td></td><td></td><td class="s4" dir="ltr">DropPoint!DNS</td><td></td></tr><tr style="height: 20px"><td></td><td></td><td class="s4" dir="ltr">DGADomainRequest.C!DNS</td><td></td></tr><tr style="height: 20px"><td></td><td></td><td class="s4" dir="ltr">DriveBySourceTraffic!DNS</td><td></td></tr><tr style="height: 20px"><td></td><td></td><td class="s4" dir="ltr">PhishingDomainRequest!DNS</td><td></td></tr></tbody></table></div>
</div>

## ツール毎の検知ルール

項目名と検知説明です。（基本、機械翻訳なので読みづらい箇所があります）

### Falco Rules - Sysdig

[https://github.com/falcosecurity/rules/blob/8e4ed0c27da21b22b91bf07e416afefb3e1243da/rules/falco_rules.yaml:title]

#### ディレクトリトラバーサルによるファイル読み取り

`Directory traversal monitored file read`

　Webアプリケーションは、ルート・ディレクトリ外にあるファイルにアクセスするディレクトリ・トラバーサル攻撃に対して脆弱である可能性があります。  

「/etc」のようなシステムディレクトリは通常絶対パスでアクセスされます。  これ以外のアクセスパターン（ここではパストラバーサル）は疑わしいとみなされます。  
このルールには、ファイルオープンの失敗も含まれます。

#### 起動後に信頼された機密ファイルの読み込み

`Read sensitive file trusted after startup`

信頼されたプログラムが、起動後に機密ファイル（ユーザー/パスワード/認証情報を含むファイルなど）の読み込みの試行。  

信頼されたプログラムは、初期状態をロードするために起動時にこれらのファイルを読むかもしれないが、その後は読み込まない。必要に応じてカスタマイズできる。  
最新のコンテナ化されたクラウドインフラストラクチャでは、従来のLinux機密ファイルへのアクセスはあまり意味がないかもしれないが、ベースライン検出のための価値は残っている。  
SSHやクラウドベンダー固有の認証情報に対する追加ルールを提供していますが、お客様の環境に固有の重要なアプリケーション認証情報に対するカスタムルールを作成することで、セキュリティプログラムを大幅に強化することができます。

#### 信頼されない機密ファイルの読み込み

`Read sensitive file untrusted`

機密ファイル（ユーザー/パスワード/認証情報を含むファイルなど）の読み込みの試行。  

既知の信頼できるプログラムは例外となる。必要に応じてカスタマイズ可能。  最新のコンテナ化されたクラウドインフラストラクチャでは、従来のLinux機密ファイルへのアクセスはあまり意味がないかもしれませんが、ベースライン検出のための価値は残っています。    
SSHやクラウドベンダー固有の認証情報に対する追加ルールを提供していますが、お客様の環境に固有の重要なアプリケーション認証情報に対するカスタムルールを作成することで、セキュリティプログラムを大幅に強化することができます。    

#### 信頼されないシェルの実行

`Run shell untrusted`

非シェルアプリケーションの下にシェルを生成しようとする試み。  

監視される非シェル・アプリケーションは protected_shell_spawner マクロで定義され、 protected_shell_spawning_binaries は簡単にカスタマイズできるリストです。  
Java の親プロセスについては、Javaはしばしばカスタム・プロセス名を持っていることに注意してください。  
したがって、Javaアプリケーションを定義するには、proc.exeに依存するようにしてください。既存の徹底的なチューニングを見ればわかるように、このルールはノイズが多いかもしれない。  
しかし、このルールは非常にビヘイビア主導で広範であるため、一般的なリモートコード実行（RCE）を捕捉するために普遍的に関連します。ユースケースに合わせてこのルールをチューニングし、ノイズを減らすために時間を割いてください。  
チューニングの提案には、親プロセスの継続時間（proc.ppid.duration）を調べて、長時間実行するアプリプロセスを定義することが含まれます。  
直接の親プロセスが非シェル・アプリケーションである代わりに、proc.vpgid.name や proc.vpgid.exe などの新しいフィールドをチェックすることで、ルールをより堅牢にできる可能性があります。

#### システム・ユーザーとの対話

`System user interactive`

システムユーザー（例：ログインしていないユーザー）が新しいプロセスを生成しています。カスタムのサービスユーザー（例：apache や mysqld）を追加することもできます。

「インタラクティブ」とは、SSH セッションやログインプロセスの子孫プロセスとして新しいプロセスが生成されることを意味します。

さらに調整を行う場合は、端末 / tty 上で実行されているプロセスのみに注目することを検討してください（`proc.tty != 0`）。新しいフィールド proc.is_vpgid_leader は、そのプロセスが tty 上で「直接」実行されたかどうか、あるいはスクリプトから生成されたサブプロセスのように、同じプロセスグループ内の子孫プロセスとして実行されたかどうかを区別するのに役立つ可能性があります。

このルールは、システムへのインタラクティブなアクセスを広く監視するための優れたテンプレートルールと考えられます。ただし、そのようなカスタムルールは、あなたの環境固有のものとなるでしょう。

「コンテナ内での端末シェル」というルールは、kubectl exec を使用した場合に発火し、Kubernetes により関係がありますが、こちらのルールはより基盤となるホストに対して有用である可能性があります。

#### コンテナ内のターミナルシェル

`Terminal shell in container`

　シェルが端末に接続された状態でコンテナへのエントリポイント/実行ポイントとして使用されました。  

　親プロセスは正当な理由で既に終了しており、null になっている場合があります。これは Kubernetes で「kubectl exec」を使用する際によく見られます。
可能であれば、Kubernetes の監査ログ（k8saudit exec ログ）と相関させて、使用されたユーザーやサービスアカウントのトークンを特定してください（ネームスペースやポッド名によるあいまいな相関が可能です）。
このルールを単体で使うよりも、このコンテナ／TTY 内で他のルールが発火した際に併せて確認する、汎用的な監査ルールとして活用するのが望ましいでしょう。

#### コンテナから K8S API サーバーへ問い合わせ

`Contact K8S API Server From Container`

　プロファイルされていないユーザーによる、コンテナから Kubernetes API サーバーへの通信の試みを検出します。  

　Kubernetes の API は、クラスター管理のライフサイクルを構成する上で極めて重要な役割を果たします。そのため、API サーバーへの不正アクセスの可能性を検出することは非常に重要です。  
インフラ全体を監査し、ネットワーク構成に基づいて API サーバーにアクセス可能な可能性のあるマシンを特定してください。  
Falco をすべてのマシン上で動作させることができない場合は、追加のデータソースとして Kubernetes の監査ログ（通常はコントロールノードから収集されます）を分析することを検討してください。Falco には k8saudit プラグインが用意されており、コントロールプレーン内での検出に利用できます。

#### Netcat によるコンテナでのリモート・コード実行

`Netcat Remote Code Execution in Container`

Netcat プログラムは、リモートでのコード実行を可能にするコンテナ内で実行され、様々なリバース・シェル・ペイロード ( `https://github.com/swisskyrepo/PayloadsAllTheThings/` ) の一部として利用される可能性がある。  

これらのプログラムは、UNIX 系 OS に一般的にインストールされているため、関連性が高い。  
別の event.type を使用するため、「`Redirect STDOUT/STDIN to Network Connection in Container`」ルールと組み合わせて実行できます。  

#### 秘密鍵やパスワードの検索

`Search Private Keys or Passwords`

grep や find コマンドを使って秘密鍵やパスワードを検索しようとする試みを検出する。  

bash のビルトインを使用してファイルにアクセスする多くの方法があるため、これは気付かれない可能性があり、洗練されていない攻撃者によく見られます。  
いずれにせよ、これは、許容可能なノイズ・レベルを維持しながら、これらのギャップをカバーするように調整することができる、堅実なベースライン検出として機能する。

#### ログ・アクティビティの削除

`Clear Log Activities`

重要なアクセスログファイルの消去を検知します。これは通常、攻撃者の行動の証拠を消すために行われます。  


この検知を効果的にカスタマイズして運用するには、自分の環境に関連するログファイルの保存先が存在しない可能性があるか確認し、アラート対象外としたいプロファイル済みコンテナを調整してください。  

#### ディスクからデータを一括削除

`Remove Bulk Data from Disk`

データの破壊を意図してディスクから大量のデータを消去するために実行されているプロセスを検出し、システムの可用性を妨げる可能性がある。  

環境をプロファイルし、"user_known_remove_data_activities"を使用してこのルールを調整する。  

#### 機密ファイルへのハードリンクの作成

`Create Hardlink Over Sensitive Files`

　「`/etc/`」またはルート・ディレクトリの下にある、機密性の高いファイルやサブ・ディレクトリの厳選されたリスト上で作成されたハードリンクを検出する。  

必要に応じてカスタマイズできる。ルール「Read sensitive file untrusted（信頼できない機密ファイルの読み取り）」内の、さらなる同等のガイダンスを参照のこと。  

#### コンテナ内にパケットソケットを作成

`Packet socket created in container`

コンテナ内のデバイスドライバ（OSI レイヤ 2）レベルで新しいパケットソケットを検出します。  

パケットソケットは、攻撃者による ARP スプーフィングと権限昇格（CVE-2020-14386）に使われる可能性がある。
"user_known_packet_socket_binaries"テンプレートリストを使用することで、ノイズを減らすことができる。

#### コンテナ内で STDOUT/STDIN をネットワーク接続にリダイレクト

`Redirect STDOUT/STDIN to Network Connection in Container`

　コンテナ内のネットワーク接続への `stdout/stdin` のリダイレクトを検出する。  

これは、dup システムコールの亜種を使用することで実現される（リバースシェルまたはリモートコード実行の可能性 `https://github.com/swisskyrepo/PayloadsAllTheThings/` ）。
　この検出は動作ベースであり、システムにノイズを発生させる可能性があるため、「user_known_stand_streams_redirect_activities」テンプレートマクロを使用して調整できる。
チューニングは、プロセスリネージやコンテナイメージに基づく既存の検出と同様に実行したり、対話型tty（tty != 0）に制限したりすることができる。

#### Linux カーネル・モジュール・インジェクションを検出

`Linux Kernel Module Injection Detected`

　`init_module` と `finit_module` の`sys`コールを使って、insmodまたはmodprobeを使ってコンテナからLinuxカーネルモジュールを注入する。  

ノイズを減らし、正当なケースを考慮するために、環境をプロファイルし、「`allowed_container_images_loading_kernel_module`」を考慮する。

#### 特権コンテナで Debugfs が起動

`Debugfs Launched in Privileged Container`

特権コンテナ内で起動されたファイルシステムデバッガ「debugfs」を検出する。このルールの適用範囲はより狭い。

#### release_agent ファイルのコンテナ脱出を検出

`Detect release_agent File Container Escapes`

「release_agent」ファイルを使用したコンテナ脱出を悪用する試みを検出する。  

コンテナを特定の機能で実行することで、特権ユーザは「release_agent」ファイルを変更し、コンテナから脱出することができる。

#### プロセスに PTRACE をアタッチ

`PTRACE attached to process`

　プロセスベースの防御を回避したり、特権を昇格させたりするために、PTRACE を使って潜在的に悪意のあるコードをプロセスに注入しようとする試みを検出する。  

一般的なアンチパターンはデバッガです。さらに、「`known_ptrace_procs`」テンプレートマクロを使用して環境をプロファイリングすることで、ノイズを減らすことができます。  
ptrace システムコールが成功すると、一度に複数のログが生成されます。

#### PTRACE によるアンチデバッグの試行

`PTRACE anti-debug attempt`

　`PTRACE_TRACEME` 引数で PTRACE システムコールの使用を検出します。  

これは、デバッガがプロセスにアタッチされるのを回避しようとプログラムが積極的に試みていることを示します。  
PTRACE の詳細については、「`PTRACE attached to process`」ルールを参照してください。  

#### AWS クレデンシャルの探索

`Find AWS Credentials`

特に標準的な AWS クレデンシャルの場所を狙って、grep や find コマンドを使って秘密鍵やパスワードを検索しようとする試みを検出する。  

これは、気付かれない可能性のある bash 組み込みインを使用してファイルにアクセスする多くの方法があるため、素朴な攻撃者によく見られる。  
いずれにせよ、これは、許容可能なノイズレベルを維持しながら、これらのギャップをカバーするように調整できる、強固なベースライン検出として機能する。  
このルールは「`Search Private Keys or Passwords`」ルールを補完する。

#### /dev/shm からの実行

`Execution from /dev/shm`

　このルールは、`/dev/shm` ディレクトリでのファイル実行を検出します。  

`/dev/shm`ディレクトリは、脅威行為者が読み取り可能、書き込み可能、時には実行可能なファイルを格納するためによく使用する手口です。  

「dev/shm」は、ホストや他のコンテナへのリンクとして機能し、同様にそれらの侵害のための脆弱性を作成します。
特に、/dev/shmはコンテナ再起動後も変更されません。このルールは、より新しい「`Drop and execute new binary in container`」ルールと並行して検討する。

#### コンテナに新しいバイナリをドロップして実行

`Drop and execute new binary in container`

コンテナのベースイメージに属さない実行ファイルが実行されているかどうかを検出する。  

ドロップして実行するパターンは、攻撃者が最初の足掛かりを得た後、非常に頻繁に観測される。  
「is_exe_upper_layer 」フィルターフィールドは、overlayfsをユニオンマウントファイルシステムとして使用するコンテナランタイムにのみ適用されます。  
採用者は、提供されているテンプレートリスト `known_drop_and_execute_containers` を利用することができます。このリストには、ベースイメージに含まれていないバイナリを実行することが知られている、許可されたコンテナイメージが含まれています。  

あるいは、ルールをさらに調整することで、Kubernetesの設定で本番環境以外のネームスペースを除外することもできます。これは、アプリケーションや環境固有の知識をこのルールに適用することで、ノイズを減らすのに役立ちます。よくあるアンチパターンには、管理者やSREが以下を実行することが含まれる。  

#### 非標準ポートのSSH接続を拒否

`Disallowed SSH Connection Non Standard Port`

　非標準ポートを使用した、ホストまたはコンテナからの新しいアウトバウンド SSH 接続を検出する。  

このルールは、SSH 接続からシェルの STDIN にパイプされた STDIN と、SSH 経由でパイプされたシェルの STDOUT を使って、被害マシンを SSH 経由で接続し返すリバースシェルのファミリを検出する可能性がある。  
このような攻撃は、コマンドインジェクションに脆弱なアプリであれば、どのようなアプリに対しても可能です。アップストリームルールは、限られた非標準ポートのみを対象としています。  
より多くのポートを追加し、環境の知識やカスタムSSHポート設定に基づいた範囲を組み込むことをお勧めします。  
このルールは、「`Redirect STDOUT/STDIN to Network Connection in Container`」または「`Disallowed SSH Connection`」ルールを補完することができる。  

#### memfd_create によるファイルレスの実行

`Fileless execution via memfd_create`

　memfd_create テクニックを使用して、バイナリがメモリから実行されたかどうかを検出します。  

これは、ペイロードをディスクに保存することなく被害者マシン上でマルウェアを実行し、実行されたものについての痕跡を残さないようにするための、よく知られた防御回避テクニックです。  
採用者は、"known_memfd_execution_processes" リストに項目を追加することで、良性の目的でファイルレス実行を使用する可能性のあるプロセスをホワイトリストに登録することができます。

### Tracee - Aqua Security

[https://github.com/aquasecurity/tracee/tree/cd3557304d9637e1b41cc90a13232cde8c65e46e/signatures/golang:title]

#### Kcoreメモリファイルの読み込み

`Kcore memory file read`

　proc/kcore ファイルを読み込もうとする試みが検出されました。  

KCore はシステムの物理メモリの完全なダンプをコアファイル形式で提供する。  
攻撃者はこのファイルを読み込んでホストの全メモリを取得し、この情報をコンテナエスケープに使用する可能性があります。

#### Cgroups リリース・エージェント・ファイルの変更

`Cgroups release agent file modification`

Cgroup release agent ファイルを変更しようとする試みが検出されました。  
Cgroup は Linux カーネルの機能であり、一連のプロセスのリソース使用を制限します。攻撃者はコンテナのエスケープにこの機能を使用する可能性があります。

#### K8s サービスアカウントトークンファイルの読み取り

`K8s service account token file read`

コンテナ上で Kubernetes サービスアカウントトークンファイルが読み込まれました。  

このトークンは Kubernetes API Server と通信するために使用されます。  
攻撃者はAPI Serverと通信して情報および/または認証情報を盗もうとしたり、さらにコンテナを実行してシステムに対する支配力を横方向に拡大しようとしたりする可能性があります。

#### ptrace を使用したコード・インジェクションの検出

`Code injection detected using ptrace`

別のプロセスへのコードインジェクションの可能性が検出されました。  

コードインジェクションは、悪意のあるコードを実行するために使用される悪用テクニックであり、敵対者はマルウェアを実行するためにこれを使用する可能性があります。

#### Cgroups notify_on_release ファイルの変更

`Cgroups notify_on_release file modification`

　Cgroup notify_on_release ファイルを変更しようとする試みが検出された。  

Cgroup は Linux カーネルの機能であり、一連のプロセスのリソース使用を制限する。攻撃者は、コンテナのエスケープにこの機能を使用する可能性があります。

#### コアダンプの設定ファイルの変更を検出

`Core dumps configuration file modification detected`

コアダンプ設定ファイル（core_pattern）の変更が検出された。  

コアダンプは通常、プログラムがクラッシュしたときにディスクに書き込まれる。特定の変更により、カーネルの core_pattern 機能を使ったコンテナ・エスケープが可能になる。

#### process_vm_writev システムコールを使用したコードインジェクションを検出

`Code injection detected using process_vm_writev syscall`

別のプロセスへのコードインジェクションの可能性が検出されました。コードインジェクションは、悪意のあるコードを実行するために使用される悪用テクニックであり、敵対者はマルウェアを実行するためにこれを使用する可能性があります。

#### Kubernetes API サーバー接続を検出

`Kubernetes API server connection detected`

kubernetes APIサーバーへの接続が検出されました。K8S APIサーバはK8Sクラスタの頭脳であり、敵対者はK8S APIサーバと通信して情報/認証情報を収集しようとしたり、さらにコンテナを実行してシステムに対する支配力を横展開しようとしたりする可能性があります。

#### K8s TLS 証明書の盗難を検出

`K8s TLS certificate theft detected`

Kubernetes のTLS証明書の盗難が検出された。  

TLS証明書は、システム間の信頼を確立するために使用されます。Kubernetes 証明書は、kubeletスケジューラーコントローラーやAPIサーバーなどの Kubernetes コンポーネント間のセキュアな通信を可能にするために使用されます。  
敵対者は、クラスタ内のKubernetesコンポーネントになりすますために、侵害されたシステム上のKubernetes証明書を盗む可能性があります。

#### コンテナ・デバイスのマウントを検出

`Container device mount detected`

コンテナ・デバイス・ファイルシステムのマウントが検出された。  

ホストデバイスファイルシステムのマウントは、コンテナエスケープを実行するために悪用される可能性があります。

#### Docker ソケットの不正使用を検出

`Docker socket abuse detected`

docker.sock は、Docker APIへのエントリポイントとして Docker が使用するUNIXソケットです。  
攻撃者は、このソケットを悪用してシステムを侵害しようとする可能性があります。

#### proc ファイルシステムでファイル操作のフックを検出

`File operations hooking on proc filesystem detected`

proc ファイルシステムでファイル操作のフックが検出された。proc ファイルシステムは実行中のプロセスをファイルとして扱うためのインターフェースである。これにより、`ps`や`top`のようなプログラムは実行中のプロセスを確認することができる。ファイル操作はファイルやディレクトリに対して定義される関数である。ファイル操作のフックには、ファイルの列挙のようなファイルやディレクトリに対する基本的なタスクを実行するために使用されるデフォルト関数を置き換えることが含まれる。敵対者は、/proc のファイル操作をフックすることで、ファイル一覧やオペレーションシステムで実行されるその他の基本機能など、特定のシステム機能を制御できるようになる。また、実行フローを乗っ取り、独自のコードを実行することもできる。ファイル操作のフッキングは、ルートキットによって実行される悪意のある動作とみなされ、ホストのカーネルが侵害されていることを示す可能性があります。隠されたモジュールは隠されたシンボルの所有者としてマークされ、敵対者のさらなる悪意のある活動を示します。

#### Sudoersファイルの変更を検出

`Sudoers file modification detected`

　sudoers ファイルが変更されました。  

sudoers ファイルは、sudo 機能の権限とオプションを制御する設定ファイルです。  
攻撃者は sudoers ファイルを変更して、特権を昇格させたり、他のユーザーとしてコマンドを実行したり、より高い特権を持つプロセスを生成したりする可能性があります。

#### ソケット経由で標準入出力の処理を検出

`Process standard input/output over socket detected`

  プロセスの標準入出力がソケットにリダイレクトされる。  

この動作はリバース・シェル攻撃の基本であり、攻撃対象のマシンから攻撃者のマシンにインタラクティブ・シェルが呼び出され、攻撃対象をインタラクティブに制御できるようになる。  
攻撃者はリバース・シェルを使用して、ネットワーク・ファイアウォールなどのセキュリティ対策を回避しながら、侵害された標的を制御することができる。

#### デフォルトのダイナミックローダーの変更を検出

`Default dynamic loader modification detected`

デフォルトのダイナミック・ローダーが変更された。ダイナミック・ローダーはプロセスのメモリにロードされる実行ファイルで、実行ファイルの前に実行され、プロセスにダイナミック・ライブラリをロードする。攻撃者はこのテクニックを使って、新しい各プロセスの実行コンテキストを乗っ取り、防御を迂回する可能性がある。

#### シスコールテーブルのフックを検出

`Syscall table hooking detected`

Syscall テーブル・フッキングが検出された。  

システムコールは、ユーザーアプリケーションとカーネル間のインターフェースである。シスコールテーブルをフックすることで、敵はファイルの書き込みや読み出しなど、オペレーションシステムによって実行される基本的な機能など、特定のシステム機能を制御できるようになります。  
また、実行フローを乗っ取り、独自のコードを実行することもできる。シスコールテーブルフッキングは、ルートキットによって実行される悪意のある動作とみなされ、ホストのカーネルが侵害されていることを示す可能性があります。  
隠しモジュールは隠しシンボル所有者としてマークされ、敵対者のさらなる悪意のある活動を示します。

#### システム・リクエスト・キーの設定変更

`System request key configuration modification`

システムリクエストキー設定ファイルを変更し、アクティブにしようとする試みが検出された。  

システムリクエストキーは、単純なキーの組み合わせによってカーネルへの即時入力を可能にする。攻撃者はこの機能を使って、システムを即座にシャットダウンしたり再起動したりすることができる。  
カーネルログへの読み取りアクセスにより、リストタスクや CPU レジスタなどのホスト関連情報が開示され、コンテナエスケープに使用される可能性があります。

#### sched_debug CPUファイルを読み込み

`sched_debug CPU file was read`

scheduled_debug ファイルが読み込まれた。  

このファイルにはCPUとプロセスに関する情報が含まれている。攻撃者は、その情報を収集するためにこのファイルを読むかもしれない。

#### カーネルモジュールのロードを検出

`Kernel module loading detected`

カーネルモジュールのロードを検出する。  

カーネルモジュールとは、カーネル内で実行するためのバイナリである。攻撃者は、カーネルモジュールをロードして機能を拡張し、ユーザー空間ではなくカーネルで実行することで検知を回避しようとする可能性がある。

#### ファイルレスの実行を検出

`Fileless execution detected`

ファイルレス実行が検出された。  

ファイルシステム内のファイルからではなく、メモリからプロセスを実行することは、敵が実行の検出を避けようとしていることを示す可能性がある。

#### 隠し実行ファイルの作成を検出

`Hidden executable creation detected`

隠された実行ファイル（ELFファイル）がディスク上に作成された。  

このアクティビティは合法的なものである可能性もあるが、敵対者がプログラムを隠すことで検知を回避しようとしていることを示している可能性もある。

#### スケジュールされたタスクの変更を検出

`Scheduled tasks modification detected`

タスクスケジューリング機能またはファイルが変更された。  

Crontab はブート時にタスクの実行をスケジュールしたり、タスクの実行を有効にしたりする。  
攻撃者は、再起動を持続させるためにスケジュールされたタスクを追加または変更し、影響を受けるホスト上で悪意のある実行を維持する可能性があります。

#### Rcd の変更を検出

`Rcd modification detected`

rcd ファイルはブート時とランレベルスイッチ時に実行されるスクリプトである。  

これらのスクリプトはランレベルスイッチのサービス制御を担当する。攻撃者は、再起動を持続させるためにrcdファイルを追加または変更し、感染したホスト上で悪意のある実行を維持する可能性がある。

#### アンチデバッグを検出

`Anti-Debugging detected`

デバッガーをブロックするためにアンチデバッギング技術を使用したプロセス。マルウェアは、アンチデバッギングを使用して、目に見えない状態を維持し、その動作の分析を阻害する。

#### 動的コードのロードを検出

`Dynamic code loading detected`

バイナリのメモリが書き込み可能かつ実行可能であるため、動的なコードロードの可能性が検出された。  

実行可能に割り当てられたメモリ領域への書き込みは、敵対者が実行可能ファイルを削除することなく、検出されずにコードを実行するために使用する手法である可能性がある。

#### 新しい実行ファイルをドロップ

`New executable dropped`

実行時に実行可能ファイルがシステムにドロップされました。  

コンテナイメージは通常、必要なすべてのバイナリを内蔵してビルドされます。バイナリがドロップされた場合、敵がコンテナに侵入した可能性があります。

#### プロセスメモリへのアクセスを検出

`Process memory access detected`

プロセスのメモリアクセスが検出された。  

攻撃者は他のプロセスのメモリにアクセスし、認証情報や秘密を盗む可能性がある。

#### "`/proc/<pid>/mem`"ファイルからコード・インジェクションを検出

`Code injection detected through /proc/<pid>/mem file`

別のプロセスへのコードインジェクションの可能性が検出されました。  

コードインジェクションは、悪意のあるコードを実行するために使用される悪用テクニックであり、敵対者はマルウェアを実行するためにこれを使用する可能性があります。

#### ASLR（Address Space Layout Randomization）の検査を検出

`ASLR inspection detected`

ASLR（アドレス空間配置のランダム化）の設定が検査された。  

ASLR はメモリの脆弱性を防ぐためにLinuxで使用されている。攻撃者は検知を避けるためにASLR設定を検査し、変更したいと考えるかもしれない。

#### ウェブサーバーがシェルを起動

`Web server spawned a shell`

サーバー上のウェブサーバープログラムがシェルプログラムを生成しました。  

シェルはLinuxのコマンドライン・プログラムであり、通常、ウェブ・サーバはシェル・プログラムを実行しないため、この警告は、敵がウェブ・サーバ・プログラムを悪用してサーバ上でコマンドを実行しようとしていることを示しているのかもしれない。

#### LD_PRELOAD コードインジェクションを検出

`LD_PRELOAD code injection detected`

LD_PRELOAD の使用が検出されました。  

LD_PRELOADを使用すると、他のどのライブラリよりも先に自分のライブラリをロードすることができ、プロセス内の関数をフックできるようになります。  
攻撃者はこのテクニックを使って、アプリケーションの動作を変更したり、独自のプログラムをロードしたりする可能性があります。

### GuardDuty Runtime Monitoring - AWS

[https://docs.aws.amazon.com/ja_jp/guardduty/latest/ug/findings-runtime-monitoring.html:title]

#### 暗号通貨関連のアクティビティに関連付けられている IP アドレスをクエリを検出 ( CryptoCurrency )

`CryptoCurrency:Runtime/BitcoinTool.B`

Amazon EC2 インスタンスまたはコンテナが暗号通貨関連のアクティビティに関連付けられている IP アドレスをクエリしています。

#### 既知のコマンドとコントロールサーバーに関連付けられる IP をクエリを検出 ( Backdoor )

`Backdoor:Runtime/C&CActivity.B`

Amazon EC2 インスタンスまたはコンテナは、既知のコマンドとコントロールサーバーに関連付けられる IP をクエリしています。

#### Tor リレーとして Tor ネットワークに接続を検出 ( UnauthorizedAccess )

`UnauthorizedAccess:Runtime/TorRelay`

Amazon EC2 インスタンスまたはコンテナが Tor リレーとして Tor ネットワークに接続しています。

#### Tor Guard または Authority ノードに接続を検出 ( UnauthorizedAccess )

`UnauthorizedAccess:Runtime/TorClient`

Amazon EC2 インスタンスまたはコンテナが Tor Guard または Authority ノードに接続しています。

#### 既知のブラックホールであるリモートホストのIPアドレスへの通信を検出 ( Trojan )

`Trojan:Runtime/BlackholeTraffic`

Amazon EC2 インスタンスまたはコンテナが既知のブラックホールであるリモートホストのIPアドレスに通信しようとしています。

#### 盗難されたデータを保持していることが認識されているリモートホストのIPアドレスへの通信を検出 ( Trojan )

`Trojan:Runtime/DropPoint`

Amazon EC2 インスタンスまたはコンテナが、マルウェアによって収集された認証情報やその他の盗難されたデータを保持していることが認識されているリモートホストの IP アドレスに通信しようとしています。

#### 暗号通貨のアクティビティに関連付けられているドメイン名をクエリを検出 ( CryptoCurrency )

`CryptoCurrency:Runtime/BitcoinTool.B!DNS`

Amazon EC2 インスタンスまたはコンテナが暗号通貨のアクティビティに関連付けられているドメイン名をクエリしています。

#### 既知のコマンドとコントロールサーバーに関連付けられるドメイン名をクエリを検出 ( Backdoor )

`Backdoor:Runtime/C&CActivity.B!DNS`

Amazon EC2 インスタンスまたはコンテナが、既知のコマンドとコントロールサーバーに関連付けられるドメイン名をクエリしています。

#### ブラックホールの IP アドレスにリダイレクトされるドメイン名をクエリを検出 ( Trojan )

`Trojan:Runtime/BlackholeTraffic!DNS`

Amazon EC2 インスタンスまたはコンテナがブラックホールのIPアドレスにリダイレクトされるドメイン名をクエリしています。

#### 盗難されたデータを保持していることが認識されているリモートホストのドメイン名をクエリを検出 ( Trojan )

`Trojan:Runtime/DropPoint!DNS`

Amazon EC2 インスタンスまたはコンテナがマルウェアによって収集された認証情報やその他の盗難されたデータを保持していることが認識されているリモートホストのドメイン名をクエリしています。

#### アルゴリズムを使用して生成されたドメインをクエリを検出 ( Trojan )

`Trojan:Runtime/DGADomainRequest.C!DNS`

Amazon EC2 インスタンスまたはコンテナがアルゴリズムを使用して生成されたドメインをクエリしています。 

このようなドメインは、マルウェアによって悪用されることが多く、EC2 インスタンスまたはコンテナが侵害されている場合があります。

#### ドライブバイダウンロード攻撃の既知の攻撃元であるリモートホストのドメイン名をクエリを検出 ( Trojan )

`Trojan:Runtime/DriveBySourceTraffic!DNS`

Amazon EC2 インスタンスまたはコンテナがドライブバイダウンロード攻撃の既知の攻撃元であるリモートホストのドメイン名をクエリしています。

#### フィッシング攻撃に関与しているドメインをクエリを検出 ( Trojan )

`Trojan:Runtime/PhishingDomainRequest!DNS`

Amazon EC2 インスタンスまたはコンテナがフィッシング攻撃に関与しているドメインをクエリしています。

#### 既知の悪用されたドメインに関連付けられた評価の低いドメイン名をクエリを検出 ( Impact )

`Impact:Runtime/AbusedDomainRequest.Reputation`

Amazon EC2 インスタンスまたはコンテナが、既知の悪用されたドメインに関連付けられた評価の低いドメイン名をクエリしています。

#### 暗号通貨関連のアクティビティに関連付けられている評判の低いドメイン名をクエリを検出 ( Impact )

`Impact:Runtime/BitcoinDomainRequest.Reputation`

Amazon EC2 インスタンスまたはコンテナが、暗号通貨関連のアクティビティに関連付けられている評判の低いドメイン名をクエリしています。

#### 悪意のある既知のドメインに関連付けられた評判の低いドメインをクエリを検出 ( Impact )

`Impact:Runtime/MaliciousDomainRequest.Reputation`

Amazon EC2 インスタンスまたはコンテナが、悪意のある既知のドメインに関連付けられた評判の低いドメインをクエリしています。

#### 年齢や低人気により、本質的に疑わしい、低評判のドメイン名をクエリを検出 ( Impact )

`Impact:Runtime/SuspiciousDomainRequest.Reputation`

Amazon EC2 インスタンスまたはコンテナが、年齢や低人気により、本質的に疑わしい、低評判のドメイン名をクエリしています。

#### インスタンスメタデータサービスに解決される DNS ルックアップの実行を検出 ( UnauthorizedAccess )

`UnauthorizedAccess:Runtime/MetadataDNSRebind`

Amazon EC2 インスタンスまたはコンテナがインスタンスメタデータサービスに解決される DNS ルックアップを実行しています。

#### 新しく作成・最近変更されたバイナリファイルの実行の検出 ( Execution )

`Execution:Runtime/NewBinaryExecuted`

コンテナで新しく作成、または最近変更されたバイナリファイルが実行されました。

#### Docker ソケットを使用して Docker デーモンと通信を検出 ( PrivilegeEscalation )

`PrivilegeEscalation:Runtime/DockerSocketAccessed`

コンテナ内のプロセスは、Docker ソケットを使用して Docker デーモンと通信しています。

#### runC を介したコンテナエスケープの試行を検出 ( PrivilegeEscalation )

`PrivilegeEscalation:Runtime/RuncContainerEscape`

runC を介したコンテナエスケープの試行が検出されました。

#### CGroups リリースエージェントを介したコンテナエスケープの試行を検出 ( PrivilegeEscalation )

`PrivilegeEscalation:Runtime/CGroupsReleaseAgentModified`

CGroups リリースエージェントを介したコンテナエスケープの試行が検出されました。

#### proc ファイルシステムを使用したプロセスインジェクションを検出 ( DefenseEvasion )

`DefenseEvasion:Runtime/ProcessInjection.Proc`

proc ファイルシステムを使用したプロセスインジェクションが、コンテナまたは Amazon EC2 インスタンスで検出されました。

#### ptrace システムコールを使用したプロセスインジェクションを検出 ( DefenseEvasion )

`DefenseEvasion:Runtime/ProcessInjection.Ptrace`

ptrace システムコールを使用したプロセスインジェクションが、コンテナまたは Amazon EC2 インスタンスで検出されました。

#### 仮想メモリへの直接書き込みによるプロセスインジェクションを検出 ( DefenseEvasion )

`DefenseEvasion:Runtime/ProcessInjection.VirtualMemoryWrite`

仮想メモリへの直接書き込みによるプロセスインジェクションが、コンテナまたは Amazon EC2 インスタンスで検出されました。

#### プロセスによってリバースシェルの作成を検出 ( Execution )

`Execution:Runtime/ReverseShell`

コンテナまたは Amazon EC2 インスタンス内のプロセスによってリバースシェルが作成されました。

#### プロセスがメモリからコードの実行を検出 ( DefenseEvasion )

`DefenseEvasion:Runtime/FilelessExecution`

コンテナまたは Amazon EC2 インスタンス内のプロセスがメモリからコードを実行しています。

#### 暗号通貨マイニングアクティビティに関連するバイナリファイルの実行を検出 ( Impact )

`Impact:Runtime/CryptoMinerExecuted`

コンテナまたは Amazon EC2 インスタンスが、暗号通貨マイニングアクティビティに関連するバイナリファイルを実行しています。

#### 新しく作成・最近変更されたライブラリのプロセスへのロードを検出 ( Execution )

`Execution:Runtime/NewLibraryLoaded`

新しく作成または最近変更されたライブラリが、コンテナ内のプロセスによってロードされました。

#### ホストファイルシステムのマウントを検出 ( PrivilegeEscalation )

`PrivilegeEscalation:Runtime/ContainerMountsHostDirectory`

コンテナ内のプロセスが、実行時にホストファイルシステムをマウントしました。

#### userfaultfd システム呼び出しを使用してユーザースペースのページフォールトの処理を検出 ( PrivilegeEscalation )

`PrivilegeEscalation:Runtime/UserfaultfdUsage`

あるプロセスが userfaultfd システム呼び出しを使用してユーザースペースのページフォールトを処理しました。

#### 攻撃的なセキュリティシナリオで使用されるバイナリファイルまたはスクリプトの実行を検出

`Execution:Runtime/SuspiciousTool`

コンテナまたは Amazon EC2 インスタンスはペンテストエンゲージメントなどの攻撃的なセキュリティシナリオで頻繁に使用されるバイナリファイルまたはスクリプトを実行しています。

#### 侵害を示す疑わしいコマンドの実行を検出 ( Execution )

`Execution:Runtime/SuspiciousCommand`

侵害を示す Amazon EC2 インスタンスまたはコンテナで疑わしいコマンドが実行されました。

#### Linux 防御メカニズムの変更/無効化の検出 ( DefenseEvasion )

`DefenseEvasion:Runtime/SuspiciousCommand`

リストされている Amazon EC2 インスタンスまたはコンテナでコマンドが実行され、ファイアウォールや重要なシステムサービスなどの Linux 防御メカニズムを変更または無効化しようとしています。

#### ptrace システムコールを使用してアンチデバッグ対策の実行を検出 ( DefenseEvasion )

`DefenseEvasion:Runtime/PtraceAntiDebugging`

コンテナまたは Amazon EC2 インスタンスのプロセスが、ptrace システムコールを使用してアンチデバッグ対策を実行しました。

#### 悪意のある実行可能ファイルの実行を検出 ( Execution )

`Execution:Runtime/MaliciousFileExecuted`

既知の悪意のある実行可能ファイルが Amazon EC2 インスタンスまたはコンテナで実行されました。

#### インタラクティブシェルプロセスを開始を検出 ( Execution )

`Execution:Runtime/SuspiciousShellCreated`

Amazon EC2 インスタンスまたはコンテナ内のネットワークサービスまたはネットワークからアクセス可能なプロセスがインタラクティブシェルプロセスを開始しました。

#### プロセスがルート権限を引き受けを検出 ( PrivilegeEscalation )

`PrivilegeEscalation:Runtime/ElevationToRoot`

リストされた Amazon EC2 インスタンスまたはコンテナで実行されているプロセスがルート権限を引き受けています。

#### 不審なコマンドの実行を検出 ( Discovery )

`Discovery:Runtime/SuspiciousCommand`

不審なコマンドが Amazon EC2 インスタンスまたはコンテナで実行され、攻撃者がローカルシステム、周囲の AWS インフラストラクチャ、またはコンテナインフラストラクチャに関する情報を取得できるようになりました。

#### 不審なコマンドの実行を検出 ( Persistence )

`Persistence:Runtime/SuspiciousCommand`

不審なコマンドが Amazon EC2 インスタンスまたはコンテナで実行され、攻撃者がお客様の AWS 環境でアクセスと制御を維持できるようになりました。

#### 不審なコマンドの実行されたことを検出 ( PrivilegeEscalation )

`PrivilegeEscalation:Runtime/SuspiciousCommand`

不審なコマンドが Amazon EC2 インスタンスまたはコンテナで実行されたことにより、攻撃者が権限を昇格できます。

## Grok ファクトチェック

[https://x.com/grok/status/1909947238022799745:embed]

[blog:g:11696248318754550880:banner]
[blog:g:12921228815726579926:banner]




