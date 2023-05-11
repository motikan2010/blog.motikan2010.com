<div style="text-align: center;">
[f:id:motikan2010:20191120223853p:plain]
</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　Selenium側からページが読み込むリソース情報(URL)を取得する方法を紹介します。  

　言語はPythonを使います。  

## 動作環境

- Python 3.6.9
- Selenium 3.141.0
- Google Chrome 78.0

### 取得対象

　今回の検証に利用するサンプルページのHTMLは以下のものです。  

　このページが開かれたら6つのリクエストが飛ぶようになっています。  
　(JSファイル2つ、画像ファイル1つ、Ajaxのリクエスト2つ、ファビコン1つ)  

　それらのリソースのURLをSelenium側(Python側)から取得することが本検証の目的となります。

<div class="md-code" style="width:100%">
```html
<html>
<body>
  <img src="https://dummyimage.com/600x400/000/fff.png" />
</body>
<script src="./lib/jquery-3.3.1.min.js"></script>
<script src="./lib/bootstrap.min.js"></script>
<script type="text/javascript">
  $.ajax({type: "GET", url: "ajax-get.php"});
  $.ajax({type: "POST",url: "ajax-post.php", data: "name=taro"});
</script>
</html>
```
</div>

## 実装

### パフォーマンス情報の取得

　リクエストやレスポンス等のネットワーク情報はパフォーマンス情報内に含まれています。  
まず初めにパフォーマンス情報を取得してみます。  

<div class="md-code" style="width:100%">
```python
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

url = 'http://127.0.0.1:8080/index.html'
d = DesiredCapabilities.CHROME
d['goog:loggingPrefs'] = { 'performance': 'ALL' }
driver = webdriver.Chrome(desired_capabilities=d)
driver.get(url)

for entry_json in driver.get_log('performance'):
    print(entry_json)

driver.close()
```
</div>

-  パフォーマンス情報の取得結果

　<span class="m-y">パフォーマンス情報はイベント毎に出力されるようになっており、リクエスト情報以外のイベント情報も出力されるようになっています。  </span>

　ここではURLを取得したいので、次はリクエスト情報のみ出力するようにフィルタ処理を追加します。 

<div class="md-code" style="width:100%"> 
```
{'level': 'INFO', 'message': '{"message":{"method":"Network.loadingFinished","params":{"encodedDataLength":0,"requestId":"DA7DC2455827A385EFD29AAD84A5FAD4","shouldReportCorbBlocking":false,"timestamp":31590.809391}},"webview":"7C67A40D1B7E099220778917F0FC37BF"}', 'timestamp': 1574243695252}
{'level': 'INFO', 'message': '{"message":{"method":"Page.loadEventFired","params":{"timestamp":31590.84658}},"webview":"7C67A40D1B7E099220778917F0FC37BF"}', 'timestamp': 1574243695252}
{'level': 'INFO', 'message': '{"message":{"method":"Page.frameStoppedLoading","params":{"frameId":"7C67A40D1B7E099220778917F0FC37BF"}},"webview":"7C67A40D1B7E099220778917F0FC37BF"}', 'timestamp': 1574243695253}
{'level': 'INFO', 'message': '{"message":{"method":"Page.domContentEventFired","params":{"timestamp":31590.847009}},"webview":"7C67A40D1B7E099220778917F0FC37BF"}', 'timestamp': 1574243695253}
{'level': 'INFO', 'message': '{"message":{"method":"Page.frameResized","params":{}},"webview":"7C67A40D1B7E099220778917F0FC37BF"}', 'timestamp': 1574243695253}
{'level': 'INFO', 'message': '{"message":{"method":"Network.requestWillBeSent","params":{"documentURL":"http://127.0.0.1:8080/index.html","frameId":"7C67A40D1B7E099220778917F0FC37BF","hasUserGesture":false,"initiator":{"type":"other"},"loaderId":"4BC43963C4D0AA21F19857E6EB4160C8","request":{"headers":{"Sec-Fetch-User":"?1","Upgrade-Insecure-Requests":"1","User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"},"initialPriority":"VeryHigh","method":"GET","mixedContentType":"none","referrerPolicy":"no-referrer-when-downgrade","url":"http://127.0.0.1:8080/index.html"},"requestId":"4BC43963C4D0AA21F19857E6EB4160C8","timestamp":31590.884601,"type":"Document","wallTime":1574243695.273999}},"webview":"7C67A40D1B7E099220778917F0FC37BF"}', 'timestamp': 1574243695274}
以下略...
```
</div>

