<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　証明書のインポート毎に忘却しており、証明書探しの旅に出てしまっているので「THE  備忘録」

## やること

- Burp SuiteのSSL証明書を Firefox にインポート

## 使ったブラウザ  

- Firefox : 57.0.4 

## 手順

### 1. 初期状態だと証明書エラー

[f:id:motikan2010:20180124004545p:plain:w600]


<!-- more -->


### 2. 証明書のダウンロード

[f:id:motikan2010:20180124004552p:plain:w600]  

- プロキシが動作している <span style="color: #ff0000">IPアドレス:ポート番号</span> にアクセス
- 右上の「CA Certificate」を押下

### 3. インポート済みの証明書一覧を表示

[f:id:motikan2010:20180124004556p:plain:w600]  
[f:id:motikan2010:20180124004559p:plain:w600]

### 4. ダウンロードした証明書をインポート

[f:id:motikan2010:20180124004602p:plain:w600]

### 5. ダイアログの上部にチェック後、「OK」

[f:id:motikan2010:20180124004607p:plain:w600]

### 6. 証明書がインポートがされたことを確認

[f:id:motikan2010:20180124004610p:plain:w600]

### 7. HTTPSサイトにアクセス

[f:id:motikan2010:20180124004613p:plain:w600]

- HTTPSのサイトには正常にアクセスされ、Burp Suite の証明書が利用されていることが確認できる。  
<span style="color: #ff0000">⊂(・∀・)⊃ﾔｯT！</span>