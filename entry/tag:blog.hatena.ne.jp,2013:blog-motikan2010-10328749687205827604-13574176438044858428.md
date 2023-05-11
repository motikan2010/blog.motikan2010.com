<div style="text-align:center;">
[f:id:motikan2010:20211221182408g:plain]
</div>

　**⚠️本記事で紹介するプログラムは教育目的です。<span style="color: #ff0000">実環境でのLog4Shell対策に利用しないでください！</span>**

<div class="contents-box"><p>[:contents]</p></div>

## はじめに

　Log4Shell(CVE-2021-44228)の脆弱性に対してのRASP(Runtime Application Self-Protection)を実装する方法を紹介していきます。  

　　　　　**RASP is ナニ? なひとむけ。**

### RASPとは

　RASPとは「Runtime Application Self-Protection」の略称であり、アプリケーションに組み込むことでアプリケーションをよりセキュアにするソフトウェアです。  
　<span style="color: #ff0000">本記事ではJava製のWebアプリケーションに対して「`javaagent`」として組み込みます。</span>

　Java以外だと、各言語に応じてPythonの場合はpipライブラリとして提供されていたり、PHPの場合だと拡張モジュールとして提供されていたりします。

　 OSSだと「OpenRASP(Baidu社)」が有名です。

## 開発・検証環境

　以下のバージョンで動作検証を行いました。

- Java (AdoptOpenJDK 11.0.8)
  - WebアプリケーションもRASPもこのバーションを利用します。
- Spring Boot 2.6.1
  - 攻撃対象となるWebアプリケーション
- Javassist 3.23.1-GA
  - バイトコードを操作するためにライブラリ。RASPを実装する際に利用します。
  - バイトコードを操作するライブラリとしては Byte BuddyやASM などもあります

<!-- more -->

## 実装

　脆弱なWebアプリケーションとRASPの実装の一部を記述します。  

