<figure class="figure-image figure-image-fotolife" title="Yaraといったら「Passion Yara」">[f:id:motikan2010:20200910215835p:plain]<figcaption>Yaraといったら「Passion Yara」</figcaption></figure>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　GRR Rapid Response (以下GRR)について軽く説明、GRR はGoogleが開発しているインシデント対応ツールであり、主にフォレンジックを行うために利用されます。

　このツールの特徴としては、フォレンジック対象(GRRクライアント)のサーバにログインすることなく、GRRサーバからフォレンジックを実施することが可能です。  
　GRRクライアントを増やす場合は、対象サーバにエージェントをインストールするだけで完了です。  

　前回はこのツールを利用して、プロセス情報やブラウザの閲覧履歴の取得を行いました。  
今回は<span><a href="https://grr-doc.readthedocs.io/en/v3.2.0/what-is-grr.html" target="_blank">GRRドキュメント</a></span>に「YARA」の記載がありましたので、この機能を試してみます。  

　ドキュメントには以下の説明があり、<span class="m-y">GRRのYARA機能にはプロセスメモリの解析</span>ができそうです。  
> Live remote memory analysis using YARA library.

　GRRの導入はこちらを参照ください。  
[https://blog.motikan2010.com/entry/2020/09/09/%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3_%E3%82%A4%E3%83%B3%E3%82%B7%E3%83%87%E3%83%B3%E3%83%88%E5%AF%BE%E5%BF%9C%E3%83%84%E3%83%BC%E3%83%AB_GRR_Rapid_Response_%E6%A7%8B%E7%AF%89%E7%B7%A8:embed:cite]

<!-- more -->

## 動作確認

　ここからはGRRサーバ・クライアントは導入済みという前提で説明を進めていきます。  

　では、YARAツールの動作確認を行っていきます。  
ツールを利用してプロセスのメモリから特定の文字列を検索するまでを目的にします。  

以下のPythonプログラムを動作させて、そのファイル名とプロセスを特定してみます。  
<div class="md-code" style="width:70%">
```python
hoge = 'Hello Malware'
while True:
    pass
```

</div>

### YARAルールでプロセスを特定

　GRRクライアントの詳細ページに入っているところから説明を行っていきます。

　左サイドバーの<span class="m-y">「Start new Flows」 → 「Memory - Yara Process Scan」</span>画面からYARAツールを使うことができます。  
  
　「Yara signature」にYARAルールを入力します。以下のYARAルールを１行にしたものを入力しています。    
<div class="md-code" style="width:70%">
```
rule TestRule {
    meta: description = "test" 
    strings: $text_string = "Hello Malware" 
    condition: $text_string 
}
```
</div>

　「Pids(プロセスID)」で絞り込むことも可能ですが、プロセスIDがわからない状態ですので今回は指定はしません。  

　「Launch」ボタンを押下すると検索処理が開始されます。  
[f:id:motikan2010:20200910220729p:plain:w600]  

　条件に該当するプロセスが１件見つかりました。  
プロセスIDは7402、プログラムのファイル名は test.py ということが分かりました。  

　このようにYARAツールを利用することで、マルウェアなどのプロセスの検索が容易になります。  
検索対象のサーバにログインする必要もありません。  
[f:id:motikan2010:20200910220724p:plain:w600]  

　例を１つ紹介しただけですが、YARAツールの使い方の説明は以上です。  

現状、プロセスメモリのみが解析対象で、ファイルが解析対象外となっているのが欠点ですが、定期的なプロセスの監視には便利そうです。  

## まとめ

　また少しだけGRRに詳しくなりました。  
また気にある機能として「タスクの定期的な自動実行」や「REST APIからの操作」があり、これまた便利そうな機能です。  

これらの機能と今回学習したYARAツールを組み合わせると、多くのYARAルールを使っての解析が行えるようになりますので、実際のプロダクトなどでもGRRを利用できそうです。  

　次はこの辺りをさわっていく予定です。  

## 更新履歴

- 2020年9月10日 新規作成
