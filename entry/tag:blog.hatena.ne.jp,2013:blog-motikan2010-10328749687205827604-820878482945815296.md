<div style="text-align:center;">[f:id:motikan2010:20230701153051p:plain:w600]</div>

<div class="contents-box"><p>[:contents]</p></div>


## TL;DR

[f:id:motikan2010:20230701154309p:plain:w700]

## はじめに

### subfinder とは

　「subfinder」はサブドメインを列挙するツールであり、「ProjectDiscovery」が管理しています。  

　リポジトリのスターは 約8,000 あり、サブドメイン列挙ツールの中でも有名なツールだと思います。  

[https://github.com/projectdiscovery/subfinder:embed:cite]

以下は README.md から引用した本ツールの概要です。  

> 「**subfinder**」は受動的なオンラインソースを使用して、<span style="color: #ff0000">ウェブサイトの有効なサブドメインを返すサブドメイン発見ツール</span>です。  
subfinder は、パッシブ・サブドメインの列挙というただ一つのことを行うために作られており、それは非常によくできています。  
　私たちは、パッシブ・ソースで使用されているすべてのライセンスと使用制限に準拠するように作りました。  
パッシブ・モデルは、スピードとステルス性を保証し、ペネトレーション・テスターとバグ・バウンティ・ハンターの両方に活用できます。


　subfinder は外部サービスに依存しているツールですが、<span class="m-y">APIキーの登録なく利用できます</span>。    

　しかし、以下のサービスも参照する場合には事前にAPIキーを設定する必要があります。  

<div class="md-code" style="width:100%">

```
BeVigil, BinaryEdge, BufferOver, C99, Censys
CertSpotter, Chaos, Chinaz, DnsDB, Fofa, FullHunt
GitHub, Intelx, PassiveTotal, quake, Robtex
SecurityTrails, Shodan, ThreatBook, VirusTotal, WhoisXML API
ZoomEye, ZoomEye API, dnsrepo, Hunter
```

</div>

### サブドメイン列挙方法

　サブドメイン列挙の方法は「アクティブ・サブドメイン・列挙」と「パッシブ・サブドメイン・列挙」の２つに分けられます。  

　<span class="m-y">subfinder は「パッシブ・サブドメイン・列挙」に分類されます。  </span>

#### アクティブ・サブドメイン・列挙

[f:id:motikan2010:20230701153802p:plain:w600]

#### パッシブ・サブドメイン・列挙

[f:id:motikan2010:20230701153806p:plain:w600]

## 準備

### バージョン

　まずは本記事で利用する subfinder のバージョンを確認します。  

　「`-v`」オプションでバージョンを確認できます。

- subfinder v2.6.0

<div class="md-code" style="width:100%">

```
root@bf79c6792844:/usr/local/bin# ./subfinder -v

               __    _____           __
   _______  __/ /_  / __(_)___  ____/ /__  _____
  / ___/ / / / __ \/ /_/ / __ \/ __  / _ \/ ___/
 (__  ) /_/ / /_/ / __/ / / / / /_/ /  __/ /
/____/\__,_/_.___/_/ /_/_/ /_/\__,_/\___/_/

                projectdiscovery.io

[INF] Current subfinder version v2.6.0 (latest)
[FTL] Program exiting: no input list provided
```

</div>

### オプションの確認

　「`-h`」オプションで指定することができるオプションを確認できます。  

```
# ./subfinder -h

-----

Usage:
  ./subfinder [flags]

Flags:
INPUT:
  -d, -domain string[]  サブドメインを探す
  -dL, -list string     サブドメイン発見用のドメインリストを含むファイル

SOURCE:
  -s, -sources string[]           ディスカバリーに使用する特定のソースを指定する (-s crtsh,github)。
                                  利用可能なすべてのソースを表示するには -ls を使用します。
  -recursive                      サブドメインを再帰的に扱えるソースのみを使用する（例: subdomain.domain.tld vs domain.tld）
  -all                            すべてのソースを列挙に使用する（遅い）
  -es, -exclude-sources string[]  列挙から除外するソース (-es alienvault,zoomeye)

FILTER:
  -m, -match string[]   サブドメインまたはマッチするサブドメインのリスト（ファイルまたはカンマ区切り）
  -f, -filter string[]   サブドメインまたはフィルタリングするサブドメインのリスト (ファイルまたはカンマ区切り)

RATE-LIMIT:
  -rl, -rate-limit int  1秒間に送信するhttpリクエストの最大数
  -t int                解決するゴルーチンの同時実行数（-active only）（デフォルト 10）

UPDATE:
   -up, -update                 subfinder を最新バージョンに更新
   -duc, -disable-update-check  subfinder の自動更新チェックを無効にする

OUTPUT:
  -o, -output string       ファイルに出力を書き込む。
  -oJ, -json               出力をJSONL(ines)形式で書き出す。
  -oD, -output-dir string  出力を書き込むディレクトリ (-dL only)
  -cs, -collect-sources    出力にすべてのソースを含める (-json only)
  -oI, -ip                 出力にホストIPを含める (-active only)

CONFIGURATION:
  -config string                フラグ設定ファイル (デフォルト "$HOME/.config/subfinder/config.yaml")
  -pc, -provider-config string  プロバイダ設定ファイル (デフォルト "$HOME/.config/subfinder/provider-config.yaml")
  -r string[]                   使用するリゾルバのカンマ区切りリスト
  -rL, -rlist string            使用するリゾルバのリストを含むファイル
  -nW, -active                  アクティブなサブドメインのみを表示する
  -proxy string                 サブファインダーで使用するhttpプロキシ
  -ei, -exclude-ip              ドメインリストからIPを除外する

DEBUG:
  -silent             出力にサブドメインだけを表示する
  -version            サブファインダーのバージョンを表示
  -v                  より細かく出力を表示する
  -nc, -no-color      出力の色を無効にする
  -ls, -list-sources  利用可能なすべての情報源をリストアップする

OPTIMIZATION:
  -timeout int   タイムアウトまでの待ち時間（デフォルト30秒）
  -max-time int  列挙結果を待つ分数（デフォルト10）
```

## 検証

　実際に subfinder を動かしてみます。私が重要と思ったオプションのみを紹介しています。    

### 検索対象のドメインの指定方法

#### １つのドメインを指定

　「`-d`」オプションの前に検査対象のドメインを指定します。

<div class="md-code" style="width:100%">

```
# ./subfinder -d motikan2010.com
[INF] Detected old /root/.config/subfinder/config.yaml config file, trying to migrate providers to /root/.config/subfinder/provider-config.yaml
[INF] Migration successful from /root/.config/subfinder/config.yaml to /root/.config/subfinder/provider-config.yaml.

               __    _____           __
   _______  __/ /_  / __(_)___  ____/ /__  _____
  / ___/ / / / __ \/ /_/ / __ \/ __  / _ \/ ___/
 (__  ) /_/ / /_/ / __/ / / / / /_/ /  __/ /
/____/\__,_/_.___/_/ /_/_/ /_/\__,_/\___/_/

                projectdiscovery.io

[INF] Current subfinder version v2.6.0 (latest)
[INF] Loading provider config from /root/.config/subfinder/provider-config.yaml
[INF] Enumerating subdomains for motikan2010.com
www.motikan2010.com ⬅︎ ⭐️ ここから発見されたサブドメイン
namatube.motikan2010.com
hacker-trends.motikan2010.com
sample.motikan2010.com
blog.motikan2010.com
mymacmate.motikan2010.com
once-tech.motikan2010.com
dev.motikan2010.com
pre.motikan2010.com
stg2.motikan2010.com
stg.motikan2010.com
motikan2010.com
md-to-pdf.motikan2010.com
chosensya-maker.motikan2010.com
[INF] Found 14 subdomains for motikan2010.com in 19 seconds 391 milliseconds
```

</div>

　14つのサブドメインを収集することができました。

##### 収集されたサブドメインの考察

　「www」「blog」といったものは辞書にも登録されている文字列だと考えられるためアクティブサブドメイン列挙でも収集できたと思われる。

<div class="md-code" style="width:50%">

```
motikan2010.com
www.motikan2010.com
blog.motikan2010.com
```

</div>

<br>
　辞書に登録されていないであろうサブドメインが収集されている。  
アクティブサブドメイン列挙では収集でないと考えられるため、<span class="m-y">このような推測が難しいサブドメインを収集できるのはパッシブサブドメイン列挙の利点</span>。

<div class="md-code" style="width:50%">

```
namatube.motikan2010.com
hacker-trends.motikan2010.com
mymacmate.motikan2010.com
once-tech.motikan2010.com
md-to-pdf.motikan2010.com
chosensya-maker.motikan2010.com
```

</div>

<br>
　DNSに登録していないドメインが収集されていました。目的によっては誤検出にもなりえる。  
（後述するが5年ほど前に検証のために一時的に登録したドメインであった）

<div class="md-code" style="width:50%">

```
sample.motikan2010.com
dev.motikan2010.com
pre.motikan2010.com
stg2.motikan2010.com
stg.motikan2010.com
```

</div>

#### 複数のドメインを指定

　ドメイン間にカンマを挟むことで複数のドメインを指定することができます。  
以下の例では「`motikan2010.com`」と「`motikan2010.net`」のサブドメインが検索されます。  

<div class="md-code" style="width:100%">


```
# ./subfinder -d motikan2010.com,motikan2010.net

               __    _____           __
   _______  __/ /_  / __(_)___  ____/ /__  _____
  / ___/ / / / __ \/ /_/ / __ \/ __  / _ \/ ___/
 (__  ) /_/ / /_/ / __/ / / / / /_/ /  __/ /
/____/\__,_/_.___/_/ /_/_/ /_/\__,_/\___/_/

                projectdiscovery.io

[INF] Current subfinder version v2.6.0 (latest)
[INF] Loading provider config from /root/.config/subfinder/provider-config.yaml
[INF] Enumerating subdomains for motikan2010.com
motikan2010.com ⬅︎ ⭐️ motikan.com のサブドメイン
www.motikan2010.com
stg.motikan2010.com
sample.motikan2010.com
blog.motikan2010.com
pre.motikan2010.com
mymacmate.motikan2010.com
hacker-trends.motikan2010.com
once-tech.motikan2010.com
chosensya-maker.motikan2010.com
dev.motikan2010.com
stg2.motikan2010.com
namatube.motikan2010.com
md-to-pdf.motikan2010.com
[INF] Found 14 subdomains for motikan2010.com in 2 seconds 287 milliseconds
[INF] Enumerating subdomains for motikan2010.net
poc-in-github.motikan2010.net ⬅︎ ⭐️ motikan.net のサブドメイン
www.motikan2010.net
[INF] Found 2 subdomains for motikan2010.net in 16 seconds 264 milliseconds
```

</div>

#### ファイルに列挙したドメインを指定

　「`-dL <ファイル名>`」オプションを付けるとファイルに記述されたドメインが検索されます。  

<div class="md-code" style="width:100%">

```
----- ファイルの中身
# cat domains.txt
motikan2010.com
motikan2010.net

----- ファイルに記述された２ドメインが検索されます。
root@bf79c6792844:/usr/local/bin# ./subfinder -dL domains.txt

               __    _____           __
   _______  __/ /_  / __(_)___  ____/ /__  _____
  / ___/ / / / __ \/ /_/ / __ \/ __  / _ \/ ___/
 (__  ) /_/ / /_/ / __/ / / / / /_/ /  __/ /
/____/\__,_/_.___/_/ /_/_/ /_/\__,_/\___/_/

                projectdiscovery.io

[INF] Current subfinder version v2.6.0 (latest)
[INF] Loading provider config from /root/.config/subfinder/provider-config.yaml
[INF] Enumerating subdomains for motikan2010.com
stg.motikan2010.com
motikan2010.com
www.motikan2010.com
dev.motikan2010.com
namatube.motikan2010.com
blog.motikan2010.com
stg2.motikan2010.com
md-to-pdf.motikan2010.com
once-tech.motikan2010.com
pre.motikan2010.com
sample.motikan2010.com
mymacmate.motikan2010.com
chosensya-maker.motikan2010.com
hacker-trends.motikan2010.com
[INF] Found 14 subdomains for motikan2010.com in 2 seconds 282 milliseconds
[INF] Enumerating subdomains for motikan2010.net
www.motikan2010.net
poc-in-github.motikan2010.net
[INF] Found 2 subdomains for motikan2010.net in 16 seconds 397 milliseconds
```

</div>

### ツールログを出力をより詳細に出力

　「`-v`」オプションを付けるとツールログがより詳細に出力されます。  

<div class="md-code" style="width:100%">

```
# ./subfinder -d motikan2010.com -v

               __    _____           __
   _______  __/ /_  / __(_)___  ____/ /__  _____
  / ___/ / / / __ \/ /_/ / __ \/ __  / _ \/ ___/
 (__  ) /_/ / /_/ / __/ / / / / /_/ /  __/ /
/____/\__,_/_.___/_/ /_/_/ /_/\__,_/\___/_/

                projectdiscovery.io

[INF] Current subfinder version v2.6.0 (latest)
[INF] Loading provider config from /root/.config/subfinder/provider-config.yaml
[DBG] Selected source(s) for this search: anubis, fofa, riddler, censys, crtsh, fullhunt, whoisxmlapi, certspotter, chinaz, digitorus, quake, virustotal, bufferover, chaos, dnsrepo, hackertarget, bevigil, leakix, shodan, hunter, intelx, passivetotal, securitytrails, c99, dnsdumpster, alienvault, robtex
[INF] Enumerating subdomains for motikan2010.com
[DBG] Cannot use the whoisxmlapi source because there was no API key/secret defined for it.
[DBG] Cannot use the passivetotal source because there was no API key/secret defined for it.
[DBG] Cannot use the securitytrails source because there was no API key/secret defined for it.
[DBG] Cannot use the intelx source because there was no API key/secret defined for it.
[DBG] Cannot use the shodan source because there was no API key/secret defined for it.
[DBG] Cannot use the certspotter source because there was no API key/secret defined for it.
[DBG] Cannot use the quake source because there was no API key/secret defined for it.
[DBG] Cannot use the virustotal source because there was no API key/secret defined for it.
[DBG] Cannot use the bevigil source because there was no API key/secret defined for it.
[DBG] Cannot use the leakix source because there was no API key/secret defined for it.
[DBG] Cannot use the hunter source because there was no API key/secret defined for it.
[DBG] Cannot use the c99 source because there was no API key/secret defined for it.
[DBG] Cannot use the fofa source because there was no API key/secret defined for it.
[DBG] Cannot use the robtex source because there was no API key/secret defined for it.
[DBG] Cannot use the censys source because there was no API key/secret defined for it.
[DBG] Cannot use the fullhunt source because there was no API key/secret defined for it.
[DBG] Cannot use the dnsrepo source because there was no API key/secret defined for it.
[DBG] Cannot use the chaos source because there was no API key/secret defined for it.
[DBG] Cannot use the chinaz source because there was no API key/secret defined for it.
[DBG] Cannot use the bufferover source because there was no API key/secret defined for it.
[DBG] Response for failed request against https://jonlu.ca/anubis/subdomains/motikan2010.com:
[]
[WRN] Could not run source anubis: unexpected status code 300 received from https://jonlu.ca/anubis/subdomains/motikan2010.com
[digitorus] motikan2010.com
[alienvault] blog.motikan2010.com
[alienvault] www.motikan2010.com
[DBG] Response for failed request against https://riddler.io/search?q=pld:motikan2010.com&view_type=data_table:
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<HTML><HEAD><META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=iso-8859-1">
<TITLE>ERROR: The request could not be satisfied</TITLE>
</HEAD><BODY>
<H1>403 ERROR</H1>
<H2>The request could not be satisfied.</H2>
<HR noshade size="1px">
Request blocked.
We can't connect to the server for this app or website at this time. There might be too much traffic or a configuration error. Try again later, or contact the app or website owner.
<BR clear="all">
If you provide content to customers through CloudFront, you can find steps to troubleshoot and help prevent this error by reviewing the CloudFront documentation.
<BR clear="all">
<HR noshade size="1px">
<PRE>
Generated by cloudfront (CloudFront)
Request ID: 43OkGZsMblidN8c685tqsQfTqu5FTpj6M4DOIxOLhlQH2yuJrMqe3A==
</PRE>
<ADDRESS>
</ADDRESS>
</BODY></HTML>
[WRN] Could not run source riddler: unexpected status code 403 received from https://riddler.io/search?q=pld:motikan2010.com&view_type=data_table
[leakix] blog.motikan2010.com
[leakix] www.motikan2010.com
[hackertarget] namatube.motikan2010.com
[hackertarget] mymacmate.motikan2010.com
[hackertarget] md-to-pdf.motikan2010.com
[hackertarget] once-tech.motikan2010.com
[hackertarget] chosensya-maker.motikan2010.com
[hackertarget] hacker-trends.motikan2010.com
[dnsdumpster] namatube.motikan2010.com
[dnsdumpster] mymacmate.motikan2010.com
[dnsdumpster] md-to-pdf.motikan2010.com
[dnsdumpster] once-tech.motikan2010.com
[dnsdumpster] chosensya-maker.motikan2010.com
[dnsdumpster] hacker-trends.motikan2010.com
[crtsh] www.motikan2010.com
[crtsh] blog.motikan2010.com
[crtsh] motikan2010.com
[crtsh] dev.motikan2010.com
[crtsh] pre.motikan2010.com
[crtsh] stg2.motikan2010.com
[crtsh] stg.motikan2010.com
[crtsh] sample.motikan2010.com
dev.motikan2010.com
pre.motikan2010.com
stg.motikan2010.com
blog.motikan2010.com
namatube.motikan2010.com
md-to-pdf.motikan2010.com
sample.motikan2010.com
www.motikan2010.com
chosensya-maker.motikan2010.com
hacker-trends.motikan2010.com
stg2.motikan2010.com
motikan2010.com
mymacmate.motikan2010.com
once-tech.motikan2010.com
[INF] Found 14 subdomains for motikan2010.com in 7 seconds 674 milliseconds
```

</div>

### 名前解決できるドメインのみ表示

　「`-active`」オプションを付けることで名前解決できるサブドメインを出力することができます。  

　名前解決の処理が追加されるため遅くなります。

<div class="md-code" style="width:100%">

```
# ./subfinder -d motikan2010.com -active

               __    _____           __
   _______  __/ /_  / __(_)___  ____/ /__  _____
  / ___/ / / / __ \/ /_/ / __ \/ __  / _ \/ ___/
 (__  ) /_/ / /_/ / __/ / / / / /_/ /  __/ /
/____/\__,_/_.___/_/ /_/_/ /_/\__,_/\___/_/

                projectdiscovery.io

[INF] Current subfinder version v2.6.0 (latest)
[INF] Loading provider config from /root/.config/subfinder/provider-config.yaml
[INF] Enumerating subdomains for motikan2010.com
once-tech.motikan2010.com
chosensya-maker.motikan2010.com
hacker-trends.motikan2010.com
md-to-pdf.motikan2010.com
namatube.motikan2010.com
mymacmate.motikan2010.com
www.motikan2010.com
blog.motikan2010.com
[INF] Found 8 subdomains for motikan2010.com in 22 seconds 520 milliseconds
```

</div>

### サブドメインのみを表示 (情報ログの出力無効)

　「`-silent`」オプションを付けることでログを出力せずにサブドメインのみを出力することができます。

<div class="md-code" style="width:100%">

```
# ./subfinder -d motikan2010.com -active -silent
namatube.motikan2010.com
hacker-trends.motikan2010.com
once-tech.motikan2010.com
www.motikan2010.com
blog.motikan2010.com
mymacmate.motikan2010.com
chosensya-maker.motikan2010.com
md-to-pdf.motikan2010.com
```

</div>

### ファイルに出力

#### 通常形式

　「`-o <ファイル名>`」オプションを付けることでサブドメインをファイルに出力することができます。

<div class="md-code" style="width:100%">

```
# ./subfinder -d motikan2010.com -o result.txt

               __    _____           __

(...省略...)

stg.motikan2010.com
[INF] Found 14 subdomains for motikan2010.com in 15 seconds 365 milliseconds

----- 出力内容
# cat result.txt
hacker-trends.motikan2010.com
dev.motikan2010.com
stg2.motikan2010.com
sample.motikan2010.com
mymacmate.motikan2010.com
www.motikan2010.com
md-to-pdf.motikan2010.com
chosensya-maker.motikan2010.com
pre.motikan2010.com
motikan2010.com
namatube.motikan2010.com
once-tech.motikan2010.com
stg.motikan2010.com
blog.motikan2010.com
```

</div>

#### JSON形式

　「`-oJ`」オプションを付けることでJSON形式でファイルに出力することができます。  

厳密に言うとサブドメイン単位でJSON形式で出力されます。

<div class="md-code" style="width:100%">

```
# ./subfinder -d motikan2010.com -o result.json -oJ

               __    _____           __

(...省略...)

{"host":"motikan2010.com","input":"motikan2010.com","source":"digitorus"}
[INF] Found 14 subdomains for motikan2010.com in 2 seconds 248 milliseconds

----- 出力内容
# cat result.json
{"host":"mymacmate.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"motikan2010.com","input":"motikan2010.com","source":"digitorus"}
{"host":"dev.motikan2010.com","input":"motikan2010.com","source":"crtsh"}
{"host":"pre.motikan2010.com","input":"motikan2010.com","source":"crtsh"}
{"host":"stg.motikan2010.com","input":"motikan2010.com","source":"crtsh"}
{"host":"sample.motikan2010.com","input":"motikan2010.com","source":"crtsh"}
{"host":"namatube.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"blog.motikan2010.com","input":"motikan2010.com","source":"alienvault"}
{"host":"www.motikan2010.com","input":"motikan2010.com","source":"alienvault"}
{"host":"chosensya-maker.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"hacker-trends.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"stg2.motikan2010.com","input":"motikan2010.com","source":"crtsh"}
{"host":"md-to-pdf.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"once-tech.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
```

</div>

#### IPと参照サービスも出力

　「`-cs`（出力にすべてのソースを含める）」や「`-oI` （出力にホストIPを含める）」オプションを付与することでサブドメイン以外にも「検索で利用してサービス名」や「名前解決後のIPアドレス」を出力することができます。

<div class="md-code" style="width:100%">

```
# ./subfinder -d motikan2010.com -o result.json -oJ -cs -oI -active

               __    _____           __

(...省略...)

[INF] Found 8 subdomains for motikan2010.com in 11 seconds 210 milliseconds

----- 出力内容
root@bf79c6792844:/usr/local/bin# cat result.json
{"host":"mymacmate.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"motikan2010.com","input":"motikan2010.com","source":"digitorus"}
{"host":"dev.motikan2010.com","input":"motikan2010.com","source":"crtsh"}
{"host":"pre.motikan2010.com","input":"motikan2010.com","source":"crtsh"}
{"host":"stg.motikan2010.com","input":"motikan2010.com","source":"crtsh"}
{"host":"sample.motikan2010.com","input":"motikan2010.com","source":"crtsh"}
{"host":"namatube.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"blog.motikan2010.com","input":"motikan2010.com","source":"alienvault"}
{"host":"www.motikan2010.com","input":"motikan2010.com","source":"alienvault"}
{"host":"chosensya-maker.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"hacker-trends.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"stg2.motikan2010.com","input":"motikan2010.com","source":"crtsh"}
{"host":"md-to-pdf.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"once-tech.motikan2010.com","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"namatube.motikan2010.com","ip":"172.67.175.231","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"hacker-trends.motikan2010.com","ip":"104.21.64.35","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"md-to-pdf.motikan2010.com","ip":"104.21.64.35","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"mymacmate.motikan2010.com","ip":"172.67.175.231","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"www.motikan2010.com","ip":"185.199.110.153","input":"motikan2010.com","source":"alienvault"}
{"host":"blog.motikan2010.com","ip":"54.199.90.60","input":"motikan2010.com","source":"alienvault"}
{"host":"chosensya-maker.motikan2010.com","ip":"104.21.64.35","input":"motikan2010.com","source":"dnsdumpster"}
{"host":"once-tech.motikan2010.com","ip":"104.21.64.35","input":"motikan2010.com","source":"dnsdumpster"}
```

</div>

## なぜ名前解決できないサブドメインが表示されたのか

　最初に subfinder を実行した時に利用していないサブドメインが収集されました。  

　これはどうしてでしょうか？  

　このサブドメインは「`crtsh`」というサービスから収集されているようです。  

<div class="md-code" style="width:100%">

```
# ./subfinder -d motikan2010.com -v

(...省略...)

[crtsh] www.motikan2010.com
[crtsh] blog.motikan2010.com
[crtsh] motikan2010.com
[crtsh] dev.motikan2010.com
[crtsh] pre.motikan2010.com
[crtsh] stg2.motikan2010.com
[crtsh] stg.motikan2010.com
[crtsh] sample.motikan2010.com
```

</div>


　crt.sh（`https://crt.sh/`）にアクセスして、検索対象のドメインを入力します。  

[f:id:motikan2010:20230701150412p:plain:w600]  


　結果を見ると、このサービスは SSL/TLS 証明書 の内容が確認できるサービスのようです。

　2018年に SSL/TLS 証明書 の情報が subfinder に返却されているため、現在利用していないサブドメインが収集されるようになっていました。  

[f:id:motikan2010:20230701151043p:plain:w600]  


[https://crt.sh/?id=951780863]

## まとめ

　subfinder 以外のサブドメイン列挙ツールを試してみましたが一番良さそうなものが subfinder でしたので紹介してみました。  

　名前解決するオプション(`-active`)は実行時にはほぼ必須といっても良いのではないでしょうか。

## 参考

- Subdomain enumeration tools and techniques  
https://www.ceeyu.io/resources/blog/subdomain-enumeration-tools-and-techniques