　両ソースコードは以下のリポジトリにあります。  
[https://github.com/motikan2010/RASP-CVE-2021-44228:embed:cite]

### 脆弱アプリケーション(Spring Boot)側の実装

　Log4Shellの脆弱性を含んでいるアプリケーション側の実装は以下の通りです。  

　`X-Api-Version`ヘッダの値が`Logger#info`に渡される渡されるようになっています。

　値に攻撃文字列を指定することで、攻撃が成立するようになっています。  

<div class="md-code" style="width:100%">
```java
@RestController
public class MainController {

    private static final Logger logger = LogManager.getLogger("MainController");

    @GetMapping("/")
    public String get(@RequestHeader("X-Api-Version") String apiVersion) {
        logger.info("Received a request for API version " + apiVersion);
        return "Hello, world!";
    }

}
```
</div>

　`Log4j`は脆弱なバージョンである`2.14.1`を利用しています。
<div class="md-code" style="width:100%">
```
dependencies {
	(...snip...)
	implementation group: 'org.apache.logging.log4j', name: 'log4j-api', version: '2.14.1'
	implementation group: 'org.apache.logging.log4j', name: 'log4j-core', version: '2.14.1'
}
```
</div>

#### 動作確認 (RASP適用前)

　RASPを適用前に動作確認を行います。攻撃が成功することが確認できます。

<div class="md-code" style="width:100%">
```
==================== アプリケーションを起動 ====================
$ java -jar build/libs/web_application-0.0.1-SNAPSHOT.jar
(...snip...)
2021-12-21 20:38:15.353  INFO 44920 --- [           main] o.s.b.w.e.t.TomcatWebServer              : Tomcat started on port(s): 8080 (http) with context path ''
2021-12-21 20:38:15.363  INFO 44920 --- [           main] c.e.d.DemoApplication                    : Started DemoApplication in 1.896 seconds (JVM running for 2.912)


==================== 攻撃リクエストを送信 ====================
$ curl 127.0.0.1:8080 -H 'X-Api-Version: ${jndi:ldap://127.0.0.1:1389/a}'


==================== アプリケーション側のログ ====================
2021-12-21 20:38:15.353  INFO 44920 --- [           main] o.s.b.w.e.t.TomcatWebServer              : Tomcat started on port(s): 8080 (http) with context path ''
2021-12-21 20:38:15.363  INFO 44920 --- [           main] c.e.d.DemoApplication                    : Started DemoApplication in 1.896 seconds (JVM running for 2.912)
12月 21, 2021 8:39:07 午後 org.apache.catalina.core.ApplicationContext log
情報: Initializing Spring DispatcherServlet 'dispatcherServlet'
2021-12-21 20:39:07.962  INFO 45042 --- [nio-8080-exec-1] o.s.w.s.DispatcherServlet                : Initializing Servlet 'dispatcherServlet'
2021-12-21 20:39:07.964  INFO 45042 --- [nio-8080-exec-1] o.s.w.s.DispatcherServlet                : Completed initialization in 1 ms
-------HACKED------　⭐️　攻撃が成功している ⭐️
2021-12-21 20:39:07.995  INFO 45042 --- [nio-8080-exec-1] MainController                           : Received a request for API version javax.el.ELProcessor@b4b9e7a
```
</div>

### RASP の実装

　次にRASPを実装していきます。検証用ですので、必要最小限の実装になっています。

#### AgentMain クラス

　javaagentと読み込まれた際のエンドポイントとして`premain`メソッドを定義しています。  
<div class="md-code" style="width:100%">
```java
package com.example.cve_2021_44228_rasp;

import java.lang.instrument.Instrumentation;

public class AgentMain {

    public static void premain(String agentArg, Instrumentation inst) {
        System.out.println("Loading CVE-2021-44228 RASP");
        inst.addTransformer(new JndiLookupTransformer());
    }

}
```
</div>

#### JndiLookupTransformer クラス

　前述のpremainから呼ばれるクラスであり、本記事ではこのクラスが一番重要です。

　主に以下の処理を行うプログラムです。

- ①「`org.apache.logging.log4j.core.lookup.JndiLookupクラス`」の
- ②「`lookup`」メソッドの最初に
- ③「`/`を`x`」に置換する処理を追加

　<span style="color: #ff0000">例えば、`X-Api-Version`ヘッダに攻撃リクエストに含まれている文字列である「`ldap://127.0.0.1:1389/a`」が指定された場合に、「`ldap:xx127.0.0.1:1389xa`」に変更することで攻撃が成功しないようにしています。</span>  

　今回は置換で対応していますが、処理を中断するために例外を投げるなどの処理でも問題ないと思います。  

<div class="md-code" style="width:100%">
```java
package com.example.cve_2021_44228_rasp;

import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;
import javassist.LoaderClassPath;

import java.io.ByteArrayInputStream;
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;

public class JndiLookupTransformer implements ClassFileTransformer {

    public byte[] transform(ClassLoader loader,
                            String className,
                            Class<?> classBeingRedefined,
                            ProtectionDomain protectionDomain,
                            byte[] classfileBuffer) throws IllegalClassFormatException {

        if ( className.equals("org/apache/logging/log4j/core/lookup/JndiLookup") ) { // ① org.apache.logging.log4j.core.lookup.JndiLookup クラスを対象に
            try {
                ClassPool classPool = new ClassPool();
                classPool.appendSystemPath();
                classPool.appendClassPath(new LoaderClassPath(loader));

                ByteArrayInputStream stream = new ByteArrayInputStream(classfileBuffer);
                CtClass ctClass = classPool.makeClass(stream);

                CtMethod ctMethod = ctClass.getDeclaredMethod("lookup"); // ② lookup メソッドを対象に
                // 以下の処理を追加
                ctMethod.insertBefore(
                        "System.out.println(\"Before : \" + key);" + // 無害化処理の前
                            "key = com.example.cve_2021_44228_rasp.JndiLookupTransformer.sanitizing(key);" + // ③ 無害化処理 (下記メソッド参照)
                            "System.out.println(\"After : \" +key);" // 無害化処理の後
                );
                return ctClass.toBytecode();
            } catch ( Exception ex ) {
                ex.printStackTrace();
            }
        }

        return null;
    }

    /**
     * 無害化
     */
    public static String sanitizing(String key) {
        System.out.println("JndiLookupTransformer#sanitizing");
        return key.replace('/', 'x'); // 「/」を「x」に変換
    }

}
```
</div>

#### RASPロード時の処理のイメージ

　今回処理を追加している「`JndiLookup#lookup`」メソッドの処理内容は以下になっています。
[f:id:motikan2010:20211221185946p:plain:w600]

　RASPをロードすることによって、以下の処理内容のようになります。
[f:id:motikan2010:20211221185950p:plain:w600]


#### 動作確認 (RASP適用後)

　最後にRASPを「`-javaagent`」オプションで読み込んでWebアプリケーションを起動します。

　先ほどと同様に攻撃を行ってみると、`javax.naming.CommunicationException`が発生し攻撃が失敗していることが確認できます。

<div class="md-code" style="width:100%">
```
$ java -javaagent:./lib/cve_2021_44228_rasp-jar-with-dependencies.jar -jar build/libs/web_application-0.0.1-SNAPSHOT.jar
Loading CVE-2021-44228 RASP ⭐️　RASPが適用されていることが確認できる　⭐️
(...snip...)
2021-12-21 20:51:12.335  INFO 45330 --- [           main] o.s.b.w.e.t.TomcatWebServer              : Tomcat started on port(s): 8080 (http) with context path ''
2021-12-21 20:51:12.347  INFO 45330 --- [           main] c.e.d.DemoApplication                    : Started DemoApplication in 2.02 seconds (JVM running for 3.163)


==================== 攻撃リクエストを送信 ====================
$ curl 127.0.0.1:8080 -H 'X-Api-Version: ${jndi:ldap://127.0.0.1:1389/a}'


==================== アプリケーション側のログ ====================
12月 21, 2021 8:51:59 午後 org.apache.catalina.core.ApplicationContext log
情報: Initializing Spring DispatcherServlet 'dispatcherServlet'
2021-12-21 20:51:59.158  INFO 45330 --- [nio-8080-exec-1] o.s.w.s.DispatcherServlet                : Initializing Servlet 'dispatcherServlet'
2021-12-21 20:51:59.159  INFO 45330 --- [nio-8080-exec-1] o.s.w.s.DispatcherServlet                : Completed initialization in 1 ms
Before : ldap://127.0.0.1:1389/a
JndiLookupTransformer#sanitizing
After : ldap:xx127.0.0.1:1389xa ⭐️　入力値が変わり、「javax.naming.CommunicationException」が発生している　⭐️
2021-12-21 20:51:59,226 http-nio-8080-exec-1 WARN Error looking up JNDI resource [ldap:xx127.0.0.1:1389xa]. javax.naming.CommunicationException: localhost:389 [Root exception is java.net.ConnectException: Connection refused (Connection refused)]
	at java.naming/com.sun.jndi.ldap.Connection.<init>(Connection.java:237)
	at java.naming/com.sun.jndi.ldap.LdapClient.<init>(LdapClient.java:137)
	at java.naming/com.sun.jndi.ldap.LdapClient.getInstance(LdapClient.java:1610)
	at java.naming/com.sun.jndi.ldap.LdapCtx.connect(LdapCtx.java:2751)
	at java.naming/com.sun.jndi.ldap.LdapCtx.<init>(LdapCtx.java:319)
	at java.naming/com.sun.jndi.url.ldap.ldapURLContextFactory.getUsingURLIgnoreRootDN(ldapURLContextFactory.java:60)
	at java.naming/com.sun.jndi.url.ldap.ldapURLContext.getRootURLContext(ldapURLContext.java:61)
	at java.naming/com.sun.jndi.toolkit.url.GenericURLContext.lookup(GenericURLContext.java:204)
	at java.naming/com.sun.jndi.url.ldap.ldapURLContext.lookup(ldapURLContext.java:94)
	at java.naming/javax.naming.InitialContext.lookup(InitialContext.java:409)
	at org.apache.logging.log4j.core.net.JndiManager.lookup(JndiManager.java:172)
	at org.apache.logging.log4j.core.lookup.JndiLookup.lookup(JndiLookup.java:56)
(...snip...)
2021-12-21 20:51:59.191  INFO 45330 --- [nio-8080-exec-1] MainController                           : Received a request for API version ${jndi:ldap://127.0.0.1:1389/a}
```
</div>

　「`ldap:xx127.0.0.1:1389xa`」と通信を試みるようになり攻撃が失敗しています。  


### WAFをバイパスするような文字列を送信

　試しにWAFをバイパスするような文字列を送信してみます。

　アプリケーションのログに「`Before : LDap:ldap://127.0.0.1:1389/a`」が表示されていることが分かります。

　<span style="color: #ff0000">つまりRASPではJNDI Lookupのパース（解析）後の値をもとに攻撃文字列であるかを判定することが可能です。</span>

　この結果もRASPならではの面白い結果だと思います。

<div class="md-code" style="width:100%">
```
==================== 攻撃リクエストを送信 ====================
curl 127.0.0.1:8080  -H 'X-Api-Version: ${${upper:j}ndi:${upper:l}${upper:d}a${lower:p}:ldap://127.0.0.1:1389/a}'

==================== アプリケーション側のログ ====================
Before : LDap:ldap://127.0.0.1:1389/a
JndiLookupTransformer#sanitizing
After : LDap:ldap:xx127.0.0.1:1389xa
2021-12-22 20:16:13,901 http-nio-8080-exec-2 WARN Error looking up JNDI resource [LDap:ldap:xx127.0.0.1:1389xa]. javax.naming.NoInitialContextException: Need to specify class name in environment or system property, or in an application resource file: java.naming.factory.initial
	at java.naming/javax.naming.spi.NamingManager.getInitialContext(NamingManager.java:691)
	at java.naming/javax.naming.InitialContext.getDefaultInitCtx(InitialContext.java:305)
(...snip...)
```
</div>

## まとめ

<div style="text-align:center;">
[f:id:motikan2010:20211225194558g:plain]
</div>


- RASPでLog4Shellを対応する時代くるかも
  - 追記: とっくにきてた → <span><a href="https://www.contrastsecurity.com/security-influencers/waf-rasp-and-log4shell" target="_blank">WAF, RASP and Log4Shell</a></span>
  - <span><a href="https://github.com/Contrast-Security-OSS/safelog4j" target="_blank">safelog4j</a></span> (Contrast Security社) というLog4Shel用のRASPも存在しています
    - こちらはバイトコード操作に Byte Buddy を用いている
- WAFと違って**Webアプリケーション以外のアプリケーションにも適用可能**なので便利かも
- プロテクションしたい関数(メソッド)の実行時にのみ動作するセキュリティ機能なので**軽量**かも
- RASPの実装に興味持った方はBaidu社がOSSとして開発しているOpenRASP(Java・PHP向けがある)を見てみてもいいかも
- RASP業界はこんな感じかも ↓
[https://twitter.com/motikan2010/status/1361135810406281217:embed]



## 更新履歴

- 2021年12月21日 新規作成