　フィルタ条件を調べる為にリクエスト情報に着目します。一番下の出力がリクエスト情報です。  

　その情報を整形したのが下記のものです。「`method`」値が「`Network.requestWillBeSent`」になっているものをフィルタすれば良さそうです。  

<div class="md-code" style="width:100%">
```json
{
  "message": {
    "method": "Network.requestWillBeSent", ← この値がフィルタに使えそう
    "params": {
      "documentURL": "http://127.0.0.1:8080/index.html",
      "frameId": "7C67A40D1B7E099220778917F0FC37BF",
      "hasUserGesture": false,
      "initiator": {
        "type": "other"
      },
      "loaderId": "4BC43963C4D0AA21F19857E6EB4160C8",
      "request": {
        "headers": {
          "Sec-Fetch-User": "?1",
          "Upgrade-Insecure-Requests": "1",
          "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"
        },
        "initialPriority": "VeryHigh",
        "method": "GET",
        "mixedContentType": "none",
        "referrerPolicy": "no-referrer-when-downgrade",
        "url": "http://127.0.0.1:8080/index.html"
      },
      "requestId": "4BC43963C4D0AA21F19857E6EB4160C8",
      "timestamp": 31590.884601,
      "type": "Document",
      "wallTime": 1574243695.273999
    }
  },
  "webview": "7C67A40D1B7E099220778917F0FC37BF"
}
```
</div>

### ブラウザが読み込むリソースのURLを取得

　URLのみを表示するフィルタを追加したコードが下記になります。  
for内部を以下のようにすることでリクエストURLを取得することが可能です。

<div class="md-code" style="width:100%">
```python
import json

# ...

for entry_json in driver.get_log('performance'):
    entry = json.loads(entry_json['message'])
    if entry['message']['method'] != 'Network.requestWillBeSent' :
        continue
    print(entry['message']['params']['request']['url'])
```
</div>

　実行結果です。（先頭のページ表示時のURLも表示されていますが、許容できる範囲でしょう）
<div class="md-code" style="width:100%">
```
http://127.0.0.1:8080/index.html
https://dummyimage.com/600x400/000/fff.png
http://127.0.0.1:8080/lib/jquery-3.3.1.min.js
http://127.0.0.1:8080/lib/bootstrap.min.js
http://127.0.0.1:8080/ajax-get.php
http://127.0.0.1:8080/ajax-post.php
http://127.0.0.1:8080/favicon.ico
```
</div>

　リクエスト時の`Network.requestWillBeSent`のようなイベントは他にもあり以下のコードで一覧が記載されています。  
取得したい情報に応じてフィルタを掛けることが可能です。  
[https://github.com/ChromeDevTools/devtools-protocol/blob/241adc5c42bad68b28a1c07ac90a60a107d1baf2/types/protocol-mapping.d.ts#L259-L262:embed:cite]

## まとめ

　Seleniumはテストの自動化といった場面で多く活用されていますが、本記事では大量のJavaScriptファイルの通信を解析する場面を想定したSeleniumの使い方でした。  
難読化されたJavaScriptの挙動などを軽く確認したい時とかに活用できそうです。  

　実際に私も難読化された大量のJavaScriptファイルを動的解析する必要があり、本記事で紹介した方法を試みました。  

　他にも解析方法があったらSelenium問わずやってみたいです。

## 更新履歴

- 2019年 11月 19日 新規作成


[blog:g:11696248318754550880:banner][blog:g:12921228815726579926:banner]
