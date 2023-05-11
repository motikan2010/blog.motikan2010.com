<div style="text-align:center;">[f:id:motikan2010:20180225073424p:plain:w500]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　今回は拡張プラグイン内からリクエストを送信し、レスポンス内容の表示までできるようにします。  

具体的には、プロキシ等でキャプチャしたIHttpRequestResponseクラス内に定義されているリクエストを再送します。

## 完成物 

[f:id:motikan2010:20180225073435p:plain]  

　機能としては、送信ボタンを押下すると、Enableにチェックが付いているリクエストが上から順に送信されるようになっています。  
　送信したリクエストと受信したレスポンスは、右部分のテキストエリアに表示されるようになっています。

<!-- more -->

## 開発

　前回の実装をベースに開発を進めていくため、今回はその差分を中心に説明をしていきます。  
列の追加、レイアウトの変更については特筆しません。


- <span><a href="https://github.com/motikan2010/Burp-Suite-Learning-Chapter03/compare/d0fda8f27e9863f49105fc9b75f182f2c4aad081...0621544ec03ca355b520b6e7f3da41d45c7ab618" target="_blank">「その３ - 新規UI(タブ)追加 編」との差分</a></span>


▼ 完成リポジトリ  
[https://github.com/motikan2010/Burp-Suite-Learning-Chapter03/tree/chapter4:embed:cite]  

##### RequestTableManager.java

　１つのメソッドを追加しています。
このメソッドを呼び出すことで、List<RequestResponseEntity>クラス内に定義されている、IHttpRequestResponseインスタンスを返すようになっています。

<div class="md-code" style="width:100%">
```java
public List<IHttpRequestResponse> getRequestResponseList() {
    List<RequestResponseEntity> requestResponseEntityList = this.model.getRequestResponseEntityList();
    return requestResponseEntityList.stream()
            .filter(RequestResponseEntity::isEnabled) // 列項目「Enable」にチェックが付いている行を対象
            .map(RequestResponseEntity::getRequestResponse).collect(Collectors.toList());
}
```
</div>

##### SampleTab.java

　下記のメソッドを使用することで、HTTP通信を行うことができます。  
第１引数の`IHttpService`のインスタンスは、`IHttpRequestResponse.getHttpService()`で取得することができます。  
　`IHttpRequestResponse`のインスタンスは、前回の章で独自クラスである`RequestResponseEntity`クラス内に保存しているため、そちらを使用しています。  

<div class="md-code" style="width:100%">
```
IHttpRequestResponse makeHttpRequest(IHttpService httpService, byte[] request)
```
</div>

<a href="https://portswigger.net/burp/extender/api/burp/IBurpExtenderCallbacks.html#makeHttpRequest(burp.IHttpService,%20byte[])" target="_blank">IBurpExtenderCallbacks</a>

<div class="md-code" style="width:100%">
```java
// ◆「送信」ボタン押下時に発火
ExecutorService executor = Executors.newSingleThreadExecutor();
sendButton.addActionListener(e -> executor.submit(() ->
        // requestResponseEntityList(テーブル内のデータ)に格納されているリクエストを一行ずつ送信
        requestTableManager.getRequestResponseList().stream().forEach(iHttpRequestResponse -> {
            // リクエストをテキストエリアに追加
            requestResponseTextArea.append(requestResponseUtils.showRequest(iHttpRequestResponse) // リクエスト情報
                    + requestResponseUtils.getNewLine() + requestResponseUtils.getNewLine());

            // リクエストの送信
            IHttpRequestResponse response = BurpExtender.getCallbacks().makeHttpRequest(iHttpRequestResponse.getHttpService(), iHttpRequestResponse.getRequest());

            // レスポンスをテキストエリアに追加
            requestResponseTextArea.append(requestResponseUtils.showResponse(response) // レスポンス情報
                    + requestResponseUtils.getNewLine() + requestResponseUtils.getNewLine() + requestResponseUtils.getNewLine()); 
        })
));
```
</div>

　取得したレスポンス情報は、ウィンドウの右部分のテキストエリアに追記しています。

## まとめ

　今回は差分ということもあり、内容が短くなってしまいましたが、IHttpRequestResponseのインスタンスがあれば簡単に同一のリクエストを送信することができることが分かりました。  

　次回は、同一のリクエストではなく、リクエスト内容を改変して送信する方法を説明してきます。

## 更新履歴

- 20年2月25日 新規作成
