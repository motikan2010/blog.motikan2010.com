<div style="text-align: center;">[f:id:motikan2010:20180120011546j:plain:w500]</div>  

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　JSONのパース処理はいろんな場面で利用していますが、処理速度を意識して利用していなかった。  
そこでJSONパーサーライブラリの処理速度を比べてみて、いざという時に備えておく。  

## 比べるライブラリは下記の2種類

### Gson

[f:id:motikan2010:20200324191247p:plain]

[https://github.com/google/gson:embed:cite]

### Jackson

[f:id:motikan2010:20200324191243p:plain]

[https://github.com/FasterXML/jackson:embed:cite]

## 下記のパターン（特徴）で比較
- ノーマル
- Date型
- 長い文字列
- Date型 ×2
- 大量のメンバ変数

<span class="m-y">結果から言うと、Jacksonが優勢であった。</span>

<!-- more -->

## 処理速度の計測方法

<div class="sm-code">
```java
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;

import java.time.Duration;
import java.time.Instant;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class SampleApplication {

    public static void main(String[] args) throws Exception{
        // 1. 30万含んだユーザリストを作成
        List<User> userList = new ArrayList<>();
        for (int i=0; i < 300_000; i++) {
            userList.add(new User1("firstName", "lastName"));
        }

        // 2. ユーザリストをJSONテキストに変換
        String jsonString = null;
        for (int i=0; i < 5;i++) {
            Instant createStart = Instant.now();
            jsonString = createJson(userList);
            System.out.println("Create JSON : " + Duration.between(createStart, Instant.now()).toNanos() / 1000000.0 + " [ms]");
        }

        // 3. JSONテキストからユーザリストを復元
        for (int i=0; i < 5;i++) {
            Instant parseStart = Instant.now();
            User user = parseJson(jsonString);
            System.out.println("Parse  JSON : " + Duration.between(parseStart, Instant.now()).toNanos() / 1000000.0 + " [ms]");
        }
    }
}
```
</div>

　それぞれのパース処理の内容は以下のようになっています。

### "Gson"を使ったJSON文字列の作成・解析

<div class="sm-code">
```java
private static String createJson (List<User> userList) {
    Gson gson = new Gson();
    String json = gson.toJson(userList);
    return json;
}

private static User parseJson (String json) {
    Gson gson = new Gson();
    List<User> userList = gson.fromJson(json, new TypeToken<List<User1>>(){}.getType());
    return userList.get(0);
}
```
</div>

### "Jackson"を使ったJSON文字列の作成・解析

<div class="sm-code">
```java
private static String createJson (List<User> userList) throws Exception{
    ObjectMapper objectMapper = new ObjectMapper();
    String json = objectMapper.writeValueAsString(userList);
    return json;
}

private static User parseJson (String json) throws Exception {
    ObjectMapper objectMapper = new ObjectMapper();
    List<User> userList = objectMapper.readValue(json, new TypeReference<List<User1>>(){});
    return userList.get(0);
}
```
</div>

　Userクラスの中身は以下のようになっており、**メンバ変数の個数**や**データ型**を変えていき、処理速度に影響しているかを確認していきます。  

<div class="sm-code">
```java
@Data
@AllArgsConstructor
public class User1 implements User {

    private String firstName;

    private String lastName;
}
```
</div>

### ノーマル

||データ型|値|
|-|-|-|
|第1引数|String|firstName|
|第2引数|String|lastName|

　JSON文字列・オブジェクトの変換処理はそれぞれ5回行い、処理毎に早くなっているが分かる。  
　だが2つの処理速度に大差は感じられない。

#### インスタンス → JSON

||Gson|Jackson|
|-|-|-|
|1回目|1082 ms|968 ms|
|2回目|697 ms|512 ms|
|3回目|665 ms|144 ms|
|4回目|349 ms|150 ms|
|5回目|324 ms|205 ms|

#### JSON → インスタンス

||Gson|Jackson|
|-|-|-|
|1回目|293 ms|386 ms|
|2回目|164 ms|160 ms|
|3回目|159 ms|104 ms|
|4回目|175 ms|132 ms|
|5回目|140 ms|101 ms|

### 特徴 : Date型

　メンバ変数にDate型を加えてパース処理を実施。
||データ型|値|
|-|-|-|
|第1引数|String|firstName|
|第2引数|String|lastName|
|第3引数|Date|new Date()|

　ここで処理速度に差が生じてきた。
Gsonは**Date型の変換が苦手**なのか、著しく処理が遅くなった事が確認できる。

#### オブジェクト → JSON文字列

||Gson|Jackson|
|-|-|-|
|1回目|<span style="color: #ff0000">2356</span> ms|<span style="color: #2196f3">1141</span> ms|
|2回目|1101 ms|618 ms|
|3回目|793 ms|277 ms|
|4回目|606 ms|180 ms|
|5回目|431 ms|253 ms|

#### JSON文字列 → オブジェクト

||Gson|Jackson|
|-|-|-|
|1回目|<span style="color: #ff0000">3845</span> ms|<span style="color: #2196f3">388</span> ms|
|2回目|3007 ms|193 ms|
|3回目|2928 ms|169 ms|
|4回目|2847 ms|163 ms|
|5回目|2823 ms|176 ms|

### 特徴 : 長い文字列
||データ型|値|
|-|-|-|
|第1引数|String|firstName|
|第2引数|String|lastName|
|第3引数|String|(500文字)|

　Jacksonが相変わらず早いが、先の例ほどは差がなく、文字長はそこまで処理時間に関係なさそう。

#### オブジェクト → JSON文字列

||Gson|Jackson|
|-|-|-|
|1回目|2578 ms|2247 ms|
|2回目|1833 ms|964 ms|
|3回目|1379 ms|860 ms|
|4回目|1315 ms|836 ms|
|5回目|1251 ms|768 ms|

#### JSON文字列 → オブジェクト

||Gson|Jackson|
|-|-|-|
|1回目|1354 ms|742 ms|
|2回目|666 ms|445 ms|
|3回目|990 ms|687 ms|
|4回目|873 ms|521 ms|
|5回目|963 ms|609 ms|

ちなみに、第３引数に600文字を指定して実行すると、"OutOfMemoryError"になった。

### 特徴 : Date型 ×2

||データ型|値|
|-|-|-|
|第1引数|String|firstName|
|第2引数|String|lastName|
|第3引数|Date|new Date()|
|第4引数|Date|new Date()|

#### オブジェクト → JSON文字列

||Gson|Jackson|
|-|-|-|
|1回目|2925 ms|989 ms|
|2回目|1477 ms|767 ms|
|3回目|840 ms|384 ms|
|4回目|842 ms|324 ms|
|5回目|877 ms|150 ms|

#### JSON文字列 → オブジェクト

||Gson|Jackson|
|-|-|-|
|1回目|6182 ms|538 ms|
|2回目|5400 ms|359 ms|
|3回目|5429 ms|201 ms|
|4回目|5378 ms|212 ms|
|5回目|5447 ms|202 ms|

### 特徴 : 大量のメンバ変数

||データ型|値|
|-|-|-|
|第1引数|String|SampleText|
|第2引数|String|SampleText|
|第3引数|String|SampleText|
|第4引数|String|SampleText|
|||(以下略)|
|第30引数|String|SampleText|

　ここにきてGsonの強みが確認された。  
gsonは処理を終えることができたが、<span class="m-y">Jacksonの場合「オブジェクト → JSON文字列」の処理で"OutOfMemoryError"</span>となった。

#### オブジェクト → JSON文字列

||Gson|Jackson|
|-|-|-|
|1回目|5338 ms|3503 ms|
|2回目|5237 ms|- ms|
|3回目|2868 ms|- ms|
|4回目|2744 ms|- ms|
|5回目|2555 ms|- ms|

#### JSON文字列 → オブジェクト

||Gson|Jackson|
|-|-|-|
|1回目|4633 ms|4854 ms|
|2回目|4089 ms|5163 ms|
|3回目|2629 ms|3910 ms|
|4回目|5575 ms|2939 ms|
|5回目|3798 ms|2981 ms|
