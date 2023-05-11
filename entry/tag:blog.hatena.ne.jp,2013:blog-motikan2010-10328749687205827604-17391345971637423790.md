<div style="text-align:center;">[f:id:motikan2010:20180422143449p:plain:w600]</div>  

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　今回はNetFlixが開発したOSSである「FlameScope」を利用して、Javaアプリケーションのパフォーマンスの可視化をしてみます。  

　最終的には上記画像のように、一定期間のCPU利用割合をメソッド単位で表示させてみます。  
<span class="m-y">このツールを使うことで、どのメソッドがボトルネックになっているのかを特定することが確認することができます。</span>  

[https://github.com/Netflix/flamescope:embed:cite]

## 検証環境

<div class="md-code" style="width:100%">
```
$ uname -a
Linux ip-172-31-23-235 4.9.81-35.56.amzn1.x86_64 #1 SMP Fri Feb 16 00:18:48 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
</div>

## 環境構築

### perf インストール

　プロファイリングツールには「<span><a href="https://ja.wikipedia.org/wiki/Perf" target="_blank">perf</a></span>」を利用します。

　初期ではインストールされていなかっため、導入します。
<div class="md-code" style="width:100%">
```
$ sudo yum -y install perf
```
</div>

### FlameScope インストール

　perfだけではプロファイリングで取得した情報を可視化することができない為、可視化する為に『FlameScope』をインストールします。  

　FlameScopeは、動画配信サービスでお馴染みであるNetflixが開発・公開しているツールです。  
NetflixがGitHub上で公開しているツールは、他にも多々あり一見する価値有りです！

[https://github.com/Netflix:title]

<div class="md-code" style="width:100%">
```
$ git clone https://github.com/Netflix/flamescope
$ cd flamescope
$ pip install -r requirements.txt
$ python run.py
```
</div>

### 検証用Javaアプリケーションの導入

　今回はSpring bootのアプリケーションを利用します。  

　１つのコントローラが存在し、その中から負荷が異なる3つのメソッドを呼び出しています。  
ここのコード自体は今回の検証にはあまり関係ありません、メソッドごとに処理の負荷が異なるということが重要です。  

- lightMethod メソッド
- normalMethod メソッド
- heavyMethod メソッド

<div class="md-code" style="width:100%">
```java
@Controller
public class MainController {
    @RequestMapping("/")
    public String mainpage(Model model) {
        lightMethod();
        normalMethod();
        heavyMethod();
        return "mainpage";
    }

    private void lightMethod() {
        long sum = 0;
        for (long i = 0; i < 1_000_000_000; i++) {
            sum = sum + i;
        }
        System.out.println(sum);
    }

    private void normalMethod() {
        long sum = 0;
        for (long i = 0; i < 1_000_000_000; i++) {
            sum = sum + i;
        }
        System.out.println(sum);
        sum = 0;
        for (long i = 0; i < 1_000_000_000; i++) {
            sum = sum + i;
        }
        System.out.println(sum);
    }

    private void heavyMethod() {
        long sum = 0;
        for (long i = 0; i < 1_000_000_000; i++) {
            sum = sum + i;
        }
        System.out.println(sum);
        sum = 0;
        for (long i = 0; i < 1_000_000_000; i++) {
            sum = sum + i;
        }
        System.out.println(sum);
        sum = 0;
        for (long i = 0; i < 1_000_000_000; i++) {
            sum = sum + i;
        }
        System.out.println(sum);
    }
}
```
</div>

### perf-map-agent インストール

　このままではJVM上で動作するアプリケーションのプロファイリング情報をメソッド単位で取得できません。([unknown]と表示されてしまいます)  
[f:id:motikan2010:20180422173925p:plain]  

　メソッド単位で表示させるため「perf-map-agent」を導入します。  

[https://github.com/jvm-profiling-tools/perf-map-agent:embed:cite]

<div class="md-code" style="width:100%">
```
$ git clone https://github.com/jvm-profiling-tools/perf-map-agent.git
$ cd ~/perf-map-agent/
$ sudo yum -y install cmake gcc-c++
$ cmake .
$ make
Scanning dependencies of target attach-main
[ 20%] Building Java objects for attach-main.jar
[ 40%] Generating out/CMakeFiles/attach-main.dir/java_class_filelist
[ 60%] Creating Java archive attach-main.jar
[ 60%] Built target attach-main
Scanning dependencies of target perfmap
[ 80%] Building C object CMakeFiles/perfmap.dir/src/c/perf-map-agent.c.o
[100%] Building C object CMakeFiles/perfmap.dir/src/c/perf-map-file.c.o
Linking C shared library out/libperfmap.so
[100%] Built target perfmap
```
</div>

## プロファイリング開始

### Javaアプリケーションの起動

<div class="md-code" style="width:100%">
```
$ java -Xmx1024m -Djava.security.egd=file:/dev/./urandom -jar demo-0.0.1-SNAPSHOT.war
```
</div>

### Javaアプリケーション用のマッピングファイル(*.map)の作成

　起動中のJavaアプリケーションのPIDを取得し、「`create-java-perf-map.sh`」の引数に渡して実行すると、`/tmp`以下に作成されます。

<div class="md-code" style="width:100%">
```
$ ps a | grep demo-0.0.1-SNAPSHOT.war
32035 pts/0    Sl+    2:17 java -Xmx1024m -Djava.security.egd=file:/dev/./urandom -jar demo-0.0.1-SNAPSHOT.war

$ cd ~/perf-map-agent/bin/
$ ./create-java-perf-map.sh 32035

$ ll /tmp/perf-32035.map
-rw-rw-r-- 1 root root 252906 Apr 22 04:24 /tmp/perf-32035.map
```
</div>

### プロファイルの実行

　30秒間プロファイルの取得しています。  
<div class="md-code" style="width:100%">
```
$ sudo perf record -F 49 -a -g -- sleep 30
[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.293 MB perf.data (1470 samples) ]

// プロファイル情報の確認
$ ls -l perf.data
-rw------- 1 root root 310164 Apr 22 04:26 perf.data

// プロファイリングを保存
$ sudo perf script --header > stacks.log

// 可視化のためにflamescopeに配置
$ mv stacks.log ~/flamescope/examples/
```
</div>

### プロファイルの結果

　CPUの使用率を「normalMethod メソッド」「heavyMethod メソッド」が大半を占めていることが分かります。  
[f:id:motikan2010:20180422183219p:plain:w600]

「 lightMethod メソッド」も少しですが、CPU使用率が確認できます。  
[f:id:motikan2010:20180422183230p:plain]

## 参考

- <span><a href="http://d.hatena.ne.jp/yohei-a/20180408/1523196368" target="_blank">Netflix のオープンソース可視化ツール FlameScope を使ってみた - ablog</a></span>
- <span><a href="http://nopipi.hatenablog.com/entry/2018/04/11/024213" target="_blank">clock_gettime()で負荷をかけたEC2をNetflix FlameScopeでのぞいてみた - のぴぴのメモ</a></span>

## 更新履歴

- 2018年4月22日 新規作成
- 2020年10月11日 整形
