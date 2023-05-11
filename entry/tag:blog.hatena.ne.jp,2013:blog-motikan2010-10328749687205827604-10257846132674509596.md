<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　最近、セキュリティ強化の為にTLS1.1以下の接続を不可にするようなサービスが増えてきています。  
勝手にTLS1.2で接続されるだろうとWevDAVクライアントを放置していたが、接続できないという事象が発生した。
 
## 検証用WebDAVサーバ

[https://github.com/motikan2010/WebDAV-Apache-Docker/tree/20181123:title]  

| | |
|-|-|
| 認証情報 | webdav / pass123 |
| ディレクトリ | /wevdav |

## エラー発生

　TLS1.2のみを許可しているWebDAVサーバをマウントしようとしたら以下のエラーが発生した。
```
$ sudo mount -t davfs https://127.0.0.1/webdav /media/127.0.0.1
/sbin/mount.davfs: Mounting failed.
SSL handshake failed: SSL alert received: Error in protocol version
```

## 原因

　gnutlsパッケージが古いのが原因。

[https://ja.wikipedia.org/wiki/GnuTLS:title]  

### 現在のバージョン確認

```
$ sudo yum list installed | grep gnutls
gnutls.x86_64                        2.8.5-4.el6_2.2               installed
```

## 解決策

　gnutlsパッケージをアップデートすればTLS1.2に対応される。

```
$ sudo yum -y update gnutls
```

### アップデート後のバージョン

```
$ sudo yum list installed | grep gnutls
gnutls.x86_64                        2.12.23-21.18.amzn1           @amzn-main
```

### 動作確認

　検証として利用しているWevDAVサーバの証明書は適当なものを使っているので、警告が出ているが問題なくマウントができていることが確認できる。
```
$ sudo mount -t davfs https://127.0.0.1/webdav /media/127.0.0.1
/sbin/mount.davfs: the server certificate does not match the server name
/sbin/mount.davfs: the server certificate is not trusted
  issuer:      XX, XX, XX, XX
  subject:     XX, XX, XX, XX
  identity:    XX
  fingerprint: 9b:dc:c1:34:b5:3c:68:10:c7:c5:6d:8f:fa:25:7a:51:46:70:22:ff
You only should accept this certificate, if you can
verify the fingerprint! The server might be faked
or there might be a man-in-the-middle-attack.
Accept certificate for this session? [y,N] y
/sbin/mount.davfs: Warning: can't write entry into mtab, but will mount the file system anyway

# マウントができている
$ df | grep webdav
https://127.0.0.1/webdav  26666664 13333332  13333332  50% /media/127.0.0.1
```

