<div style="text-align:center;">[f:id:motikan2010:20180218185211p:plain:w500]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　今回はBurp Suiteに新規タブ(画面)を追加していきます。  

画面を追加することによって、複雑な機能を追加させることが可能になります。

## 完成のイメージ

![](https://i.imgur.com/Zn5u3Cg.jpg)

　「Alerts」タブの右側に新規に「`Sample Tab Extender`」という名前のタブが追加されているのが確認できます。    

　画像のプラグインでは「`Save Request`」というコンテキストメニューを押下後、新規タブの左側のテーブルに保存されます。   
 
　テーブル内の行を選択したら、右側にテキストエリアに通信のリクエスト（上部）とレスポンス（下部）が表示されます。

今回はこのような新規タブの追加方法・レイアウトについて記載します。

<!-- more -->

## Burp Suite Extender レイアウト

　タブを追加した場合、自分で画面のレイアウトを作成しなくてはなりません。
 
Burp Suite Extenderのレイアウトは、<span class="m-y">AWT(Abstract Window Toolkit)</span>を利用して作成するため、自分好みのレイアウトが簡単に作成することが可能です。

AWTはBurp Suite特有の技術では無く、JavaのGUIアプリでよく利用されている（いた?）ツールキットであり、汎用的な技術であるため比較的容易に実装にあたっての情報を探すことができます。

　先に記述した通り、AWTに関する情報は既に大量にありますので、この記事ではレイアウトの細かい設定等については言及しません。主要なレイアウトと新規追加されるタブとBurp Suiteを連携させるかのかを説明していきます。

### 学ぶこと

- タブの追加
- 新規タブ内のレイアウト
- コンポーネント（テーブルやテキストエリア）の配置

　この記事では2つのプロジェクト(サンプルアプリ)で説明します。  
１つ目は、様々な主要レイアウトを反映させただけの拡張プラグイン  
２つ目は、Burp Suiteで取得した通信ログを見る拡張プラグイン（上記完成物画像を参考）

## レイアウト確認用 拡張プラグイン

[https://github.com/motikan2010/Burp-Suite-Learning-Chapter03/tree/sample1:embed:cite]

### ディレクト構成

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
                    └── SampleTab.java
```
</div>

##### BurpExtender.java

- タブに表示するテキストの設定
- タブ選択時に表示するGUIを設定

##### SampleTab.java

- タブ選択時に表示するGUIの中身

### 開発

##### BurpExtender.java

<div class="md-code" style="width:100%">
```java
package burp;  
  
import com.motikan2010.SampleTab;  
  
import javax.swing.*;  
import java.awt.*;  
  
public class BurpExtender implements IBurpExtender, ITab {  

    private static final String EXTENSION_NAME = "Sample Tab Extender";
    
    private SampleTab sampleTab;  
    
    public void registerExtenderCallbacks(final IBurpExtenderCallbacks iBurpExtenderCallbacks) {
        iBurpExtenderCallbacks.setExtensionName(EXTENSION_NAME);
        
        SwingUtilities.invokeLater(() -> {  
           sampleTab = SampleTab.getInstance();  
           sampleTab.render(SampleTab.FLOW_LAYOUT);  
           iBurpExtenderCallbacks.addSuiteTab(BurpExtender.this);  
        });  
    }  
    
    public String getTabCaption() {  
        return EXTENSION_NAME;
    }  
    
    public Component getUiComponent() {  
        return sampleTab;  
    }  
}

```
</div>

##### SampleTab.java

「BurpExtender.java」内の

<div class="md-code" style="width:100%">
```java
sampleTab.render(SampleTab.FLOW_LAYOUT);
```
</div>
で指定されたレイアウトが呼び出されるようになっています。

<div class="md-code" style="width:100%">
```java
package com.motikan2010;  
  
import javax.swing.*;  
import java.awt.*;  
  
public class SampleTab extends JPanel {  
  
    private static SampleTab panel;  
  
    public static final int FLOW_LAYOUT     = 0;  
    public static final int GRID_LAYOUT     = 1;  
    public static final int GRID_BAG_LAYOUT = 2;  
    public static final int BORDER_LAYOUT   = 3;  
    public static final int BOX_LAYOUT      = 4;  
    public static final int PANEL_LAYOUT    = 5;  
    
    public static SampleTab getInstance() {  
        if (panel == null) {  
            panel = new SampleTab();  
       }  
       return panel;  
    }  
    
    public void render(int type) {  
        switch (type) {  
            case 0:  
                renderFlowLayout();  
                break;  
                
            case 1:  
                renderGridLayout();  
                break;  

            case 2:  
                renderGridBagLayout();  
                break; 
        
            case 3:  
                renderBorderLayout();  
                break; 
                
            case 4:  
                renderBoxLayout();  
                break;  
                
            case 5:  
                renderPanelLayout();  
                break;  
        }  
    }

    /* 下記に記載 */

}
```
</div>

### レイアウトの設定

　ここでは、AWTの主要なレイアウトを設定し、Java製GUIアプリで普段から使われているレイアウトが、Burp拡張プラグラインでも利用可能であるかを確認してみました。  
下記のサイトのコードをそのまま使うことができたので、より複雑なことをしない限り、例外なく使えると考えられます。  

[http://www.tohoho-web.com/java/layout.htm:title]

#### FlowLayout (フローレイアウト)

![](https://i.imgur.com/DRIact5.png)

<div class="md-code" style="width:100%">
```java
private void renderFlowLayout() {  
    setLayout(new FlowLayout());  
    
    JButton jButton1 = new JButton("ボタン1");  
    JButton jButton2 = new JButton("ボタン2");  
    JButton jButton3 = new JButton("ボタン3");
    
    add(jButton1);
    add(jButton2);
    add(jButton3);  
}
```
</div>

#### GridLayout (グリッドレイアウト)

![](https://i.imgur.com/i0MTavu.png)

<div class="md-code" style="width:100%">
```java
private void renderGridLayout() {  
    setLayout(new GridLayout(2, 3));  
    
    add(new JButton("ボタン1"));
    add(new JButton("ボタン2"));
    add(new JButton("ボタン3"));
    add(new JButton("ボタン4"));
    add(new JButton("ボタン5"));
    add(new JButton("ボタン6"));  
}
```
</div>

#### GridBagLayout (グリッドバッグレイアウト)

![](https://i.imgur.com/vFGc7xL.png)

<div class="md-code" style="width:100%">
```java
private void renderGridBagLayout() {  
    GridBagLayout gridBagLayout = new GridBagLayout();
    setLayout(gridBagLayout);  
    
    GridBagConstraints gbc = new GridBagConstraints();
    
    JButton jButton1 = new JButton("ボタン1");  
    gbc.gridx = 0;  
    gbc.gridy = 0;  
    gridBagLayout.setConstraints(jButton1, gbc);  
    
    JButton jButton2 = new JButton("ボタン2");  
    jButton2.setFont(new Font("Arial", Font.PLAIN, 30));  
    gbc.gridx = 1;  
    gbc.gridy = 0;  
    gridBagLayout.setConstraints(jButton2, gbc);  
    
    JButton jButton3 = new JButton("ボタン3");  
    gbc.gridx = 1;  
    gbc.gridy = 1;  
    gridBagLayout.setConstraints(jButton3, gbc);  
    
    add(jButton1);  
    add(jButton2);  
    add(jButton3);  
}
```
</div>

#### BorderLayout (ボーダーレイアウト)

![](https://i.imgur.com/XifFu2J.png)

<div class="md-code" style="width:100%">
```java
private void renderBorderLayout() {  
    setLayout(new BorderLayout());  
    
    add("North", new JButton("ボタン1"));  
    add("East", new JButton("ボタン2"));  
    add("South", new JButton("ボタン3"));  
    add("West", new JButton("ボタン4"));  
    add("Center", new JButton("ボタン5"));  
}
```
</div>

#### BoxLayout (ボックスレイアウト)

![](https://i.imgur.com/p6haidK.png)  

[https://www.javadrive.jp/tutorial/boxlayout/index2.html:title]

<div class="md-code" style="width:100%">
```java
private void renderBoxLayout() {  
    setLayout(new BoxLayout(this, BoxLayout.X_AXIS));  
    
    add(new JButton("ボタン1"));  
    add(new JButton("ボタン2"));  
    add(new JButton("ボタン3"));  
}
```
</div>

#### Panel (パネルを利用したレイアウト)

![](https://i.imgur.com/zdbyJ4j.png)  

　少々手間とはなりますが、パネルを組み合わせることで複雑なレイアウトを作成できるので、今回はこの方法を使って拡張プラグインを作成していきます。

<div class="md-code" style="width:100%">
```java
private void renderPanelLayout() {  
    setLayout(new GridLayout(1, 2));  
    
    Panel panel1 = new Panel();  
    Panel panel2 = new Panel();  
    panel1.setLayout(new GridLayout(1, 1));  
    panel2.setLayout(new GridLayout(3, 1));  
    
    panel1.add(new JButton("ボタン1"));  
    panel2.add(new JButton("ボタン2"));  
    panel2.add(new JButton("ボタン3"));
    panel2.add(new JButton("ボタン4"));  
    
    add(panel1);  
    add(panel2);  
}
```
</div>

## HTTP通信確認用 拡張プラグイン

　ここからは Burp Suiteならではの拡張プラグイン を作成していきます。  

[https://github.com/motikan2010/Burp-Suite-Learning-Chapter03/tree/chapter3:embed:cite]

### ディレクトリ構成

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
                    ├── RequestTableManager.java
                    ├── RequestTableModel.java
                    ├── SampleTab.java
                    ├── entity
                    │   └── RequestResponseEntity.java
                    └── util
                        └── RequestResponseUtils.java
```
</div>

##### BurpExtender.java

- 新規タブのインスタンスの生成と反映
- 新規タブの命名

##### SampleTab.java

- レイアウトの設定
- テーブル内へのデータを格納、テキストエリアへの反映

##### RequestContextMenu.java

- 「Save Request」がクリックされた際のイベントの設定

##### RequestTableModel.java
- テーブルの各列の設定など
- 実際にテーブル内の表示するデータのリストを保持

##### RequestTableManager.java
- Modelへの操作の橋渡しの役割

##### entity/RequestResponseEntity.java
- 保存されるデータのクラス
テーブル内に表示される「通信先ホスト」「HTTPメソッド」「リクエストパス」「レスポンスのステータスコード」の情報が格納されています。

##### util/RequestResponseUtils.java
- 前回同様の役割

### レイアウトの構成図
　JPanelクラスをベースに各コンポーネントを配置するようにしています。  
  
JTableはデータ数が一定数になるとパネル外にはみ出してしまうので、JScrollPane内に配置し、スクロールでデータを追えるようにしています。 

![](https://i.imgur.com/LKCTNGl.jpg)


##### BurpExtender.java

<div class="md-code" style="width:100%">
```java
package burp;

import com.motikan2010.SampleTab;
import com.motikan2010.RequestContextMenu;
import com.motikan2010.util.RequestResponseUtils;

import javax.swing.*;
import java.awt.*;
import java.util.*;
import java.util.List;

public class BurpExtender implements IBurpExtender, IContextMenuFactory, ITab {

    private static IBurpExtenderCallbacks callbacks;
    private static IExtensionHelpers helpers;

    private static final String EXTENSION_NAME = "Sample Tab Extender";

    private SampleTab sampleTab;
    private RequestResponseUtils requestResponseUtils;

    public void registerExtenderCallbacks(final IBurpExtenderCallbacks c) {
        c.setExtensionName(EXTENSION_NAME);
        callbacks = c;
        helpers = c.getHelpers();
        this.requestResponseUtils = RequestResponseUtils.getInstance();

        SwingUtilities.invokeLater(() -> {
            sampleTab = new SampleTab(requestResponseUtils);
            sampleTab.render();
            c.addSuiteTab(BurpExtender.this);
        });

        c.registerContextMenuFactory(this);
    }

    public String getTabCaption() {
        return EXTENSION_NAME;
    }

    public Component getUiComponent() {
        return sampleTab;
    }

    public static IBurpExtenderCallbacks getCallbacks() {
        return callbacks;
    }

    public static IExtensionHelpers getHelpers() {
        return helpers;
    }

    @Override
    public List<JMenuItem> createMenuItems(IContextMenuInvocation iContextMenuInvocation) {
        IHttpRequestResponse[] httpRequestResponseArray = iContextMenuInvocation.getSelectedMessages();
        if (null == httpRequestResponseArray) {
            return null;
        }

        List<JMenuItem> jMenuItemList = new LinkedList<>();

        JMenuItem requestJMenuItem = new JMenuItem("Save Request");
        requestJMenuItem.addMouseListener(new RequestContextMenu(httpRequestResponseArray, sampleTab));
        jMenuItemList.add(requestJMenuItem);

        return jMenuItemList;
    }
}
```
</div>

##### RequestContextMenu.java

<div class="md-code" style="width:100%">
```java
package com.motikan2010;

import com.motikan2010.entity.RequestResponseEntity;

import javax.swing.table.AbstractTableModel;
import java.util.ArrayList;
import java.util.List;

public class RequestTableModel extends AbstractTableModel {

    private final List<RequestResponseEntity> requestResponseEntityList = new ArrayList<>();;

    private static final String[] COLUMN_NAMES = {"Host", "Method", "Path", "Status"};

    private static final int TABLE_COLUMN_COUNT = 4;

    public static final int HOST_COLUMN_INDEX   = 0;
    public static final int METHOD_COLUMN_INDEX = 1;
    public static final int PATH_COLUMN_INDEX   = 2;
    public static final int STATUS_COLUMN_INDEX = 3;

    public void addRequestResponse(RequestResponseEntity requestResponse) {
        requestResponseEntityList.add(requestResponse);

        // https://docs.oracle.com/javase/8/docs/api/javax/swing/table/AbstractTableModel.html#fireTableRowsInserted-int-int-
        fireTableRowsInserted(0, requestResponseEntityList.size() - 1);
    }

    public RequestResponseEntity getRequestResponse(int rowIndex) {
        return requestResponseEntityList.get(rowIndex);
    }

    public int getColumnCount() {
        return TABLE_COLUMN_COUNT;
    }

    public int getRowCount() {
        return requestResponseEntityList.size();
    }

    public Object getValueAt(int rowIndex, int columnIndex) {
        RequestResponseEntity requestResponseEntity = requestResponseEntityList.get(rowIndex);

        switch (columnIndex) {
            case 0:
                return requestResponseEntity.getHost();
            case 1:
                return requestResponseEntity.getMethod();
            case 2:
                return requestResponseEntity.getPath();
            case 3:
                return requestResponseEntity.getResponseStatus();
            default:
                return "";
        }
    }

    @Override
    public String getColumnName(int columnIndex) {
        return COLUMN_NAMES[columnIndex];
    }

    @Override
    public Class<?> getColumnClass(int columnIndex) {
        return getValueAt(0, columnIndex).getClass();
    }

    @Override
    public boolean isCellEditable(int rowIndex, int columnIndex) {
        return true;
    }
}
```
</div>

##### RequestTableManager.java

<div class="md-code" style="width:100%">
```java
package com.motikan2010;

import com.motikan2010.entity.RequestResponseEntity;

public class RequestTableManager {

    private RequestTableModel model;

    public RequestTableManager(RequestTableModel requestTableModel) {
        this.model = requestTableModel;
    }

    /**
     * テーブルの添字からデータ（エンティティ）の取得
     * 
     * @param rowIndex
     * @return RequestResponseEntity
     */
    public synchronized RequestResponseEntity getRequestResponse(int rowIndex) {
        return this.model.getRequestResponse(rowIndex);
    }

    /**
     * テーブル内にデータを追加
     *
     * @param requestResponse
     */
    public void addRequestResponse(RequestResponseEntity requestResponse) {
        this.model.addRequestResponse(requestResponse);
    }
}
```
</div>

##### RequestTableModel.java

<div class="md-code" style="width:100%">
```java
package com.motikan2010;

import com.motikan2010.entity.RequestResponseEntity;

import javax.swing.table.AbstractTableModel;
import java.util.ArrayList;
import java.util.List;

public class RequestTableModel extends AbstractTableModel {

    private final List<RequestResponseEntity> requestResponseEntityList = new ArrayList<>();;

    private static final String[] COLUMN_NAMES = {"Host", "Method", "Path", "Status"};

    private static final int TABLE_COLUMN_COUNT = 4;

    public static final int HOST_COLUMN_INDEX   = 0;
    public static final int METHOD_COLUMN_INDEX = 1;
    public static final int PATH_COLUMN_INDEX   = 2;
    public static final int STATUS_COLUMN_INDEX = 3;

    public void addRequestResponse(RequestResponseEntity requestResponse) {
        requestResponseEntityList.add(requestResponse);

        // https://docs.oracle.com/javase/8/docs/api/javax/swing/table/AbstractTableModel.html#fireTableRowsInserted-int-int-
        fireTableRowsInserted(0, requestResponseEntityList.size() - 1);
    }

    public RequestResponseEntity getRequestResponse(int rowIndex) {
        return requestResponseEntityList.get(rowIndex);
    }

    public int getColumnCount() {
        return TABLE_COLUMN_COUNT;
    }

    public int getRowCount() {
        return requestResponseEntityList.size();
    }

    public Object getValueAt(int rowIndex, int columnIndex) {
        RequestResponseEntity requestResponseEntity = requestResponseEntityList.get(rowIndex);

        switch (columnIndex) {
            case 0:
                return requestResponseEntity.getHost();
            case 1:
                return requestResponseEntity.getMethod();
            case 2:
                return requestResponseEntity.getPath();
            case 3:
                return requestResponseEntity.getResponseStatus();
            default:
                return "";
        }
    }

    @Override
    public String getColumnName(int columnIndex) {
        return COLUMN_NAMES[columnIndex];
    }

    @Override
    public Class<?> getColumnClass(int columnIndex) {
        return getValueAt(0, columnIndex).getClass();
    }

    @Override
    public boolean isCellEditable(int rowIndex, int columnIndex) {
        return true;
    }
}
```
</div>

##### SampleTab.java

<div class="md-code" style="width:100%">
```java
package com.motikan2010;


import com.motikan2010.entity.RequestResponseEntity;
import com.motikan2010.util.RequestResponseUtils;

import javax.swing.*;

import java.awt.*;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;

public class SampleTab extends JPanel {

    private final RequestTableModel requestTableModel;
    private final RequestTableManager requestTableManager;

    private JTextArea requestTextArea;
    private JTextArea responseTextArea;

    private RequestResponseUtils requestResponseUtils;

    public SampleTab(RequestResponseUtils utils) {
        requestResponseUtils = utils;

        requestTableModel = new RequestTableModel();
        requestTableManager = new RequestTableManager(requestTableModel);
    }

    public void render() {

        setLayout(new GridLayout(1, 2));

        Panel panel1 = new Panel();
        Panel panel2 = new Panel();
        panel1.setLayout(new GridLayout(1, 1));
        panel2.setLayout(new GridLayout(2, 1));

        // テーブルの作成
        JTable jTable = new JTable(requestTableModel);
        jTable.setShowVerticalLines(true); // 縦線はあり
        jTable.setShowHorizontalLines(false); // 横線はなし

        // Click table row
        jTable.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseClicked(MouseEvent event) {
                selectRequest(jTable.getSelectedRow());
            }
        });

        // Host Column
        jTable.getColumnModel().getColumn(RequestTableModel.HOST_COLUMN_INDEX).setPreferredWidth(150);
        jTable.getColumnModel().getColumn(RequestTableModel.HOST_COLUMN_INDEX).setMinWidth(100);
        jTable.getColumnModel().getColumn(RequestTableModel.HOST_COLUMN_INDEX).setMaxWidth(250);

        // Method Column
        jTable.getColumnModel().getColumn(RequestTableModel.METHOD_COLUMN_INDEX).setMinWidth(50);
        jTable.getColumnModel().getColumn(RequestTableModel.METHOD_COLUMN_INDEX).setMaxWidth(60);

        // Path Column
        jTable.getColumnModel().getColumn(RequestTableModel.PATH_COLUMN_INDEX).setMinWidth(200);
        jTable.getColumnModel().getColumn(RequestTableModel.PATH_COLUMN_INDEX).setMaxWidth(300);

        // Status Column
        jTable.getColumnModel().getColumn(RequestTableModel.STATUS_COLUMN_INDEX).setMinWidth(50);
        jTable.getColumnModel().getColumn(RequestTableModel.STATUS_COLUMN_INDEX).setMaxWidth(60);

        // スクロールパネルにテーブルを追加します
        JScrollPane requestScrollPane = new JScrollPane(jTable);

        // リクエスト内容を表示するテキストエリア
        requestTextArea = new JTextArea();
        requestTextArea.setLineWrap(true);
        JScrollPane requestTextAreaPane = new JScrollPane(requestTextArea);

        // レスポンス内容を表示するテキストエリア
        responseTextArea = new JTextArea();
        responseTextArea.setLineWrap(true);
        JScrollPane responseTextAreaPane = new JScrollPane(responseTextArea);

        panel1.add(requestScrollPane);
        panel2.add(requestTextAreaPane);
        panel2.add(responseTextAreaPane);

        // ベースに２つのパネルを追加
        add(panel1);
        add(panel2);
    }

    /**
     * テーブルの行を選択
     * @param rowNum 行番号
     */
    public void selectRequest(int rowNum) {
        RequestResponseEntity requestResponseEntity = requestTableManager.getRequestResponse(rowNum);
        String request = requestResponseUtils.showRequest(requestResponseEntity.getRequestResponse());
        String response  = requestResponseUtils.showResponse(requestResponseEntity.getRequestResponse());
        requestTextArea.setText(request);
        responseTextArea.setText(response);
    }

    /**
     * リクエスト&レスポンスを保存
     *
     * @param requestResponseEntity 保存対象のリクエスト&レスポンスのエンティティ
     */
    public void keepRequest(RequestResponseEntity requestResponseEntity) {
        requestTableManager.addRequestResponse(requestResponseEntity);
    }
}
```
</div>

##### entity/RequestResponseEntity.java

<div class="md-code" style="width:100%">
```java
package com.motikan2010.entity;

import burp.*;

import java.net.URL;

public class RequestResponseEntity {

    public RequestResponseEntity(IHttpRequestResponse requestResponse) {
        IRequestInfo iRequestInfo = BurpExtender.getHelpers().analyzeRequest(requestResponse);
        IResponseInfo iResponseInfo = BurpExtender.getHelpers().analyzeResponse(requestResponse.getResponse());

        this.requestResponse = requestResponse;
        this.url = iRequestInfo.getUrl();
        this.method = iRequestInfo.getMethod();
        this.responseStatus = iResponseInfo.getStatusCode();
    }

    private IHttpRequestResponse requestResponse;

    private URL url;

    private String method;

    private int responseStatus;

    // リクエスト&レスポンスの詳細を取得する
    public IHttpRequestResponse getRequestResponse() {
        return requestResponse;
    }

    // 通信ホストの取得
    public String getHost() {
        return this.url.getHost();
    }

    // HTTPメソッドの取得
    public String getMethod() {
        return this.method;
    }

    // パスの取得
    public String getPath() {
        return this.url.getPath();
    }

    // レスポンスステータスコードの取得
    public int getResponseStatus() {
        return this.responseStatus;
    }
}
```
</div>

### まとめ

　新規UIを追加することで、拡張プラグインらしくなってきました。  
次は、テーブルに保存されたリクエストを再度送信する（Repeaterのようなもの）拡張プラグインを作成し、プログラム上からHTTP通信する方法を書いていきます。
