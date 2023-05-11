<div style="text-align:center;">
<figure class="figure-image figure-image-fotolife" title="作成したアプリのスクリーンショット">[f:id:motikan2010:20200715205641p:plain:w500]<figcaption>作成したアプリのスクリーンショット</figcaption></figure>
</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　最近、「MD-to-PDF(仮名)」というWebアプリケーションを開発してみたので、<span class="m-y">利用したライブラリといった技術などを紹介</span>します。  
(まだ荒い完成度ですが、動作するので紹介)

　このアプリケーションを一言で説明すると、<span class="m-y">GitHub上のmd(Markdown)ファイルをPDFに変換</span>するWebアプリケーションです。  
認証なしでも利用可能ですが、<span class="m-y">GitHub認証をすることでプライベートリポジトリ内のMDファイルを変換対象</span>にすることができます。

<div style="text-align:center;">
<figure class="figure-image figure-image-fotolife" title="MDファイルから変換されたPDFファイル">[f:id:motikan2010:20200716001823p:plain:w500]<figcaption>MDファイルから変換されたPDFファイル</figcaption></figure>
</div>

　 開発のモチベーションとしては、実際にGitHub上のファイルをPDFとして取得したかったというものがあります。  
ローカル上のファイルを変換するアプリケーションは存在していましたが、ファイルをローカルに移動させるのが手間であったため、GitHub上のファイルを変換するものを開発しました。 

また、GitHub上のファイルを利用するようなアプリケーションを開発したいというものがありました。  

　下記サイトがそのアプリケーションです。  
[MD to PDF](https://md-to-pdf.motikan2010.com/)

### 開発環境

- PHP 7.3
- Laravel 6.18
- Vue.js 2.6.11

　ソースコードはこちらです。  
[https://github.com/motikan2010/MD-to-PDF/tree/v0.1.0:embed:cite]

## 利用技術

### MDをPDFに変換

　MDファイルをPDFファイルに変換する処理はこのアプリケーションのコア部分ですが、主に2つのライブラリを利用して実現しています。  

MDファイルからHTMLへの変換には`erusev/parsedown`。HTMLからPDFへの変換は`dompdf/dompdf`を用いています。

　具体的なMD形式からPDFファイルへの変換の具体的なコードは以下のような感じです。  
<div class="sm-code" style="width:60%">
```php
<?php

require '../vendor/autoload.php';

$markdownText = <<<EOLL
## Hello World!!

- sample1
- sample2
- sample3

## こんにちは、世界！！

- サンプル1
- サンプル3
- サンプル3
EOLL;

// Markdown を HTML に変換
$parsedown = new \Parsedown();
$html = $parsedown->text($markdownText);

// HTML を PDF に変換
$domPdf = new \Dompdf\Dompdf();
$domPdf->loadHtml($html);
$domPdf->render();
echo $domPdf->output();
```
</div>

<div class="sm-code" style="width:60%">
```
$ php main.php > test.php
```
</div>

　以下が生成されたPDFファイルです。  
MD形式からPDFファイルに変換できていることが確認できます。  
<div style="text-align:center;">
<figure class="figure-image figure-image-fotolife" title="出力されたPDFファイル">[f:id:motikan2010:20200715223753p:plain:w500]<figcaption>出力されたPDFファイル</figcaption></figure>
</div>

変換されたことは確認できましたが、<span class="m-y">日本語表記の部分が「????」と文字化け</span>を起こしています。  
日本語に対応させるために、なんらかの作業が必要そうです。  

### 日本語の対応(dompdfの日本語対応)

　PDFファイルは無事出力されましたが、日本語表記の部分が文字化けが発生しています。   
この問題を解消するには<span class="m-y">日本語フォントを取得し、dompdfに設定</span>する必要があります。    

　dompdfの設定には`dompdf/utils`を使います。  
[https://github.com/dompdf/utils:embed:cite]

　フォント設定の流れは以下の通りです。IPAが配布しているIPAフォントを利用させていただきます。  
<div class="sm-code" style="width:60%">
```
// IPAフォントのダウンロード
$ wget https://ipafont.ipa.go.jp/IPAfont/ipag00303.zip
$ unzip ipag00303.zip

// 「dompdf/utils」のダウンロードと解答
$ https://github.com/dompdf/utils/archive/master.zip
$ unzip master.zip

// IPAフォントをdompdfに適用
$ php utils-master/load_font.php ipag ipag00303/ipag.ttf

// 後片付け
$ rm -rf ipag00303.zip master.zip utils-master ipag00303
```
</div>

　IPAフォントをdompdfに適用できましたが、そのままではIPAフォントは利用されません。  
PDFファイルに変換するHTML内に利用するフォントの設定(`font-family: ipag;`)をする必要があります。

　以下が設定例です。  
<div class="sm-code" style="width:100%">
```php
$html = '<html><style>body {font-family: ipag;}</style><body>' . $html . '</body></html>';
```
</div>

　ちなみに、本アプリケーションではPDFファイルに変換するHTMLはBladeテンプレートファイルに記述しています。  
以下がその部分です。
[https://github.com/motikan2010/MD-to-PDF/blob/v0.1.0/resources/views/pdf/body.blade.php:title]


## 課題

　アプリケーションは公開してみましたが、まだバージョンは 0.1.0 という位置付けとしています。現時点で見つかっている課題が多数あるからです。  
例えば、

- デザインが悪い。環境によってはヘッダーが消えたり。
- 生成されるPDFファイルのフォーマットが悪い。太文字や斜文字が使えなかったり。
- GitHub用のMD形式に未対応。

があります。  

　このアプリケーションの重要部分は完成し、当初の目的であるMDファイルをPDFファイルとして保存というのが達成されたので、開発モチベが少し下がっていますが時間があるときにでも改修していこうと考えています。

## まとめ

　適当に使った技術を一部まとめてみました。  
フロント(Vue.js)については別の機会に書いてみようかなと思った次第。ファイルがツリー上で表示される部分はお気に入りだったり。

## 更新履歴

- 2020年7月15日 新規作成
