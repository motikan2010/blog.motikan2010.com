## 動作確認

### サンプルアプリケーションの動作確認

#### シリアライズ後の値を取得

```
$ curl 'http://127.0.0.1:8080/?name=TaroYamada&age=30'

rO0ABXNyADZjb20uZXhhbXBsZS5pbnNlY3VyZV9kZXNlcmlhbGl6YXRpb24uY29udHJvbGxlci5QZXJzb26eHGFQi1tVeAIAAkkAA2FnZUwABG5hbWV0ABJMamF2YS9sYW5nL1N0cmluZzt4cAAAAB50AApUYXJvWWFtYWRh
```


```
$echo -n 'rO0ABXNyADZjb20uZXhhbXBsZS5pbnNlY3VyZV9kZXNlcmlhbGl6YXRpb24uY29udHJvbGxlci5QZXJzb26eHGFQi1tVeAIAAkkAA2FnZUwABG5hbWV0ABJMamF2YS9sYW5nL1N0cmluZzt4cAAAAB50AApUYXJvWWFtYWRh' | base64 -D | hexdump -C

00000000  ac ed 00 05 73 72 00 36  63 6f 6d 2e 65 78 61 6d  |....sr.6com.exam|
00000010  70 6c 65 2e 69 6e 73 65  63 75 72 65 5f 64 65 73  |ple.insecure_des|
00000020  65 72 69 61 6c 69 7a 61  74 69 6f 6e 2e 63 6f 6e  |erialization.con|
00000030  74 72 6f 6c 6c 65 72 2e  50 65 72 73 6f 6e 9e 1c  |troller.Person..|
00000040  61 50 8b 5b 55 78 02 00  02 49 00 03 61 67 65 4c  |aP.[Ux...I..ageL|
00000050  00 04 6e 61 6d 65 74 00  12 4c 6a 61 76 61 2f 6c  |..namet..Ljava/l|
00000060  61 6e 67 2f 53 74 72 69  6e 67 3b 78 70 00 00 00  |ang/String;xp...|
00000070  1e 74 00 0a 54 61 72 6f  59 61 6d 61 64 61        |.t..TaroYamada|
0000007e
```

[3-3. シリアル化と情報漏洩](https://www.ipa.go.jp/security/awareness/vendor/programmingv1/a03_03.html)

> 最初の２バイト「AC ED」はシリアル化ストリームに固有のマジックナンバー，次の２バイト「00 05」はシリアル化仕様のバージョン番号である。


#### デシリアライズの確認


```
$ echo -n 'rO0ABXNyADZjb20uZXhhbXBsZS5pbnNlY3VyZV9kZXNlcmlhbGl6YXRpb24uY29udHJvbGxlci5QZXJzb26eHGFQi1tVeAIAAkkAA2FnZUwABG5hbWV0ABJMamF2YS9sYW5nL1N0cmluZzt4cAAAAB50AApUYXJvWWFtYWRh' | \
base64 -D | \
curl 'http://127.0.0.1:8080/' -X POST --data-binary @-

「TaroYamada」30歳
```

### どのように攻撃するのか


[java.lang.Runtime.exec() Payload Workarounds - @Jackson_T](http://www.jackson-t.ca/runtime-exec-payloads.html)

Mac OS環境の場合は「`base64 -D`」を指定します。

#### PoC生成ツール「ysoserial」

[frohoff/ysoserial: A proof-of-concept tool for generating payloads that exploit unsafe Java object deserialization.](https://github.com/frohoff/ysoserial)  
　PHPでいう「PHPGGC」です。

[ysoserial/CommonsCollections2.java at 30099844c60958e10f60749dfad6311f3e732d3d · frohoff/ysoserial](https://github.com/frohoff/ysoserial/blob/30099844c60958e10f60749dfad6311f3e732d3d/src/main/java/ysoserial/payloads/CommonsCollections2.java)

[commons-collections/TransformingComparator.java at collections-4.1 · apache/commons-collections](https://github.com/apache/commons-collections/blob/collections-4.1/src/main/java/org/apache/commons/collections4/comparators/TransformingComparator.java)

```
compile group: 'org.apache.commons', name: 'commons-collections4', version: '4.0'
```

「`commons-collections4`」を依存ライブラリとして追加していない場合に発生するエラー。
```
java.lang.ClassNotFoundException: org.apache.commons.collections4.comparators.TransformingComparator
```