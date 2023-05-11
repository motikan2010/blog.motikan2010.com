[f:id:motikan2010:20170211223100p:plain]  

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　Go言語でWebアプリ開発をしていみたいと思っていましたので、Webフレームワークを調べてみると色々なものがあった。  

[http://qiita.com/jumbOrNot/items/45f86db15a5a6c8a0622:title]  
その中で速度が速く、人気もある『Gin』を手始めにさわってみることにします。

[https://github.com/gin-gonic/gin:embed:cite]  

<!-- more -->

## 実装

　作成するものは「SQLiteを使ったTODOリストアプリ」です。  

こちらの記事を参考にして作成しました。  
[https://developers.eure.jp/tech/go_web_application_1/:title]


作成順序としては  
　ビュー  → コントローラ → モデル  
です。  

下記のコマンドでGinをインストールすることができます。
```
$ go get gopkg.in/gin-gonic/gin.v1
```

### 1. HTMLテンプレートを呼び出す

まずは「/」にアクセスしたら、「index.tmpl」を呼び出し、出力するだけのコードを書いていきます。  
```
$ vim main.go
```

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()
	router.LoadHTMLGlob("views/*")

	router.GET("/", func(c *gin.Context) {
		c.HTML(http.StatusOK, "index.tmpl", gin.H{
                    "title": "Hello Gin!",
                })
	})

	router.Run(":8080")
}

```
#### 1-1. HTMLテンプレート作成

main.goで
```
gin.H{
	"title": "Hello Gin!",
}
```
と定義しているので、テンプレート側で「{{ .title }}」と記述してレンダリングさせることができます。

```
$ mkdir views
$ vim views/index.tmpl
```

```html=
<!DOCTYPE html>
<html>
	<head>
	    <title>{{ .title }}</title>
	</head>
	<body>
		<h3>{{ .title }}</h3>
	</body>
</html>
```

ここまでできたら起動させてアクセスしてみます。

#### 1-2. 動作確認

```
$ go run main.go
```
「http://127.0.0.1:8080」にアクセスしてみます。  
[f:id:motikan2010:20170211215514p:plain]  

### 2. コントローラを作る

　ビューとモデル(DB)を橋渡しをするコントローラを作成していきます。  

#### 2-1.  タスク一覧を渡すコントローラ作成
今はタスク一覧はDBから持ってくるのではなく、コントローラ内でをタスク配列として用意し、  
そのタスク一覧をテンプレート側に渡す処理を書いていきます。
```
$ mkdir controllers
$ vim controllers/task.go
```

```go
package task

import "strconv"

// idとテキストを保持する構造体
type Task struct {
	ID   int
	Text string
}

func NewTask() Task {
	return Task{}
}

// タスク構造体一覧を返す
func (c Task) GetAll() interface{} {

	// テストデータとして５つタスクを作成
	tasks := make([]*Task, 5)
	for i := 1; i <= 5; i++ {
		tasks[i-1] = &Task{ID: i, Text: "Task Text " + strconv.Itoa(i)}
	}

	return tasks
}
```

#### 2-2. コントローラの呼び出し

```
$ vim main.go
```

```go
//・・・

router.GET("/", func(c *gin.Context) {
    controller := task.NewTask()
    tasks := controller.GetAll()

    c.HTML(http.StatusOK, "index.tmpl", gin.H{
        "title": "TODO List",
        "tasks": tasks,    //　追記 テンプレートにタスクを渡す
    })
})

//・・・
```

#### 2-3. テンプレートを修正

　タスク一覧をリストとして表示するforを記述していきます。

```
$ vim views/index.tmpl
```

```html
//・・・

<body>
<h3>{{ .title }}</h3>
    <ul>
    {{ range $index, $task := .tasks }}
        <li>{{ $task.ID }}: {{ $task.Text }} </li>
    {{ end }}
    </ul>
</body>

//・・・
```

[f:id:motikan2010:20170211215616p:plain]

### 3. モデルを作成して、DBにタスクを保存する

　タスク内容をDB内に保存できるようにします。  
今回DBはSQLiteを使っていきます。

#### 3-1. Modelを作成

　タスクを登録するためにCreate関数を用意しています。  
文字列の引数を受け取り、その文字列をDBに挿入するような動作を行います。

```
$ mkdir models
$ vim models/task.go
```

```go
package task

import (
	"github.com/jinzhu/gorm"
	_ "github.com/mattn/go-sqlite3"
)

var db *gorm.DB

func init() {
	var err error

	db, err = gorm.Open("sqlite3", "task.db")

	db.DropTableIfExists(&Task{})
	db.CreateTable(&Task{})

	if err != nil {
		panic(err)
	}
}

type Task struct {
	ID   int    `gorm:"primary_key"`
	Text string `gorm:"size:140"`
}

type Tasks []Task

type TaskRepository struct {
}

func NewTaskRepository() TaskRepository {
	return TaskRepository{}
}

// データベースに一行登録する
func (m TaskRepository) Create(text string) {
	var task = Task{Text: text}
	db.NewRecord(task)
	db.Create(&task)
	db.Save(&task)
}
```

#### 3-2. コントローラ − Create関数を追加

　こちらでもCreate関数を定義しています。  
モデル内に定義されているCreate関数に対して、ユーザが送信したタスク文字列を渡しています。

```
$ vim controllers/task.go
```

```go
import (
	"strconv"

	task "../models"
)

//・・・

func (c Task) Create(text string) {
	repo := task.NewTaskRepository()
	repo.Create(text)
}
```

#### 3-3. POSTデータの受け取り

　「text := c.PostForm("text")」でユーザが送信したパラメータを取得することができます。
```
$ vim main.go
```

```go
func main() {

	//・・・

	router.POST("/", func(c *gin.Context) {
		text := c.PostForm("text")
		ctrl := task.NewTask()
		ctrl.Create(text)

		c.Redirect(http.StatusMovedPermanently, "/")
	})
```

#### 3-4. 登録フォームの作成

　アプリケーションにタスク文字列を送信するためにフォームを追加します。  

```
$ vim views/index.tmpl
```

```html
//・・・

<ul>
<form action="/" method="post">
  <input type="text" name="text"></input>
  <input type="submit" value="送信">
</form>
{{ range $index, $task := .tasks }}
    <li>{{ $task.ID }}: {{ $task.Text }} </li>
{{ end }}
</ul>

//・・・
```

#### 3-5. タスクを登録
```
$ go run main.go
```
[f:id:motikan2010:20170211215637p:plain]

#### 3-6. 登録されたタスクの確認

```
$ sqlite3 task.db

sqlite> .tables
tasks

sqlite> select * from tasks;
1|Test task
```

### 4. データベースに登録したタスクを出力

#### 4-1. モデル − GetAll関数を追加

　GetAll関数はDBに登録されているタスクを全て返します。
```
$ vim models/task.go
```

```go
func (m TaskRepository) GetAll() Tasks {
	var tasks = Tasks{}
	db.Find(&tasks)

	return tasks
}
```

#### 4-2. コントローラ − GetAll関数を修正

```
$ vim controllers/task.go
```

```go
func (c Task) GetAll() interface{} {
	repo := task.NewTaskRepository()
	tasks := repo.GetAll()

	return tasks
}
```

[f:id:motikan2010:20170211215706p:plain]

### 5. idを指定してタスクを取得

#### 5-1. モデル - GetByID関数を追加

　GetByID関数はタスクIDを引数として受け取り、該当するタスクを返します。
```
$ vim models/task.go
```
```go
func (m TaskRepository) GetByID(id int) Tasks {
	var tasks = Tasks{}
	db.Find(&tasks, id)
	return tasks
}
```

#### 5-2. コントローラ - Get関数を追加
```
$ vim controllers/task.go
```

```go
func (c Task) Get(n int) interface{} {
	repo := task.NewTaskRepository()
	tasks := repo.GetByID(n)

	return tasks
}
```

#### 5-3. Getを呼び出す

```go
func main() {

    //・・・

    router.GET("/:id", func(c *gin.Context) {
            var id, _ = strconv.Atoi(c.Param("id"))
            ctrl := task.NewTask()
            tasks := ctrl.Get(id)

            c.HTML(http.StatusOK, "index.tmpl", gin.H{
                "tasks": tasks,
            })
        })
```

#### 5-4. 動作確認
「`http://127.0.0.1:8080/2`」にアクセスすると、idが2のタスクが表示されます。  
[f:id:motikan2010:20170211215739p:plain]