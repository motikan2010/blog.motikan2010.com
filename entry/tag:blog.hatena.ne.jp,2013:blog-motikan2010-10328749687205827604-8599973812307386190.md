<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　Specificationインタフェースを利用した副問合せに関して、あまり情報が見当たらなかったので、メモ程度に紹介します。

## 実装

### サンプルテーブル

　本記事では以下のサンプルテーブルを例に説明していきます。  
<span class="m-y">「1対多」の関係を例に説明していきます。</span>  ユーザが1、ツイートが多となっています。

[f:id:motikan2010:20171013000832p:plain:w400]  

### コード

　本記事での目的は「同じ内容のツイートを3回以上しているユーザ」を選択するという少し面倒なSQL文を発行するというものです。  
　ネイティブクエリを利用すれば簡単ですが、Specificationで副問合せのSQLを発行するやり方です。

[f:id:motikan2010:20171014134003j:plain]  

<!-- more -->

#### ファイル構成

```
 ── dbapp
    ├── DbappApplication.java
    ├── model
    │   ├── Tweet.java
    │   ├── Tweet_.java
    │   ├── User.java
    │   └── User_.java
    ├── repository
    │   ├── TweetRepository.java
    │   └── UserRepository.java
    └── spec
        └── BadUserSpec.java
```

#### Specification

　発行クエリを生成する部分です。ここで副問合せとなるSQLを生成しますので、本記事で一番重要な部分です。

##### BadUserSpec.java

<div class="sm-code">
```java
package com.motikan2010.dbapp.spec;

import com.motikan2010.dbapp.model.Tweet;
import com.motikan2010.dbapp.model.Tweet_;
import com.motikan2010.dbapp.model.User;
import com.motikan2010.dbapp.model.User_;
import org.springframework.data.jpa.domain.Specification;

import javax.persistence.criteria.*;
import java.util.ArrayList;
import java.util.List;

public class BadUserSpec implements Specification<User> {

    @Override
    public Predicate toPredicate(Root<User> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
        final List<Predicate> predicates = new ArrayList<>();

        // サブクエリ
        Subquery<Tweet> subquery = query.subquery(Tweet.class);
        Root<Tweet> subRoot = subquery.from(Tweet.class);
        subquery.select(subRoot.get(Tweet_.user.getName()));
        subquery.where(cb.equal(root.get(User_.id.getName()), subRoot.get(Tweet_.user.getName())));
        // ツイート内容でグループ化
        subquery.groupBy(subRoot.get(Tweet_.body.getName()));
        // 条件 
        subquery.having(cb.and(
                cb.greaterThanOrEqualTo(cb.count(subRoot), 3L)
        ));

        predicates.add(cb.exists(subquery));

        return cb.and(predicates.toArray(new Predicate[predicates.size()]));
    }

}
```
</div>

#### エンティティ（モデル）

##### User.java
<div class="sm-code">
```java
package com.motikan2010.dbapp.model;

import lombok.Data;

import javax.persistence.*;
import java.util.List;

@Entity
@Data
@Table(name = "user")
public class User {

    @Id
    @GeneratedValue
    @Column(name = "id")
    private int id;
    
    @Column(name = "nickname")
    private String nickname;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Tweet> tweetList;

}
```
</div>

##### Tweet.java
<div class="sm-code">
```java
package com.motikan2010.dbapp.model;

import com.sun.istack.internal.NotNull;
import lombok.Data;

import javax.persistence.*;

@Entity
@Data
@Table(name = "tweet")
public class Tweet {

    @Id
    @GeneratedValue
    @Column(name = "id")
    private int id;
    
    @Column(name = "body")
    private String body;

    @NotNull
    @Column(name = "user_id")
    private int userId;
    
    @ManyToOne(targetEntity=User.class)
    @JoinColumn(name = "user_id", referencedColumnName = "id", insertable=false, updatable=false)
    private User user;

}
```
</div>

#### メタモデル

　メタモデルを作成していきます。（恥ずかしながら最近メタモデルの存在を知りました）  

[https://cloudear.jp/blog/?p=2101:title]  

　ちなみにエンティティと同じパッケージに所属させないといけないらしい。  
別のパッケージに配置し、ヌルポと格闘したのはいい思い出・・・。

[https://stackoverflow.com/questions/3854687/jpa-hibernate-static-metamodel-attributes-not-populated-nullpointerexception:embed:cite]

##### User_.java

<div class="sm-code">
```java
package com.motikan2010.dbapp.model;

import javax.persistence.metamodel.ListAttribute;
import javax.persistence.metamodel.SingularAttribute;
import javax.persistence.metamodel.StaticMetamodel;

@StaticMetamodel(User.class)
public class User_ {
    public static volatile SingularAttribute<User, Integer> id;
    public static volatile SingularAttribute<User, String> nickname;
    public static volatile ListAttribute<User, Tweet> tweetList;
}
```
</div>

##### Tweet_.java

<div class="sm-code">
```java
package com.motikan2010.dbapp.model;

import javax.persistence.metamodel.SingularAttribute;
import javax.persistence.metamodel.StaticMetamodel;

@StaticMetamodel(Tweet.class)
public class Tweet_ {
    public static volatile SingularAttribute<Tweet, Integer> id;
    public static volatile SingularAttribute<Tweet, String> body;
    public static volatile SingularAttribute<Tweet, User> user;
}
```
</div>

#### リポジトリ

##### UserRepository.java

　中身は空っぽで問題ないです。

<div class="sm-code">
```java
package com.motikan2010.dbapp.repository;

import com.motikan2010.dbapp.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;

public interface UserRepository extends JpaRepository<User, Integer>, JpaSpecificationExecutor<User>{
}
```
</div>


#### 呼び出し側

##### DbappApplication.java

<div class="sm-code">
```java
package com.motikan2010.dbapp;

import com.motikan2010.dbapp.model.User;
import com.motikan2010.dbapp.repository.UserRepository;
import com.motikan2010.dbapp.spec.BadUserSpec;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.util.List;

@SpringBootApplication
public class DbappApplication implements CommandLineRunner {

    @Autowired
    UserRepository userRepo;

    public static void main(String[] args) {
        SpringApplication.run(DbappApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {

        BadUserSpec spec = new BadUserSpec();
        List<User> userList = userRepo.findAll(spec);

        for(User user : userList){
            System.out.println(user.getId() + " : " + user.getNickname());
        }

    }

}
```
</div>

### 発行SQL

<div class="sm-code">
```sql
select user0_.id as id1_1_, user0_.nickname as nickname2_1_ from user user0_ where exists (select tweet1_.user_id from tweet tweet1_, user user2_ where tweet1_.user_id=user2_.id and user0_.id=tweet1_.user_id group by tweet1_.body having count(tweet1_.id)>=3)
```
</div>

　分かりやすく & 整形すると以下のようになります。
<div class="sm-code">
```sql
select id , nickname 
from user user1 
where exists (
    select user_id 
    from tweet, user user2 
    where tweet.user_id = user2.id and user1.id = tweet.user_id 
    group by body 
    having count(tweet.id)>=3
)
```
</div>

　想定通りに Specificationを利用した副問合せができています。  
countの部分をsumやavgに変えるなどして様々なパターンに応用できそうです。


## 更新履歴
- 2017年10月13日 新規作成
