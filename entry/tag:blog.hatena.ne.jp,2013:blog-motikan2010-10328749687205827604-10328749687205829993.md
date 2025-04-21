[f:id:motikan2010:20170112012951p:plain]  

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　今回はBot等に自動認証を許さないために用いられる「CAPTCHA」をRailsアプリケーションに実装してみます。  
 ImageMagickあたりでつまずいたので、備忘録感覚にメモ。  


<!-- more -->


## 環境確認とアプリの作成 

　まずは動作環境の確認とサンプルサプリケーションの作成を行っていきます。  
現在普及している Rails 4系 を使っていきます。

<div class="md-code" style="width:100%">

```
$ cat /etc/redhat-release
CentOS release 6.8 (Final)
$ rails _4.2.0_ new app
```
</div>

### （失敗）『simple-captcha』のセットアップ

　本記事でメインとなるキャプチャのライブラリですが、上位の人気を誇っている「**simple-captcha**」を入れてみました。

<div class="md-code" style="width:100%">
```
$ vim Gemfile

(追加)
gem 'simple_captcha', :git => 'git://github.com/galetahub/simple-captcha.git'
```
</div>

　そしてインストールを実行します。

<div class="md-code" style="width:100%">
```
# bundle install
```
</div>

### 

　ここで問題が発生しました。

<div class="md-code" style="width:100%">
```
$ rails generate simple_captcha
Running via Spring preloader in process 11946
   identical  app/views/simple_captcha/_simple_captcha.erb
/opt/rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/simple-captcha-2602bf19a63d/lib/generators/simple_captcha_generator.rb:18:in `create_migration': wrong number of arguments (given 3, expected 0) (ArgumentError)
  from /opt/rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/gems/railties-4.2.0/lib/rails/generators/migration.rb:63:in `migration_template'
  from /opt/rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/simple-captcha-2602bf19a63d/lib/generators/simple_captcha_generator.rb:19:in `create_migration'
```
</div>

　https://github.com/pludoni/simple-captcha/issues/18

　上記URL先を見てみると、新バージョンである『**SimpleCaptcha2**』を使えば解決できそうな予感。

### 『SimpleCaptcha2』のセットアップ

<div class="md-code" style="width:100%">
```
$ vim Gemfile
(追加)
gem 'simple_captcha2', git: 'https://github.com/pludoni/simple-captcha.git', require: true
```
</div>

　そして再度インストール

<div class="md-code" style="width:100%">
```
$ bundle install
```
</div>

### 改めて「rails generate simple_captcha」コマンド

<div class="md-code" style="width:100%">
```
$ rails generate simple_captcha
Running via Spring preloader in process 1067
Expected string default value for '--helper'; got true (boolean)
Expected string default value for '--jbuilder'; got true (boolean)
      create  app/views/simple_captcha/_simple_captcha.erb
      create  db/migrate/20170112000239_create_simple_captcha_data.rb
```
</div>

　様々なファイルが作成されたということで今回は成功したらしい。  
  
　`db/migrate/`に下記のようなマイグレーションファイルが作成されたことが確認できます。

<div class="md-code" style="width:100%">
```ruby
$ cat db/migrate/20170112000239_create_simple_captcha_data.rb

class CreateSimpleCaptchaData < ActiveRecord::Migration
  def self.up
    create_table :simple_captcha_data do |t|
      t.string :key, :limit => 40
      t.string :value, :limit => 6
      t.timestamps
    end

    add_index :simple_captcha_data, :key, :name => "idx_key"
  end

  def self.down
    drop_table :simple_captcha_data
  end
end
```
</div>

　下記コマンドでテーブルを作成します。

<div class="md-code" style="width:100%">
```
$ rake db:migrate
== 20170112000239 CreateSimpleCaptchaData: migrating ==========================
-- create_table(:simple_captcha_data)
   -> 0.0026s
-- add_index(:simple_captcha_data, :key, {:name=>"idx_key"})
   -> 0.0012s
== 20170112000239 CreateSimpleCaptchaData: migrated (0.0040s) =================
```

　テーブルが正常に作成されたことを確認して、下記のように `application.rb` コントローラに「include SimpleCaptcha::ControllerHelpers」の一行を追記します。

```ruby
$ vim app/controllers/application.rb
 
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception
  include SimpleCaptcha::ControllerHelpers
end
```
</div>

### テストアプリを作成

　ここでは簡単にRESTアプリを作成します。

<div class="md-code" style="width:100%">
```
# rails g scaffold Memo title:string description:text
# rake db:migrate
```
</div>

#### ビューを変更

　下記のようにキャプチャ入力項目を追記します。

<div class="md-code" style="width:100%">
```
$ vim app/views/memos/_form.html.erb

(省略)
<div class="field">
    <%= f.label :description %><br>
    <%= f.text_area :description %>
  </div>
  <div class="field">
    <%= f.label "Simple-Captcha" %><br>
    <%= f.simple_captcha :label => "上の文字を記入してくだい", :placeholder => "ここに入力" %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
(省略)
```
</div>

#### ブラウザでキャプチャの確認

　ここまで進めますと「`http://localhost:3000/memos/new`」にアクセスするとキャプチャ入力項目が表示されるはずですので、確認してみます。

[f:id:motikan2010:20170112001609p:plain:w200]  

　キャプチャ画像が表示されていません・・・。  

　サーバ側のログを確認すると以下のようなエラーが表示されています。

