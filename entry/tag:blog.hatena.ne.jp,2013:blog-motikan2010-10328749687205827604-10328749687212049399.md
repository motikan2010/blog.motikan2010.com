<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　「"通信を発生させずにHTMLを解析"をしたい・・・」という要望があったので執筆。

　今までPHPでHTML解析するときは「Simple HTML DOM Parser」を利用しており、数年ぶりの利用で使い方を確認していると下記のような記事がありました。  
[http://localdisk.hatenablog.com/entry/2014/02/05/%E3%81%9D%E3%82%8D%E3%81%9D%E3%82%8D_Simple_HTML_DOM_Parser_%E3%82%92%E4%BD%BF%E3%81%86%E3%81%AE%E3%81%AF%E3%82%84%E3%82%81%E3%81%9F%E3%81%BB%E3%81%86%E3%81%8C%E3%81%84%E3%81%84:embed:cite]


「Simple HTML DOM Parser」はパフォーマンス面に優れておらず、今では「Goutte」を使った方がいいとか。  


[https://github.com/FriendsOfPHP/Goutte:embed:cite]

<!-- more -->


　いざ「Goutte」の使い方を簡単に調べてみると「Simple HTML DOM Parser」のようにHTML文字列を引数に与えて解析対象のオブジェクトを作成することができなさそう。    

- Simple HTML DOM Parser

```php
// Simple HTML DOM Parserを使った文字列からオブジェクトを生成
$html = str_get_html( '<html><body>Hello!</body></html>' );
```
- Goutte  
オブジェクトを生成するためにはリクエスト通信を発生させないといけなさそう。
```php
<?php
$client = new Client();
$crawler = $client->request('GET', 'https://www.symfony.com/blog/');
// 通信で取得したオブジェクトが解析対象になる
```

## 通信を発生させずにHTMLを解析

　Goutteでもファイル内に記載されているHTMLを解析できそう。

[http://ja.stackoverflow.com/questions/17366/goutte%E3%81%A7http%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%81%AA%E3%81%97%E3%81%AB%E6%96%87%E5%AD%97%E5%88%97%E3%81%8B%E3%82%89%E3%82%B9%E3%82%AF%E3%83%AC%E3%82%A4%E3%83%97%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95:embed:cite]


　下記のコードで通信を発生せずに引数に与えたHTML文字列を解析することができます。  
```php
<?php

use Symfony\Component\DomCrawler\Crawler;

$crawler = new Crawler(null);

// addHtmlContentの引数にHTML文字列
$crawler->addHtmlContent('<html><body>Hello!</body></html>');

echo $crawler->filter('html')->html();
//=> <body>Hello!</body>

echo $crawler->filter('body')->text();
//=> Hello!
```