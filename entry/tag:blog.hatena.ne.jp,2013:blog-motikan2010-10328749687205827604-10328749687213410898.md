<div style="text-align: center;">[f:id:motikan2010:20170204200342p:plain:w600]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに  

　IoTというワードが流行りつつあり、<span class="m-y">Bluetoothの通信を見てみたい</span>という欲求が出てきたので、Bluetooth通信のプロキシを探してみました。  

「btproxy」というツールが良さげ、このキャプチャ技術を応用したら以下の場面などで活用できそう。  

- Bluetoothデバイス開発のデバッグ  
- Bluetoothデバイスに対してのセキュリティ診断  

▼ btproxy のリポジトリ  
[https://github.com/conorpp/btproxy:embed:cite]

[f:id:motikan2010:20170204200304p:plain:w600]

　本記事では、「RaspberryPi3 (以下ラズパイ)」にbtproxyをインストールし、<span class="m-y">Macbook Proからスマートフォンへのファイル転送をキャプチャ</span>します。

　ラズパイのOSは「**2016-03-18-raspbian-jessie.img**」を使用します。  

[https://www.raspberrypi.org/downloads/raspbian/:title]


　では、早速導入していきましょう。  

## btproxyをRaspberryPi3に導入

### BlueZをインストールする前の下準備

　Linux上でBluetoothを扱うには **BlueZ** をインストールする必要があります。  
まずはBlueZをインストールするために必要な依存パッケージをインストールします。

#### 必要パッケージのインストール(※失敗)

<div class="md-code" style="width:100%">
```
pi@raspberrypi:~ $ sudo apt-get -y install libusb-dev libdbus-1-dev libglib2.0-dev libudev-dev libical-dev libreadline-dev
(省略)
E: Failed to fetch http://mirrordirector.raspbian.org/raspbian/pool/main/s/systemd/libudev-dev_215-17+deb8u3_armhf.deb  404  Not Found [IP: 5.153.225.207 80]
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```
</div>

　ラズパイにOSをインストールしただけでは、パッケージリストが古い状態らしい。  

　下記のコマンドで解決します。

#### APTのパッケージリストの更新

<div class="md-code" style="width:100%">
```
pi@raspberrypi:~ $ sudo apt-get update
```
</div>

　改めて依存パッケージのインストール

#### 再び必要パッケージのインストール

<div class="md-code" style="width:100%">
```
pi@raspberrypi:~ $ sudo apt-get -y install libusb-dev libdbus-1-dev libglib2.0-dev libudev-dev libical-dev libreadline-dev
```
</div>

　今度は成功しました。

### BlueZ − インストール

　BlueZの最新版は5系ですがbtproxyが動作しなかったので、4系のBlueZを使う。  
インストール作業は"/usr/src/"ディレクトリで行う。

<div class="md-code" style="width:100%">
```
pi@raspberrypi:~ $ cd /usr/src/
$ sudo wget http://www.kernel.org/pub/linux/bluetooth/bluez-4.101.tar.xz
$ sudo tar xvf bluez-4.101.tar.xz
$ cd bluez-4.101/
$ sudo ./configure --disable-systemd
$ sudo make
$ sudo make install
```
</div>

### btproxy − インストール

　btproxy はGitHub上で管理されており、インストールを非常に簡単に行えるようになっています。  

<div class="md-code" style="width:100%">
```
pi@raspberrypi:/usr/src $ sudo apt-get -y install python2.7-dev
$ sudo git clone https://github.com/conorpp/btproxy.git
$ cd btproxy
$ sudo python setup.py install
```
</div>

## btproxyを使う

### 動作確認

　ヘルプが表示されれば正常にインストールされています。

<div class="md-code" style="width:100%">
```
pi@raspberrypi:~ $ sudo btproxy
usage: btproxy [-h] [-a SET_ADDRESS] [-n] [-c] [-i INTERFACE] [-s SCRIPT] [-l]
               [-1 MASTER_NAME] [-2 SLAVE_NAME] [-C] [-v] [-z] [-q]
               [addr_master] [addr_slave]

Bluetooth MiTM Proxy. For analyzing bluetooth connections actively.

positional arguments:
  addr_master           Bluetooth address of target master device
  addr_slave            Bluetooth address of target slave device

optional arguments:
  -h, --help            show this help message and exit
  -a SET_ADDRESS, --set-address SET_ADDRESS
                        Address to set for Bluetooth adaptor (requires -i)
(省略)
  -q, --inquire-again   Inquire the services again, don't reuse saved
                        settings.
```
</div>

### 通信のキャプチャ

　まずはBluetooth機器のMACアドレスをスキャンします。  

　**MacとスマートフォンのBluetooth機能をONに設定しておきます。**  

[f:id:motikan2010:20170204194936p:plain:w400]  

　まずは、通信を行う同士のデバイスのMACアドレスを取得します。  
下記のコマンドの流れで、周囲にあるBluetooth機器のMACアドレスをスキャンします。

<div class="md-code" style="width:100%">
```
pi@raspberrypi:~ $ bluetoothctl
[NEW] Controller B8:27:EB:0A:55:1C raspberrypi [default]
[bluetooth]# scan on
Discovery started
[CHG] Controller B8:27:EB:0A:55:1C Discovering: yes
[NEW] Device 6C:76:60:8A:23:21 KCP01K
[NEW] Device B8:E8:56:2E:23:37 XXXXXX の MacBook Pro

[bluetooth]# quit // 終了
```
</div>

　２つのBluetooth機器のMACアドレスを取得できたので、２つのデバイスを中継する『btproxy』を起動させます。  

`$ btproxy <マスターデバイスのMACアドレス> <スレーブデバイスのMACアドレス>`

<div class="md-code" style="width:100%">
```
pi@raspberrypi:/usr/src/btproxy $ sudo btproxy B8:E8:56:2E:23:37 6C:76:60:8A:23:21
Running proxy on master  B8:E8:56:2E:23:37  and slave  6C:76:60:8A:23:21
Using shared adapter
Slave adapter:  hci0
Master adapter:  hci0
Looking up info on slave (6C:76:60:8A:23:21)
Looking up info on master (B8:E8:56:2E:23:37)
Spoofing master name as  KCP01K_btproxy
paired
Spoofing master name as  KCP01K_btproxy
Proxy listening for connections for "None"
Proxy listening for connections for "Headset Gateway"
Proxy listening for connections for "Handsfree Gateway"
Proxy listening for connections for "AV Remote Control Target"
Proxy listening for connections for "Advanced Audio"
Proxy listening for connections for "Android Network Access Point"
Proxy listening for connections for "MAP SMS/MMS"
Proxy listening for connections for "MAP EMAIL"
Proxy listening for connections for "OBEX Phonebook Access Server"
Proxy listening for connections for "OBEX Object Push"
Attempting connections with 10 services on slave
Now you're free to connect to "KCP01K_btproxy" from master device.
Connected to service "OBEX Object Push"
```
</div>

 　ペアリングの確認ダイアログが表示されるので許可します。  

・ノートPC側  
[f:id:motikan2010:20170409233455p:plain:w300][f:id:motikan2010:20170409233634p:plain:w300]  

・スマートフォン側  
[f:id:motikan2010:20170204194939p:plain:w300][f:id:motikan2010:20170204194941p:plain:w300]

##### ノートPC側からスマートフォン側へテキストファイルを送ってみます。  

　下記のファイルを送ってみます。  

`TEST.txt`

<div class="md-code" style="width:100%">
```
Hello btproxy!!
```
</div>

[f:id:motikan2010:20170204194954p:plain:w400]  

送信と同時にbtproxyによってキャプチャされた内容が表示されます。  

<div class="md-code" style="width:100%">
```
Accepted connection from  ('B8:E8:56:2E:23:37', 12)
<<  '\x80\x00\x07\x10\x00\x1f@'
>>  '\xa0\x00\x0c\x10\x00\xff\xfe\xcb\x00\x00\x00\x01'
<<  '\x82\x00/\x01\x00\x15\x00T\x00E\x00X\x00T\x00.\x00t\x00x\x00t\x00\x00\xc3\x00\x00\x00\x0fI\x00\x12Hello btproxy!!'
>>  '\xa0\x00\x0b\xcb\x00\x00\x00\x01I\x00\x03'
<<  '\x81\x00\x03'
>>  '\xa0\x00\x08\xcb\x00\x00\x00\x01'
(104, 'Connection reset by peer') socket slave reconnecting...
Reconnecting...
(104, 'Connection reset by peer') socket master reconnecting...
```
</div>

　「Hello btproxy!!」という文字が表示されておりキャプチャができていることを確認することができます。  

　`/libbtproxy/replace.py`ファイルを編集することによって、通信内容を改ざんすることも可能です。