<div class="md-code" style="width:100%">
```
StandardError (Error while running convert: convert: not authorized `ACTWH' @ error/constitute.c/ReadImage/453.
convert: missing an image filename `jpeg:-' @ error/convert.c/ConvertImageCommand/3015.
):
  /opt/rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/simple-captcha-b145495ab9e5/lib/simple_captcha/utils.rb:17:in `run'
  /opt/rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/simple-captcha-b145495ab9e5/lib/simple_captcha/image.rb:83:in `generate_simple_captcha_image'
  /opt/rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/simple-captcha-b145495ab9e5/lib/simple_captcha/middleware.rb:42:in `make_image'
  /opt/rbenv/versions/2.3.0/lib/ruby/gems/2.3.0/bundler/gems/simple-captcha-b145495ab9e5/lib/simple_captcha/middleware.rb:21:in `call'
(以下省略)
```
</div>

#### キャプチャ画像を表示

　画像の加工処理がうまくいっていないようですので、その処理に必要なパッケーシをインストールします。

<div class="md-code" style="width:100%">
```
# yum -y install ghostscript
# yum -y install ImageMagick-devel
```
</div>

　ここでブラウザで確認してみても画像が表示されていません。
http://www.srcw.net/wiki/index.php?SimpleCaptcha

　調べてみるとImageMagickに脆弱性があり、設定ポリシーが厳しくなりデフォルト設定だと表示されていないと推測。

　その脆弱性に関する情報はこちらを参照

[http://d.hatena.ne.jp/Kango/20160504/1462352882:title]

　その設定ファイルである「ポリシーファイル」を以下のように編集してみます。

<div class="md-code" style="width:100%">
```
$ vim /etc/ImageMagick/policy.xml

（省略）
<policymap>
  <!-- <policy domain="system" name="precision" value="6"/> -->
  <!-- <policy domain="resource" name="temporary-path" value="/tmp"/> -->
  <!-- <policy domain="resource" name="memory" value="2GiB"/> -->
  <!-- <policy domain="resource" name="map" value="4GiB"/> -->
  <!-- <policy domain="resource" name="area" value="1gb"/> -->
  <!-- <policy domain="resource" name="disk" value="16eb"/> -->
  <!-- <policy domain="resource" name="file" value="768"/> -->
  <!-- <policy domain="resource" name="thread" value="4"/> -->
  <!-- <policy domain="resource" name="throttle" value="0"/> -->
  <!-- <policy domain="resource" name="time" value="3600"/> -->
<!--
  <policy domain="coder" rights="none" pattern="EPHEMERAL" />
  <policy domain="coder" rights="none" pattern="HTTPS" />
  <policy domain="coder" rights="none" pattern="HTTP" />
  <policy domain="coder" rights="none" pattern="URL" />
  <policy domain="coder" rights="none" pattern="FTP" />
  <policy domain="coder" rights="none" pattern="MVG" />
  <policy domain="coder" rights="none" pattern="MSL" />
  <policy domain="coder" rights="none" pattern="TEXT" />
  <policy domain="coder" rights="none" pattern="LABEL" />
  <policy domain="path" rights="none" pattern="@*" />
-->
</policymap>
```
</div>

　編集を終えたら再度ブラウザでアクセスし確認します。  
[f:id:motikan2010:20170112001211p:plain:w200]  

　今度はうまく表示できているようです。

　ですが肝心なキャプチャをデタラメな値を入力しても作成できるようになっています。  

　正常に動作させるためにモデルとコントローラの編集を行います。

### 正常に動作させるためモデルとコントローラを編集

#### モデル

　「attr_accessor :captcha_key, :captcha」の一行を追記します。

<div class="md-code" style="width:100%">
```ruby
$ vim app/models/memo.rb

class Memo < ActiveRecord::Base
  attr_accessor :captcha_key, :captcha
end
```
</div>

#### コントローラ

末尾にあるmemo_params関数の中身を以下のように編集します。  
「`, :captcha_key, :captcha`」を追加

<div class="md-code" style="width:100%">
```ruby
$ vim app/controllers/memos_controller.rb

(省略)
    def memo_params
        params.require(:memo).permit(:title, :description, :captcha_key, :captcha)
    end
end
```
</div>

同じコントローラファイルに入力されたキャプチャ文字列が正しいかの検証処理をcreateメソッド内に記述します。  以下のように編集します。

<div class="md-code" style="width:100%">
```ruby
$ vim app/controllers/memos_controller.rb

  # POST /memos
  # POST /memos.json
  def create
    @memo = Memo.new(memo_params)

    if simple_captcha_valid?
      respond_to do |format|
        if @memo.save
          format.html { redirect_to @memo, notice: 'Memo was successfully created.' }
          format.json { render :show, status: :created, location: @memo }
        else
          format.html { render :new }
          format.json { render json: @memo.errors, status: :unprocessable_entity }
        end
      end
    else
        redirect_to :action => "new"
    end
  end
```
</div>

　最後に入力された値の引き渡しがうまくいくように下記のコードを記述します。

<div class="md-code" style="width:100%">
```ruby
$ vim config/environment.rb

module SimpleCaptcha
  module ControllerHelpers
    def simple_captcha_valid?
      return true if Rails.env.test?
      if params[:memo][:captcha]
        data = SimpleCaptcha::Utils::simple_captcha_value(params[:memo][:captcha_key] || session[:captcha])
        result = data == params[:memo][:captcha].delete(" ").upcase
        SimpleCaptcha::Utils::simple_captcha_passed!(session[:captcha]) if result
        return result
      else
        return false
      end
    end
  end
end
```
</div>

これで入力された文字列と画像に表示されている文字列の比較が行われ、等しい場合に正常な処理が行われるようになります。  

## 参考サイト

[https://joppot.info/2014/06/08/1555:embed:cite]

