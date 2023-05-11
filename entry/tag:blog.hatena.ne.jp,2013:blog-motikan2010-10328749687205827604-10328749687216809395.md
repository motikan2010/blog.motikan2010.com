<div style="text-align: center;">[f:id:motikan2010:20201204200444p:plain:w600]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

本記事は、  
・<b>bcrypt.GenerateFromPassword</b>  
・<b>bcrypt.CompareHashAndPassword</b>  
を使ってみる話です。  

　Go言語を使ったWeb開発で認証機能を実装したくて調べてみたら、「sessionauth」というパッケージがありました。  
試しに sessionauth を追加って認証機能を実装してみることにする。

[https://github.com/martini-contrib/sessionauth:embed:cite]  

　認証の有無のセッションを管理してくれるのは便利なのだが、パスワード格納(DBへの保存)の機能は提供されていないので、その部分は自ら実装する必要がありそう。  

## 「sessionauth」を使ってのパスワード保存状態

　現に sessionauth のサンプルコードではパスワードは平文で保存されていました。これではイカン!!
<div class="md-code">
```
$ sqlite3 martini-sessionauth.bin
SQLite version 3.16.0 2016-11-04 19:09:39
Enter ".help" for usage hints.
sqlite> .table
users
sqlite> select * from users;
1|testuser|password
```
</div>

<!-- more -->


　そんな訳で、パスワードの格納の定番と言えば、『<b>ハッシュ化</b>』ということで、Go言語でのパスワードハッシュに関して調べてみた。  
下記のサイトが参考になった。

[http://hachibeechan.hateblo.jp/entry/2015/03/11/Go%E8%A8%80%E8%AA%9E%E3%81%A7%E3%83%91%E3%82%B9%E3%83%AF%E3%83%BC%E3%83%89%E3%81%A8%E3%81%8B%E3%81%AE%E3%83%8F%E3%83%83%E3%82%B7%E3%83%A5%E5%8C%96:embed:cite]

　ハッシュ化のライブラリには主に２つ用意されているらしい。  

- scrypt  
- bcrypt

　『bcrypt』を使ってハッシュ化する方法を紹介します。

[https://godoc.org/golang.org/x/crypto/bcrypt:title]

### パスワードのハッシュ化

　<b>GenerateFromPassword関数</b>を利用することで、パスワード文字列をByte型のハッシュ値に変換することができます。  

<div class="md-code">
```go
// パスワードのハッシュ化
hash, err := bcrypt.GenerateFromPassword([]byte("パスワード"), bcrypt.DefaultCost)

// Byteで返されるので文字列に変換して表示
fmt.Println(string(hash))

// 毎回値の異なるハッシュ値が取得できる(ちなみに"password"のハッシュ値)
// $2a$10$iuJaubQvGTawiwa6UFa08uvOGwFaa25Wz29llEKEFHyPT3w262Qw6
// $2a$10$HFZ4bmj98bEePKO3gNsbZO3XsgXORvjFhexZV6HADm46/CuaE6M/m
// $2a$10$BSzyPPKOOs0YwC1h6UoD2eNFAyWYVfS.hmZQuQLLTRyC/Z.z3fzsy
```
</div>

#### パスワード文字列とハッシュ値を比較

　「ハッシュ値」と「パスワード文字列」の比較は<b>CompareHashAndPassword関数</b>を利用することで可能です。

<div class="md-code">
```go
// 一致している場合はerrにnilが返されます。一致していない場合はエラーが返されます。
err = bcrypt.CompareHashAndPassword([]byte("ハッシュ値"), []byte("パスワード"))
```
</div>

#### ハッシュ値の生成・比較を1つのコードにまとめる

　下記のコードでは「Success」が表示されます。
<div class="md-code" style="width:100%">
```go
package main

import (
	"fmt"
	"golang.org/x/crypto/bcrypt"
)

func main() {
	storePass := "password"
	inputPass := "password"

	// ハッシュ値の生成
	hash, err := bcrypt.GenerateFromPassword([]byte(storePass), bcrypt.DefaultCost)
	if err != nil {
		return
	}

	hashStr := string(hash)

	// ハッシュ値とパスワード文字列を比較
	err = bcrypt.CompareHashAndPassword([]byte(hashStr), []byte(inputPass))
	if err != nil {
		fmt.Println("Failure")
	} else {
		fmt.Println("Success")
	}

}
```
</div>

### 認証のデモ

　パスワード認証機能が正常に動作確認をすることが目的のため、今回はデータベースなどは用意せずにログインIDとパスワードを格納することができるUser構造体を使って動作確認を行います。

下記が今回使ったコードとなります。
<div class="md-code">
```go
package main

import (
	"fmt"
	"time"

	"golang.org/x/crypto/bcrypt"
)

type User struct {
	LoginId  string
	Password string
}

type Users []*User

var users Users

func register(loginId, pass string) {
	/*
	   bcrypt.MinCost = 4
	   bcrypt.MaxCost = 31
	   bcrypt.DefaultCost = 10
	*/
	hash, err := bcrypt.GenerateFromPassword([]byte(pass), bcrypt.DefaultCost)
	if err != nil {
		return
	}
	users = append(users, &User{LoginId: loginId, Password: string(hash)})
}

func login(loginId, password string) {
	var hashStr = ""
	start := time.Now()
	for _, user := range users {
		if loginId == user.LoginId {
			hashStr = user.Password
			break
		}
	}
	err := bcrypt.CompareHashAndPassword([]byte(hashStr), []byte(password))
	end := time.Now()
	fmt.Printf("%fs\t", (end.Sub(start)).Seconds())
	if err == nil {
		// 成功
		fmt.Print("Success")
	} else {
		// 失敗
		fmt.Print("Failure")
	}
	fmt.Printf("\t%s/%s\n", loginId, password)
}

func main() {
	users = Users{}

	// 登録
	register("user1", "password1")
	register("user2", "password2")
	register("user3", "password3")
	register("user4", "password4")
	register("user5", "password5")

	// 認証
	login("user1", "password1")
	login("user2", "password2")
	login("user3", "password3")
	login("user4", "password4")
	login("user5", "password5")
	login("user6", "password1")
	login("user1", "")
	login("user3", "password1")
	login("user3", "password2")
	login("user3", "password3")
	login("user3", "password4")
}
```
</div>

##### 出力結果

<div class="md-code">
```
0.099179s	Success	user1/password1
0.094606s	Success	user2/password2
0.092299s	Success	user3/password3
0.092068s	Success	user4/password4
0.094260s	Success	user5/password5
0.000001s	Failure	user6/password1  // ← 他に比べて認証時間短い!?
0.095547s	Failure	user1/
0.092764s	Failure	user3/password1
0.090236s	Failure	user3/password2
0.089934s	Success	user3/password3
0.091101s	Failure	user3/password4
```
</div>

　正常に認証処理ができていることが確認できました。

　しかし、認証時間を着目してみると、ログインIDに「user6」を指定した時だけ処理時間が異様に短いことが分かります。  
　存在しないユーザIDであり、保存されているハッシュ値が格納されるようになっている hashStr変数 に空文字が入っており「CompareHashAndPassword」の処理が短くなっているからだと思われる。  

　細かい点だが、『<span class="m-y">レスポンス時間からユーザの有無が知られるのはセキュリティ上良くない</span>』ので少し修正してみます。  

　空文字の初期化ではなく、文字列を格納するような初期化を行ってみる。

<div class="md-code">
```go
var hashStr = "$2a$10$XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
```
</div>

　出力結果は下記のようになり、ユーザの有無に関係なく、認証処理時間に偏りがなくなったことが確認できました。  
<div class="md-code">
```go
0.097851s	Success	user1 / password1
0.095720s	Success	user2 / password2
0.097851s	Success	user3 / password3
0.097147s	Success	user4 / password4
0.104292s	Success	user5 / password5
0.099853s	Failure	user6 / password1
0.098755s	Failure	user1 /
0.094102s	Failure	user3 / password1
0.093256s	Failure	user3 / password2
0.100131s	Success	user3 / password3
0.097537s	Failure	user3 / password4
```
</div>

## 更新履歴

- 2017年 2月13日 新規作成
