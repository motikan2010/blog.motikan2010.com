[f:id:motikan2010:20190127165714p:plain]
### TL;DR
・「Konva.Transformer」コンストラクタの引数を指定することでデザインを変更可能


### 背景
　先日開発したWebアプリケーションでは下画像を見たら分かるようにアンカーをドラッグすることによって画像のサイズ・角度を変えられるようになっています。  
↓そのWebアプリケーション  
[https://chosensya-maker.motikan2010.com:title]  



<!-- more -->



　これはCanvasのライブラリである「Konva」を利用して実現しています。
[f:id:motikan2010:20190127093257g:plain]  

### 問題

　このライブラリを用いることによって簡単に画像の変形ができるようになったが、<b><span style="color: #ff5252">スマホで操作した場合にアンカー（ドラッグ対象の正方形）が小さく変形が難しい</span></b>という問題がありました。  
[f:id:motikan2010:20190127162336p:plain:w300]  
　Konva.Transformerについて調べてみるとプロパティを追加するだけで、アンカーサイズや枠色といったスタイルを変えられることが分かりました。

##### デモページ
[https://konvajs.github.io/docs/select_and_transform/Transformer_Styling.html:title]

### 対応
##### 変更前
```javascript
var tr = new Konva.Transformer();
```

##### 変更後
　引数に値を設定することでアンカーサイズや枠のデザインを変更しています。  
設定値には今回使用したもの以外にもたくさんあるようです。  
[https://konvajs.github.io/api/Konva.Transformer.html:title]
```javascript
var tr = new Konva.Transformer({
        anchorStroke: '#FFBF00', // アンカー枠の色
        anchorFill: '#F2F2F2', // アンカー塗りつぶしの色
        anchorSize: 20, // アンカーのサイズ
        borderStroke: '#F7BE81', // 枠の色
        borderDash: [3, 3], // 枠のデザイン
    });
```

### 結果
　左が対応前・右が対応後の画像です。アンカーのサイズを大きくすることでスマホ上での操作も簡単に行えるようになりました。  
[f:id:motikan2010:20190127091656p:plain:alt=対応前画像:w300]
[f:id:motikan2010:20190127091650p:plain:alt=対応後画像:w300] 