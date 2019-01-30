---
title: "GolangとEchoでお手軽にAPIサーバを立てる"
date: 2019-01-31T00:20:00+09:00
draft: false
keywords: ["golang","echo"]
description: "GolangとEchoでお手軽にAPIサーバを作ってみます"
tags: ["golang","echo"]
categories: ["golang"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
golangとechoを使って簡単にAPIサーバを作る方法をまとめます。  

# 動機
golangでAPIサーバを作りたい場合、どんな選択をするでしょうか  
golangは標準のnet/httpがしっかりしているので、そもそもFWが不要、という議論もよく見かけます。  
しかし、net/httpを触ってみてやっぱりFW使った方がいいなと感じました。  

* リクエストのパースが面倒
* ミドルウェアの組み込みが面倒
* http methodを意識したroutingの設定が面倒

など、net/httpも便利ですがやっぱりある程度は自分で書かないといけないと感じました。  
その結果、結局オレオレFWが出来てしまう予感がして、FWを探しました。  
その中でechoに出会いました。  

# Echoとは
golangのWeb FWです。  
一応HTMLも出力できるのですが、APIサーバとしてのユースケースが多いんじゃないかなと感じてます。  
公式によると、有名なFWのginよりもパフォーマンスがよいとのこと。  

* [echo official site](https://echo.labstack.com/)
* [echo github](https://github.com/labstack/echo)

（オフィシャルサイトがきれいに作られていますね^^）

# API作ってみる
まずはhello worldを返すAPIを実装してみます。  

今回のサンプルは↓のリポジトリにおいてあります。  

https://github.com/ken-aio/go-echo-sample

## インストール
Go Modulesを使ってechoをインストールします。  
現時点ではgo modulesを使うには環境変数を設定する必要があります。  
※ なんと、echoにgo modulesが入ったのはこの記事を書く2日前でした（!!）  
※ さらに、echo v4が出たのがこの記事を書いている間に行われました...w  

```
$ echo 'export GO111MODULE=on' >> .env
$ . .env
```

まずはgo modulesを初期化します。  

```
$ go mod init
```

そしてechoをインストールします。  
go modulesがonの状態でgo getをすると依存関係含めて諸々をダウンロードしてくれます。  
go modules、便利ですね。  

```
$ go get github.com/labstack/echo/v4
```

これでインストールは完了です。  

## hello world
早速hello worldを作ってみます。  
やり方はとても簡単、echo作って、routingとfunction書いて、startするだけ、です。  

```
package main

import (
        "net/http"

        "github.com/labstack/echo/v4"
)

func main() {
        e := echo.New()

        e.GET("/", func(c echo.Context) error {
                return c.JSON(http.StatusOK, map[string]string{"hello": "world"})
        })

        e.Logger.Fatal(e.Start(":1313"))
}
```

実行してみる。  
```
$ go run main.go

   ____    __
  / __/___/ /  ___
 / _// __/ _ \/ _ \
/___/\__/_//_/\___/ v4.0.0
High performance, minimalist Go web framework
https://echo.labstack.com
____________________________________O/_______
                                    O\
⇨ http server started on [::]:1313
```
```
$ curl localhost:1313
{"hello":"world"}
```

とても簡単ですね。  

## ロジックを分割
これだけでは流石につまらないので、ロジックを分離してみます。  
...といってもfuncに切り出しただけですが;;  

```
func main() {
	e := echo.New()

	e.GET("/", hello)

	e.Logger.Fatal(e.Start(":1313"))
}

func hello(c echo.Context) error {
	return c.JSON(http.StatusOK, map[string]string{"hello": "world"})
}
```

## routingを分割
routingも分離してみましょう。  
...簡単ですね  

```
func main() {
	e := echo.New()

	initRouting(e)

	e.Logger.Fatal(e.Start(":1313"))
}

func initRouting(e *echo.Echo) {
	e.GET("/", hello)
}

func hello(c echo.Context) error {
	return c.JSON(http.StatusOK, map[string]string{"hello": "world"})
}
```

## path parameterとquery parameter
最後にpath parameterとquery parameterを受け取ってみます。  
responseはユーザのリストを返します。  

まず、userのstructを定義します。  
```
type User struct {
	ID      int    `json:"id"`
	GroupID int    `json:"group_id"`
	Name    string `json:"name"`
	Gender  string `json:"gender"`
}
```

Routingに今回定義したAPIを追加します。  
```
func initRouting(e *echo.Echo) {
	e.GET("/", hello)
	e.GET("/api/v1/groups/:group_id/users", getUsers)
}
```

処理を書きます。  
```
func getUsers(c echo.Context) error {
	groupIDStr := c.Param("group_id")
	groupID, err := strconv.Atoi(groupIDStr)
	if err != nil {
		return errors.Wrapf(err, "errors when group id convert to int: %s", groupIDStr)
	}
	gender := c.QueryParam("gender")
	users := []*User{}
	if gender == "" || gender == "man" {
		users = append(users, &User{ID: 1, GroupID: groupID, Name: "Taro", Gender: "man"})
		users = append(users, &User{ID: 2, GroupID: groupID, Name: "Jiro", Gender: "man"})
	}
	if gender == "" || gender == "woman" {
		users = append(users, &User{ID: 3, GroupID: groupID, Name: "Hanako", Gender: "woman"})
		users = append(users, &User{ID: 4, GroupID: groupID, Name: "Yoshiko", Gender: "woman"})
	}
	return c.JSON(http.StatusOK, users)
}
```

ポイントとしては、 `c.Param` でパスパラメータを受け取れて、 `c.QueryParam` でクエリパラメータを受け取れます。  
ただし、いずれもstringでしか受け取れないので、型の変換はアプリのロジックの中でやる必要があります。  

では起動して実行してみます。  
query parameterに `pretty` をつけるとjsonをpretty printしてくれます。  
```
$ curl localhost:1313/api/v1/groups/1/users
[{"id":1,"group_id":1,"name":"Taro","gender":"man"},{"id":2,"group_id":1,"name":"Jiro","gender":"man"},{"id":3,"group_id":1,"name":"Hanako","gender":"woman"},{"id":4,"group_id":1,"name":"Yoshiko","gender":"woman"}]

$ curl localhost:1313/api/v1/groups/1/users?gender=man
[{"id":1,"group_id":1,"name":"Taro","gender":"man"},{"id":2,"group_id":1,"name":"Jiro","gender":"man"}]

$ curl 'localhost:1313/api/v1/groups/1/users?gender=woman&pretty'
[
  {
    "id": 3,
    "group_id": 1,
    "name": "Hanako",
    "gender": "woman"
  },
  {
    "id": 4,
    "group_id": 1,
    "name": "Yoshiko",
    "gender": "woman"
  }
]
```

また、エラーを返すとデフォルトでinternal server errorを返してくれます。  
```
$ curl 'localhost:1314/api/v1/groups/invalid/users?gender=woman&pretty'
{
  "message": "Internal Server Error"
}
```

ちゃんとAPIサーバが出来ましたね！  

# まとめ
今回はとても簡単な使い方についてまとめてみました。  
今度、もう少し踏み込んだ使い方もまとめてみたいと思います。  
