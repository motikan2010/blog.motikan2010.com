<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに
　最近ZF2をさわり始めたので、メモ程度に書いてみる。  
ちなみにHerokuはHTTPS前提なのでFacebookのOAuth認証といったHTTPSが必要となる動作確認で重宝した。  

## Zend Framework2 インストール

　　ZF2には雛形となるアプリケーションが用意されているので今回はこちらを利用する。  
[https://framework.zend.com/downloads/skeleton-app:embed:cite]  

```
$ composer create-project -n -sdev zendframework/skeleton-application ZF2
```

## Procfileの作成

[Customizing Web Server and Runtime Settings for PHP | Heroku Dev Center](https://devcenter.heroku.com/articles/custom-php-settings#setting-the-document-root)  
　ドキュメントルートを「/public」に設定する為、下記のコマンドで「Procfile」を作成します。
```
echo "web: vendor/bin/heroku-php-apache2 public/" > Procfile
```

## Herokuへデプロイ

　デプロイの際に「composer.lock」が.gitignoreファイルに含まれていないことを確認してください。  

[[herokuがcomposer.lock必須になったのでcomposerの入れ方をメモしておく - KayaMemo]()](http://kayakuguri.github.io/blog/2015/08/25/composer-lock-require/)

```
$ heroku create
$ git add .
$ git commit -m "first commit."
$ git push heroku master
```

## 動作確認

```
$ heroku open
```

[f:id:motikan2010:20170806185032j:plain]  
終わり。「Procfileの作成」の部分が普段と違うところだろうか。
