<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　先日、PHP-FPMにRCEの脆弱性(CVE-2019-11043)があることが公表されました。  

　脆弱性の詳細について本記事で説明しません。脆弱性を含んでいるバーションや攻撃が可能となる条件等は下のリンクの記事を参照して下さい。  

　本記事では、この脆弱性に対しての攻撃をブロックするAWS WAFのルールの作成方法を紹介します。  
「PHPのアップデート」や「Nginxの設定変更」で対応できますが、それが行えない環境下での対応手段の1つだと思っています。  
ブロックルールは単純なので、AWS WAF以外の製品にも適用できるとも考えています。

[https://jp.tenable.com/blog/cve-2019-11043-vulnerability-in-php-fpm-could-lead-to-remote-code-execution-on-nginx:embed:cite]



<!-- more -->

## AWS WAFの設定

### 文字列マッチの条件を作成

　「String and regex match conditions」を選択します。

[f:id:motikan2010:20191107000348p:plain:w500]

| 項目 | 設定値 |
| - | - |
| Name | HTTP Splitting Condition(任意) |
| Part of the request to filter on | URI |
| Match type | Contains |
| Transformation | Convert to lowercase |
| Value is base64-encoded | チェックなし |
| Value to match | %0a |

　もう1つ、「Value to match」が異なるパターンを追加します。他の項目は上記設定と同じです。  

| 項目 | 設定値 |
| - | - |
| Value to match | %0d |

　設定が環境したらこちらの画像通りになります。  
<span class="m-y">URIに「%0a」又は「%0d」が含まれていたらブロック</span>するようしています。  
「Transformation : Convert to lowercase」を指定することにより「%0A」「%0D」が入力された場合でも条件にマッチするようにしています。

[f:id:motikan2010:20191107000354p:plain:w300]

### ルールを作成

　今度はルールを作成します。ルール名は「HTTP Splitting Rule」にしています。  
このルールに先ほど作成した条件「HTTP Splitting Condition」を追加します。

[f:id:motikan2010:20191107000357p:plain:w500]

### ACLにルールを追加

　最後にACLに対して先に作成したルールを付与します。  
ここでは「sample-20191106-acl」の名前のACLに対してルールを付与しています。

[f:id:motikan2010:20191107000401p:plain:w500]

　ルールの付与が完了しているとACLのRulesにルールが表示されます。

[f:id:motikan2010:20191107001412p:plain:w500]

## 攻撃がブロックされることを確認

　攻撃がブロックされることをPoC(neex/phuip-fpizdam)を使用して確認します。  
結果は以下の通りになり、攻撃は失敗しブロックされていることが確認できます。
```
$ phuip-fpizdam http://xxxxx.ap-northeast-1.elb.amazonaws.com/script.php
2019/11/07 00:18:46 Base status code is 403
2019/11/07 00:18:50 Detect() returned error: no qsl candidates found, invulnerable or something wrong
```

　ブロックされたリクエストはAWS WAFの履歴に残っています。  
[f:id:motikan2010:20191107004606p:plain]

## 参考

[https://github.com/neex/phuip-fpizdam:title]  
　環境構築（Dockerイメージ）とPoCが紹介されています。

[https://github.com/SpiderLabs/owasp-modsecurity-crs/pull/1610:title]  
　本脆弱性に対する OWASP CRSのブロックルールです。

## 更新履歴
- 2019年11月 6日 新規作成
