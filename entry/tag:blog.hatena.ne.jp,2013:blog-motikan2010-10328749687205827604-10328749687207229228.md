　<span style="color: #ff0000">**※本記事で紹介する hey は  2020年8月6日のリリースからアップデートされていません。**</span>

<div style="text-align: center;">[f:id:motikan2010:20170117010150p:plain:w400]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>


## はじめに

　Web界隈では「Apache Bench」「JMeter」などのベンチマークツールが有名ですが、本記事ではGo言語で開発されたベンチマークツール『<span class="m-y">hey</span>』を紹介します。

[https://github.com/rakyll/hey:embed:cite]

　スターは約7,500個で人気なソフトウェアになっているのは確かです。

## 百聞は一見に如かず「hey」を使う

　結果は下記ような形式で出力されます。  
abコマンドと比べて非常にシンプルな結果表示になっています。   

　このツールの実装自体もシンプルで可読性が高く(さすがはGo)、カスタマイズしても良し、このツールを参考に自分自身でベンチマークツールを作成するのにも参考になるかと思います。
<div class="sm-code">
```
# ./hey -n 200 -c 50 http://127.0.0.1:3000/login
2 requests done.
10 requests done.
15 requests done.
21 requests done.
(省略)
192 requests done.
197 requests done.
All requests done.

Summary:
  Total:	19.2161 secs
  Slowest:	18.6380 secs
  Fastest:	0.1297 secs
  Average:	2.6002 secs
  Requests/sec:	10.4080
  Total data:	180532 bytes
  Size/request:	902 bytes

Status code distribution:
  [200]	199 responses
  [500]	1 responses

Response time histogram:
  0.130 [1]    |
  1.981 [155]  |∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎
  3.831 [5]    |∎
  5.682 [5]    |∎
  7.533 [5]    |∎
  9.384 [5]    |∎
  11.235 [4]   |∎
  13.086 [5]   |∎
  14.936 [6]   |∎∎
  16.787 [4]   |∎
  18.638 [5]   |∎

Latency distribution:
  10% in 0.3156 secs
  25% in 0.3908 secs
  50% in 0.4547 secs
  75% in 0.6077 secs
  90% in 11.2754 secs
  95% in 14.9333 secs
  99% in 18.3849 secs
```
</div>

## インストール

　Go言語が使える状態から始めます。<span class="m-y">動作するバージョンは**1.7以上**です。</span>
```
$ go version
go version go1.7.4 linux/amd64
```

インストールは非常に簡単です。下記のコマンドを実行するだけです。
```
$ go install github.com/rakyll/hey
```
　これでエラーが表示されなければ**hey**コマンドが使用できるようになります。  

　どのようなオプションが用意されているのかを確認してみます。
```
$ hey
Usage: hey [options...] <url>

Options:
  -n  Number of requests to run. Default is 200.
  -c  Number of requests to run concurrently. Total number of requests cannot
      be smaller than the concurrency level. Default is 50.
  -q  Rate limit, in seconds (QPS).
  -o  Output type. If none provided, a summary is printed.
      "csv" is the only supported alternative. Dumps the response
      metrics in comma-separated values format.

  -m  HTTP method, one of GET, POST, PUT, DELETE, HEAD, OPTIONS.
  -H  Custom HTTP header. You can specify as many as needed by repeating the flag.
      For example, -H "Accept: text/html" -H "Content-Type: application/xml" .
  -t  Timeout for each request in seconds. Default is 20, use 0 for infinite.
  -A  HTTP Accept header.
  -d  HTTP request body.
  -D  HTTP request body from file. For example, /home/user/file.txt or ./file.txt.
  -T  Content-type, defaults to "text/html".
  -a  Basic authentication, username:password.
  -x  HTTP Proxy address as host:port.
  -h2 Enable HTTP/2.

  -host	HTTP Host header.

  -disable-compression  Disable compression.
  -disable-keepalive    Disable keep-alive, prevents re-use of TCP
                        connections between different HTTP requests.
  -cpus                 Number of used cpu cores.
                        (default for current machine is 2 cores)
  -more                 Provides information on DNS lookup, dialup, request and
                        response timings.
```

## 動作確認(オプションを指定)

### GETリクエストを送信

　特にオプションを設定しない場合、GETリクエストを送信します。  
下記のように実行します。
```
$ hey http://127.0.0.1:3000/
```

### POSTリクエストを送信
- 「`-m POST`」でHTTPメソッドをPOSTに指定。
- 「`-H ヘッダ名:ヘッダ値`」でリクエストヘッダの指定。  
下記の例では「Content-Type: application/x-www-form-urlencoded」をリクエストヘッダに付与。
- `-d`オプションでPOSTパラメータを指定

```
$ hey -m POST -H "Content-Type: application/x-www-form-urlencoded" \
    -d "email=example@example.com&password=foobar" http://127.0.0.1:3000/login
```

### Basic認証に対応

　「`-a [ユーザ名]:[パスワード]`」をオプションで指定します。  
ダブルクォーテーションはあってもなくても動作します。
```
$ hey -a "hogeuser:hogepassword" http://127.0.0.1/
```

## まとめ

　最近Go言語にハマってしまいGo言語製のツールを探してみてこのツールにたどり着きましたが、やはりベンチーマークツールは楽しいですね。なによりズラズラと流れるアプリケーションログを眺めるのが非常に快感です。  
　しかし『hey』のオプションはまだまだ使え切れていないです。（今回は２つしか使っていない）  
キープアライブやHTTP/2の知識を入れて是非使いこなしてみたいツールです。

## 更新履歴

- 2017年 1月17日 新規作成
- 2020年 2月 3日 目次追加
