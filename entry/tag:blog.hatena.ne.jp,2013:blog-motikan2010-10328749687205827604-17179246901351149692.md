<div style="text-align:center;">[f:id:motikan2010:20260204235230p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　本記事では、動作マシン上で<span style="color: #ff0000">擬似攻撃を行うツールである「**flightsim ( Network Flight Simulator )**」</span>を AWS のセキュリティサービスである<span style="color: #ff0000">「**Amazon GuardDuty**」の監視対象である EC2 インスタンスで動作させます。</span>その結果、 GuardDuty でどのようが脅威が検出されるのかを確認してみます。  

　そして最後にネットワーク監視だけでなく<span style="color: #ff0000">ランタイム監視ができるようにするため LLM (Claude Code) を利用して、「flightsim」に追加実装</span>を行ってみます。（※ 「Claude Code」は有償のサービスです。）  

[https://github.com/alphasoc/flightsim:embed:cite]  

### Amazon GuardDuty の説明

Amazon GuardDutyは、AWSアカウントとワークロードを保護するための脅威検出サービスです。  
検出できる脅威の例は以下の通りです。  

- 不正なアクセス試行や侵害されたインスタンス
- 暗号通貨マイニングなどの不審な活動
- データの不正な流出
- マルウェアやボットネットとの通信

### flightsim の説明

　「flightsim」の説明は以下の通りです。  

> 「**flightsim ( Network Flight Simulator )**」は、悪意のあるネットワークトラフィックを生成し、セキュリティチームがセキュリティ制御やネットワーク監視機能を評価する際に役立つ軽量ユーティリティです。  
このツールは、DNSトンネリング、DGA（動的生成ドメイン）トラフィック、既知のアクティブなC2（コマンド＆コントロール）サーバーへのリクエスト、その他の不審なトラフィックパターンなどをシミュレートするテストを実行します。

### LLM の説明

　すごく便利なサービスです。質問や欲しいプログラムを答えてくれます。   

　本記事では**「Claude Code」を利用するために「Claude Pro ($20 : 約3,000円/月)」の課金** をしています。 
（本記事のコードは 約30円 で生成）

## flightsim (Network Flight Simulator) の検証

　Amazon GuardDuty 監視内のインスタンス(Linux)内で **flightsim** のバイナリをダウンロードして、　「`./flightsim run`」を実行します。  

　擬似攻撃が完了するまでしばらく待ちます・・・。  

　 GuardDuty の検出ログに以下の検出が追加され、期待通り不審なネットワーク通信が検出されました。実ログではより詳細な情報が出力されています。    

[f:id:motikan2010:20260204205917p:plain]  

▼ 検出ログを AWS CLI を使って取得  
<div class="md-code" style="width:90%">
```sh
# 1. DetectorId 一覧の取得
aws guardduty list-detectors

# 2. FindingId 一覧の取得
aws guardduty list-findings --detector-id <DetectorId>

# 3. Finding (検知詳細) の取得
aws guardduty get-findings --detector-id <DetectorId> --finding-ids <FindingId>
```
</div>

## LLM(Claude Code) でモジュール拡張

　ネットワーク関連の擬似攻撃では、物足りないため**ランタイム関連の擬似攻撃を行う追加実装**を行います。  

　「`DefenseEvasion:Runtime/ProcessInjection.Proc`」で検知される疑似攻撃を行うモジュールを追加します。  
[f:id:motikan2010:20260205154919p:plain:w700]  

　せっかくなので LLM (Claude Code) を利用して実装を行うことにします。  

### 準備

　コードを自動的に生成してもらうために以下の準備をしています。  

- 編集対象であるリポジトリ「flightsim」を `/tmp` に配置
- GuardDuty の検出項目がまとめられているリポジトリを `/tmp` に配置 (`https://github.com/dgwhited/guardduty-runbooks`)

<div class="md-code" style="width:100%">

```
root@localhost:/tmp# git clone https://github.com/alphasoc/flightsim.git

root@localhost:/tmp# git clone https://github.com/dgwhited/guardduty-runbooks.git
```

</div>

### LLM に指示を与える

　VSCode で以下の命令を LLM(Claude Code) を与えました。「ツールの目的」と「実装して欲しい内容」、「実装に関する情報」を提供しています。  

<div class="md-code" style="width:100%">

```
このツールは擬似攻撃の挙動を行うツールです。
このソフトウェアに「DefenseEvasion:Runtime/ProcessInjection.Proc」の擬似攻撃を行うモジュールを追加してください。
guarddutyの検出項目詳細は「/tmp/guardduty-runbooks/runbooks/*.md」に記載されています。
```

</div>

　（しばらく待つ・・・）そして以下の流れで LLM が実装を行なわれました。  

🤖 早速、GuardDutyの検出詳細である「/tmp/guardduty-runbooks/runbooks/*.md」を確認しているようです。
[f:id:motikan2010:20260204205831p:plain]

🤖 検出詳細が記載されているファイルを発見して、内容を確認しているようです。  
[f:id:motikan2010:20260204205844p:plain]

🤖 それらの情報をもとに実装が完了したらしいです。  
[f:id:motikan2010:20260204205858p:plain]

### コードの修正後

　コードの修正は以下のように行われたようです。  

- 更新 : `v2/cmd/run/run.go`
- 作成 : `v2/simulator/process-injection.go`

#### 更新：v2/cmd/run/run.go

　**既存**のコード「`v2/cmd/run/run.go`」に追記されました。  

<div class="md-code" style="width:100%">

```diff
var allModules = []Module{
                Timeout:   3 * time.Second,
                HostMsg:   "Sending random data to %s",
        },
+       Module{
+               Module:     simulator.NewProcessInjection(),
+               Name:       "process-injection",
+               Pipeline:   PipelineIP,
+               NumOfHosts: 5,
+               HeaderMsg:  "Preparing to simulate process injection via /proc/PID/mem",
+               Timeout:    3 * time.Second,
+       },
 }
```

</div>

#### 作成：v2/simulator/process-injection.go

　**新規**のファイル「`v2/cmd/run/run.go`」が作成されました。  

<div class="md-code" style="width:100%">

```diff
+package simulator
+
+import (
+       "context"
+       "fmt"
+       "os"
+       "path/filepath"
+       "strconv"
+       "syscall"
+)
+
+// ProcessInjection simulates process injection via /proc/PID/mem
+// This triggers GuardDuty's DefenseEvasion:Runtime/ProcessInjection.Proc detection
+type ProcessInjection struct {
+       bind BindAddr
+}
+
+// NewProcessInjection creates a new process injection simulator
+func NewProcessInjection() *ProcessInjection {
+       return &ProcessInjection{}
+}
+
+func (s *ProcessInjection) Init(bind BindAddr) error {
+       s.bind = bind
+       return nil
+}
+
+func (s *ProcessInjection) Cleanup() {
+}
+
+// Hosts returns a list of PIDs to target for process injection simulation
+func (s *ProcessInjection) Hosts(scope string, size int) ([]string, error) {
+       pids, err := s.getPIDs(size)
+       if err != nil {
+               return nil, err
+       }
+       return pids, nil
+}
+
+// getPIDs returns a list of process PIDs from /proc
+func (s *ProcessInjection) getPIDs(size int) ([]string, error) {
+       entries, err := os.ReadDir("/proc")
+       if err != nil {
+               return nil, fmt.Errorf("failed to read /proc: %w", err)
+       }
+
+       var pids []string
+       myPID := os.Getpid()
+
+       for _, entry := range entries {
+               if !entry.IsDir() {
+                       continue
+               }
+
+               pid, err := strconv.Atoi(entry.Name())
+               if err != nil {
+                       continue
+               }
+
+               // Skip our own process
+               if pid == myPID {
+                       continue
+               }
+
+               pids = append(pids, entry.Name())
+               if len(pids) >= size {
+                       break
+               }
+       }
+
+       if len(pids) == 0 {
+               return nil, fmt.Errorf("no target processes found in /proc")
+       }
+
+       return pids, nil
+}
+
+// Simulate attempts to open /proc/PID/mem for writing to trigger GuardDuty detection
+func (s *ProcessInjection) Simulate(ctx context.Context, pid string) error {
+       memPath := filepath.Join("/proc", pid, "mem")
+
+       // Attempt to open /proc/PID/mem for writing
+       // This action triggers GuardDuty's DefenseEvasion:Runtime/ProcessInjection.Proc detection
+       // Even if the open fails due to permissions, the attempt may be logged
+       fd, err := syscall.Open(memPath, syscall.O_WRONLY, 0)
+       if err == nil {
+               // If we successfully opened the file, close it immediately
+               // We don't actually write anything - just the open attempt is enough
+               syscall.Close(fd)
+       }
+
+       // Also try with O_RDWR flag which is more commonly used in real attacks
+       fd, err = syscall.Open(memPath, syscall.O_RDWR, 0)
+       if err == nil {
+               syscall.Close(fd)
+       }
+
+       // Try using os.OpenFile as well for broader detection coverage
+       f, err := os.OpenFile(memPath, os.O_WRONLY, 0)
+       if err == nil {
+               f.Close()
+       }
+
+       f, err = os.OpenFile(memPath, os.O_RDWR, 0)
+       if err == nil {
+               f.Close()
+       }
+
+       // We don't return errors for permission denied - that's expected behavior
+       // The attempt itself is what triggers the detection
+       return nil
+}
+
+// HostMsg returns a custom message for the host
+func (s *ProcessInjection) HostMsg(host string) string {
+       return fmt.Sprintf("Attempting to open /proc/%s/mem for writing", host)
+}
```

</div>

### 動作確認

　修正後のコードをビルドし実行します。  

　実行ログを見ると、追加実装した「**Process Injection**」の擬似攻撃が行なわれました。  

<div class="md-code" style="width:100%">

```
[root@ip-172-31-31-153 tmp]# ./flightsim run

(...snip...)

11:29:49 [process-injection] Preparing to simulate process injection via /proc/PID/mem
11:29:49 [process-injection] Attempting to open /proc/1/mem for writing
11:29:52 [process-injection] Attempting to open /proc/10/mem for writing
11:29:55 [process-injection] Attempting to open /proc/11/mem for writing
11:29:58 [process-injection] Attempting to open /proc/12/mem for writing
11:30:01 [process-injection] Attempting to open /proc/122/mem for writing
11:30:04 [process-injection] Done (5/5)

(...snip...)
```

</div>

　GuardDuty の検出ログを見てみると期待どおり「Process Injection」が新規に検出されていました。  

[f:id:motikan2010:20260204205929p:plain]

## まとめ

- Amazon GuardDuty は有効化した方がよい
  - 本記事では "GuardDuty Runtime Monitoring" も有効化していますが導入は容易
- **LLM は便利**
  - LLM に"MITRE ATT&CK"などの情報を与えれば、もっと精度の高いコードを生成してくれる予感
  - 10分ほどで追加実装ができました
