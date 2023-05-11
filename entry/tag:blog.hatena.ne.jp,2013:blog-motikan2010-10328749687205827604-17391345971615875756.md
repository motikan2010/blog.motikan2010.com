<div style="text-align:center;">[f:id:motikan2010:20180213000116j:plain:w500]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　前回に引き続き、今回はBurp Suiteの機能を活かした拡張プラグインを作成していきます。  

[http://blog.motikan2010.com/entry/2018/02/11/Burp_Suite%E3%81%AE%E6%8B%A1%E5%BC%B5%E3%82%92%E4%BD%9C%E6%88%90%E5%85%A5%E9%96%80\_%E3%81%9D%E3%81%AE%EF%BC%91\_-\_Hello\_world%E7%B7%A8:embed:cite]

### 完成のイメージ

　今回作成するもののイメージを持っていただくため、完成品を先に見せます。  

- `HTTP history`のコンテキストメニューに項目を追加します。
[f:id:motikan2010:20180213000221p:plain:w500]  

　指定したHTTPログのリクエスト&レスポンスの内容を表示するようになっています。  
[f:id:motikan2010:20180213000237p:plain:w500]  

[f:id:motikan2010:20180213000248p:plain:w500]

　まだ実用するには程遠いですが、Burp Suiteの特徴的な機能である  「リクエスト&レスポンス」の情報にアクセスすることが可能になります。  

<!-- more -->

### 学ぶこと

　主に２つあります。  

- コンテキストメニュー（右クリックで表示される項目）への追加
- リクエスト&レスポンス情報の取得(「ヘッダ」と「ボディ」へのアクセス)

## 実装

### ディレクトリ構造

　最終的なディレクトリ構造は下記のようになります。

<div class="md-code" style="width:100%">
```
.
├── pom.xml
└── src
    └── main
        └── java
            ├── burp
            │   └── BurpExtender.java
            └── com
                └── motikan2010
                    ├── RequestContextMenu.java
                    ├── ResponseContextMenu.java
                    └── util
                        └── RequestResponseUtils.java
```
</div>

　pom.xmlファイルは前回と変更はありません。

　各ファイルの役割を簡単に説明すると下記のようになります。  

| | |
| - | - |
| BurpExtender.java | ・ Burp Suiteからリクエスト&レスポンスを取得<br/> ・ コンテキストメニューに表示する項目名を設定 |
| RequestContextMenu.java | ・`[New Menu] Show Request` がクリックされた際のイベントの設定 |
| ResponseContextMenu.java |  ・`[New Menu] Show Response` がクリックされた際のイベントの設定 |
| RequestResponseUtils.java | ・ リクエスト&レスポンスの情報を受け取り、テキストとして出力するために整形 |

### 各機能の実装

##### BurpExtender.java

　リクエスト&レスポンスの情報は <span><a href="https://portswigger.net/burp/extender/api/burp/IHttpRequestResponse.html" target="_blank">IHttpRequestResponseクラス</a></span> に格納されます。  
　このIHttpRequestResponseクラスにアクセスすることで、ヘッダー情報やボディ情報を取得することができます。

　コンテキストメニューを設定するために、 <span><a href="https://portswigger.net/burp/extender/api/burp/IContextMenuFactory.html" target="_blank">IContextMenuFactoryインタフェース</a></span> を実装する必要があります。  
　IContextMenuFactoryインタフェースには、`createMenuItemsメソッド` が定義されており、その中にコンテキストメニューの項目名やマウス・リスナーの設定を行います。  

　下記のコードでは、以下の２項目が表示されるように設定しています。  

- [New Menu] Show Request :  (選択数)
- [New Menu] Show Response : (選択数)

<div class="md-code" style="width:100%">
```java
package burp;

import com.motikan2010.RequestContextMenu;
import com.motikan2010.ResponseContextMenu;

import javax.swing.*;
import java.util.LinkedList;
import java.util.List;

public class BurpExtender implements IBurpExtender, IContextMenuFactory { // IContextMenuFactoryを実装する

    private IBurpExtenderCallbacks iBurpExtenderCallbacks;

    public void registerExtenderCallbacks(IBurpExtenderCallbacks callbacks) {
        this.iBurpExtenderCallbacks = callbacks;
        this.iBurpExtenderCallbacks.setExtensionName("Context Menu Sample");

        // コンテキストメニューを登録するために追記
        this.iBurpExtenderCallbacks.registerContextMenuFactory(this);
    }

    /**
     * コンテキストメニューの作成
     *
     * @param iContextMenuInvocation
     * @return
     */
    public List<JMenuItem> createMenuItems(IContextMenuInvocation iContextMenuInvocation) {
        /*
         * リクエストを選択した状態で、コンテキストメニューがクリックされた際に、取得される情報は「IHttpRequestResponse」クラスに格納されます
         * リクエストを複数選択した状態で、コンテキストメニュークリックすることも可能であるため、配列で取得されます。
         */
        IHttpRequestResponse[] httpRequestResponseArray = iContextMenuInvocation.getSelectedMessages();

        if (null == httpRequestResponseArray) {
            return null;
        }

        List<JMenuItem> jMenuItemList = new LinkedList<>();

        // リクエスト表示
        // コンテキストに表示するテキスト
        JMenuItem requestJMenuItem = new JMenuItem("[New Menu] Show Request : " + httpRequestResponseArray.length);
        // 右クリック時の動作を設定
        requestJMenuItem.addMouseListener(new RequestContextMenu(this.iBurpExtenderCallbacks, httpRequestResponseArray));
        // コンテキストを追加
        jMenuItemList.add(requestJMenuItem);

        // レスポンス表示
        JMenuItem responseJMenuItem = new JMenuItem("[New Menu] Show Response : " + httpRequestResponseArray.length);
        responseJMenuItem.addMouseListener(new ResponseContextMenu(this.iBurpExtenderCallbacks, httpRequestResponseArray));
        jMenuItemList.add(responseJMenuItem);

        return jMenuItemList;
    }
}
```
</div>

##### RequestContextMenu.java  

　コンテキストメニューの「[New Menu] Show Request」をクリックした際のイベントを設定しています。  
　マウスのクリックイベントを設定するために <b>MouseListenerインタフェース</b>を実装しています。今回は、マウスを離した時に発火させるので、<b>mouseReleasedメソッド</b>内に処理内容を記述しています。  
処理内容は下記の通りです。

* 取得したIHttpRequestResponseのインスタンスを独自に作成したshowRequestメソッドに渡している。(showRequestメソッドはリクエスト内容を文字列で返却します)
* リクエスト文字列を出力

<div class="md-code" style="width:100%">
```java
package com.motikan2010;

import burp.IBurpExtenderCallbacks;
import burp.IHttpRequestResponse;
import com.motikan2010.util.RequestResponseUtils;

import java.awt.event.MouseEvent;
import java.awt.event.MouseListener;
import java.io.PrintWriter;

public class RequestContextMenu implements MouseListener {

    private IBurpExtenderCallbacks iBurpExtenderCallbacks;
    private IHttpRequestResponse[] iHttpRequestResponseArray;
    private RequestResponseUtils requestResponseUtils;

    public RequestContextMenu(IBurpExtenderCallbacks callbacks, IHttpRequestResponse[] requestResponseArray) {
        this.iBurpExtenderCallbacks = callbacks;
        this.iHttpRequestResponseArray = requestResponseArray;
        this.requestResponseUtils = new RequestResponseUtils(callbacks);
    }

    /**
     * マウスボタンを離すと呼び出される
     *
     * @param event イベント
     */
    @Override
    public void mouseReleased(MouseEvent event) {
        PrintWriter stdout = new PrintWriter(iBurpExtenderCallbacks.getStdout(), true);
        for (IHttpRequestResponse iHttpRequestResponse : this.iHttpRequestResponseArray) {
            // リクエストを取得
            String requestString = requestResponseUtils.showRequest(iHttpRequestResponse);

            // リクエストを出力
            stdout.println(requestString);
        }
    }

    public void mouseClicked(MouseEvent e) {}
    public void mousePressed(MouseEvent e) {}
    public void mouseEntered(MouseEvent e) {}
    public void mouseExited(MouseEvent e) {}
}
```
</div>

##### ResponseContextMenu.java

　コンテキストメニューの「[New Menu] Show Response」をクリックした際のイベントを設定しています。  
　マウスのクリックイベントを設定するために <b>MouseListenerインタフェース</b>を実装しています。今回は、マウスを離した時に発火させるので、<b>mouseReleasedメソッド</b>内に処理内容を記述しています。  
処理内容は下記の通りです。

* 取得したIHttpRequestResponseのインスタンスを独自に作成したshowResponseメソッドに渡している。(showResponseメソッドはレスポンス内容を文字列で返却します)
* レスポンス文字列を出力

<div class="md-code" style="width:100%">
```java
package com.motikan2010;

import burp.IBurpExtenderCallbacks;
import burp.IHttpRequestResponse;
import com.motikan2010.util.RequestResponseUtils;

import java.awt.event.MouseEvent;
import java.awt.event.MouseListener;
import java.io.PrintWriter;

public class ResponseContextMenu implements MouseListener {

    private IBurpExtenderCallbacks iBurpExtenderCallbacks;
    private IHttpRequestResponse[] iHttpRequestResponseArray;

    private RequestResponseUtils requestResponseUtils;

    public ResponseContextMenu(IBurpExtenderCallbacks callbacks, IHttpRequestResponse[] requestResponseArray) {
        this.iBurpExtenderCallbacks = callbacks;
        this.iHttpRequestResponseArray = requestResponseArray;
        this.requestResponseUtils = new RequestResponseUtils(callbacks);
    }

    /**
     * マウスボタンを離すと呼び出される
     *
     * @param event イベント
     */
    @Override
    public void mouseReleased(MouseEvent event) {
        PrintWriter stdout = new PrintWriter(iBurpExtenderCallbacks.getStdout(), true);
        for (IHttpRequestResponse iHttpRequestResponse : this.iHttpRequestResponseArray) {
            // レスポンスの取得
            String requestStr = requestResponseUtils.showResponse(iHttpRequestResponse);

            // レスポンスの出力
            stdout.println(requestStr);
        }
    }

    public void mouseClicked(MouseEvent e) {}
    public void mousePressed(MouseEvent e) {}
    public void mouseEntered(MouseEvent e) {}
    public void mouseExited(MouseEvent e) {}
}
```
</div>

##### RequestResponseUtils.java

　RequestResponseUtilsクラスの役割は、主にIHttpRequestResponseのインスタンスを受け取り、リクエスト・レスポンスを文字列形式で返す、ユーティリティ群です。  
　IHttpRequestResponseクラスの解析が主な処理となっており、重要な処理を担っています。  

ここで重要な点は下記になります。  

* <b>IExtensionHelpers クラス</b>
  * IHttpRequestResponse クラスから「IRequestInfo」「IResponseInfo」のインスタンスを取得する際に利用します
  * IBurpExtenderCallbacks.<b><span style="color: #ff0000">getHelpers()</span></b>で取得することができます
* <b>IRequestInfo クラス</b>
  * HTTPリクエストに関する情報が定義されています
  * IExtensionHelpers.<b><span style="color: #ff0000">analyzeRequest</span></b>(IHttpRequestResponse)
* <b>IResponseInfo クラス</b>
  * HTTPレスポンスに関する情報が定義されています
  * IExtensionHelpers.<b><span style="color: #ff0000">analyzeResponse</span></b>(IHttpRequestResponse.<b>getResponse()</b>)

<div class="md-code" style="width:100%">
```java
package com.motikan2010.util;

import burp.*;

import java.io.UnsupportedEncodingException;
import java.util.List;

public class RequestResponseUtils {

    private static final String NEW_LINE = System.lineSeparator();

    private static IBurpExtenderCallbacks iBurpExtenderCallbacks;
    private static IExtensionHelpers iExtensionHelpers;

    public RequestResponseUtils(IBurpExtenderCallbacks callbacks) {
        iBurpExtenderCallbacks = callbacks;
        iExtensionHelpers = callbacks.getHelpers();
    }

    /**
     * リクエスト情報を取得
     *
     * @param iHttpRequestResponse
     * @return String
     */
    public String showRequest(IHttpRequestResponse iHttpRequestResponse) {
        StringBuilder stringBuilder = new StringBuilder();

        // リクエスト情報を取得
        IRequestInfo iRequestInfo = iExtensionHelpers.analyzeRequest(iHttpRequestResponse);

        // リクエストヘッダ情報を取得
        List<String> headers =iRequestInfo.getHeaders();
        stringBuilder.append(this.createHeaderRaw(headers));

        // リクエストボディ情報を取得
        byte[] requestBytes = iHttpRequestResponse.getRequest();
        stringBuilder.append(this.createBodyRaw(requestBytes));

        return stringBuilder.toString();
    }

    /**
     * レスポンス情報を取得
     *
     * @param iHttpRequestResponse
     * @return String
     */
    public String showResponse(IHttpRequestResponse iHttpRequestResponse) {
        StringBuilder stringBuilder = new StringBuilder();

        // レスポンス情報を取得
        IResponseInfo iResponseInfo = iExtensionHelpers.analyzeResponse(iHttpRequestResponse.getResponse());

        // レスポンスヘッダ情報を取得
        List<String> headers = iResponseInfo.getHeaders();
        stringBuilder.append(this.createHeaderRaw(headers));

        // レスポンスボディ情報を取得
        byte[] responseBytes = iHttpRequestResponse.getResponse();
        stringBuilder.append(this.createBodyRaw(responseBytes));

        return stringBuilder.toString();
    }

    /**
     * ヘッダーリストを文字列に変換
     *
     * @param headers
     * @return
     */
    private String createHeaderRaw(List<String> headers) {
        StringBuilder stringBuilder = new StringBuilder();
        // リクエストヘッダ情報を取得
        for (String header : headers) {
            stringBuilder.append(header);
            stringBuilder.append(NEW_LINE);
        }
        return stringBuilder.toString();
    }

    /**
     * ボディのバイト配列を文字列に変換
     *
     * @param bodyBytes
     * @return
     */
    private String createBodyRaw(byte[] bodyBytes) {
        String response = "";
        try {
            response = new String(bodyBytes, "UTF-8");
            response = response.substring(iExtensionHelpers.analyzeResponse(bodyBytes).getBodyOffset());
        } catch (UnsupportedEncodingException e) {
            System.out.println("Error converting string");
        }

        if (response.length() > 0) {
            response = NEW_LINE + response;
            return response;
        } else {
            return "";
        }
    }
}
```
</div>

　このプロジェクトをビルド後、Burp Suiteにロードすることにより、コンテキストメニューが表示されるようになります。

　今回のプロジェクトのリポジトリはこちらです。  
[https://github.com/motikan2010/Burp-Suite-Learning-Chapter02:title]


　次回はタブに新規画面を追加する拡張プラグインを作成していきます。

[https://blog.motikan2010.com/entry/2018/02/18/Burp_Suite%E6%8B%A1%E5%BC%B5%E3%83%97%E3%83%A9%E3%82%B0%E3%82%A4%E3%83%B3%E4%BD%9C%E6%88%90%E5%85%A5%E9%96%80_%E3%81%9D%E3%81%AE%EF%BC%93_-_%E6%96%B0%E8%A6%8FUI%28%E3%82%BF%E3%83%96%29%E8%BF%BD%E5%8A%A0_:embed:cite]  

## 更新履歴

- 2018年2月13日 新規作成
- 2020年10月8日 体裁整え
