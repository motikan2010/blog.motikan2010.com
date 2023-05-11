<div style="text-align:center;">[f:id:motikan2010:20201205183354p:plain]</div>

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　普段私はWebアプリ開発に Laravel を利用しており、<span><a href="https://packagist.org/" target="_blank">Packagist</a></span>(パッケージリポジトリ) からパッケージを取得して開発しています。  

　しかしこれらのパッケージがどのようにして Packagist に登録されているのか分からない状態であったので、簡単なパッケージを作成して Packagist に登録してみました。  

　<span style="color: #ff0000">結論から言うと Packagist にパッケージを登録するのは非常に簡単でした。</span>  
パッケージ登録に審査等はなく「<b>1. アカウントの作成</b>」と「<b>2. パッケージの登録</b>」の手順を踏むだけでパッケージを全世界に公開することができます。  

## 登録の流れ

### 1. アカウントの作成

　メールアドレスがあれば登録できます。  

<figure class="figure-image figure-image-fotolife" title="サインアップ画面">[f:id:motikan2010:20201205040002p:plain]<figcaption>サインアップ画面</figcaption></figure>

### 2. パッケージの登録

　ヘッダの` Submit` からパッケージの登録ができます。  

　登録するパッケージの GitHub リポジトリの URL を入力します。  
<figure class="figure-image figure-image-fotolife" title="パッケージ登録画面">[f:id:motikan2010:20201204203724p:plain:w600]<figcaption>パッケージ登録画面</figcaption></figure>

　登録完了です。  
これで `composer install ~` コマンドでパッケージを導入することができるようになりました。  
<figure class="figure-image figure-image-fotolife" title="完了画面">[f:id:motikan2010:20201204204501p:plain:w600]<figcaption>完了画面</figcaption></figure>
  
　GitHub とアカウント連携を行うことでリポジトリが更新される毎に Packagist に自動的に反映させることができるようになります。  

### 3. 2FA（2要素認証）の有効化

　Packagist は 2FA に対応しています。  

<figure class="figure-image figure-image-fotolife" title="2FA は有効にしておく">[f:id:motikan2010:20201205041454p:plain]<figcaption>2FA は有効にしておく</figcaption></figure>  

　<span style="color: #ff0000">Packagist アカウントへの不正アクセスは自分だけでなく、パッケージ導入者全員に影響を与えるので 2FA は有効にしておいた方がいいでしょう。</span>  

## まとめ

　Packagist へのパッケージ登録は当初想像していたよりも簡単に行うことができました。  

　ちなみに今回登録したパッケージは下のものになります。  
[https://packagist.org/packages/motikan2010/sensitive-response-detector:embed:cite]

　どのようなパッケージかと言うと、レスポンスに機微な情報が含まれていたらレスポンスが返されないようにするものです。  
[https://github.com/motikan2010/Sensitive-Response-Detector:embed:cite]

　せっかくアカウントを作成したので、今後もパッケージ開発に勤しみたいと思った次第です。  