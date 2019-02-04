---
title: "[Golang]Echoで簡単にSwaggerを利用する"
date: 2019-02-05T00:00:00+09:00
draft: false
keywords: ["golang", "echo", "swagger"]
description: "[Golang]Echoで簡単にSwaggerを利用する"
tags: ["golang", "echo", "swagger"]
categories: ["golang", "echo"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
golangとechoを使ってJSON APIのswaggerを生成する方法について書きます。  
[swag](https://github.com/swaggo/swag) というツールが提供されているので、基本的にはこのツールを使っていきます。  

# 今回作ったサンプル置き場
使い方だけ見たい方はgithubのコードを見るのがいいと思います。  

https://github.com/ken-aio/go-echo-sample/tree/v2

# 流れ
swagを使ったswaggerドキュメントの生成は↓の流れで行います。  

1. コメントを書く
2. 自動生成する
3. swagger uiのhandlerを設定する
4. swaggerを開く

実際にサンプルプロジェクトを使ってswaggerを作ってみます。  

## 1. コメントを書く
### API全体の設定
まずは全体の設定を `main` の中に書いていきます。  
中身は利用に関する規約のURL、ライセンス情報、認証用のhttp header情報などの設定が出来ます。  
今回は内部で使うだけの場合を想定して、最低限の設定だけをしてみます。  
※ [詳細の設定内容はこちら](https://github.com/swaggo/swag#general-api-info)
```
// @title Swagger Example API
// @version 1.0
// @description This is a sample swagger server.
// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html
// @host localhost:1313
// @BasePath /api/v1
func main() {
...
```

### 各エンドポイントの設定
グループに所属しているユーザの一覧を取得するAPIのswagger docを設定します。  
group_idをpathパラメータに受け取って、性別（gender）をqueryパラメータで受け取ります。  
※ [詳細の設定パラメータはこちら](https://github.com/swaggo/swag#api-operation)

```
// getUsers is getting users.
// @Summary get users
// @Description get users in a group
// @Accept  json
// @Produce  json
// @Param group_id path int true "Group ID"
// @Param gender query string false "Gender" Enum(man, woman)
// @Success 200 {array} main.User
// @Failure 500 {object} main.HTTPError
// @Router /groups/{group_id}/users [get]
func getUsers(c echo.Context) error {
```

## 2. 自動生成する
設定した内容に従ってswagger uiのもととなるyamlを自動生成します。  
まずはswagのコマンドをインストールします。  

```
$ go get -u github.com/swaggo/swag/cmd/swag
```

自動生成します。  
生成すると、docsというdirが出来て、この中にswaggerのjsonとyamlが生成されます。  
```
$ swag i
```

## 3. swagger uiのhandlerを設定する
次にechoからswagger uiを呼び出せるようにhandlerの設定を行います。  
[echo-swagger](https://github.com/swaggo/echo-swagger) というライブラリがあるので、こちらを使ってechoからswagger uiを呼び出せるようにします。  
routerを初期化するタイミングでswagger uiの設定をします。  
```
	// swagger
	e.GET("/swagger/*", echoSwagger.WrapHandler)
```

また、docsをimportする必要があるので、以下のようにmainにdocsをimportしておきます。  
```
import (
	_ "github.com/ken-aio/go-echo-sample/docs"
)
```

## 4. swaggerを開く
ここまで出来たらあとはサーバを立ち上げてswagger uiにアクセスするだけです。  
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
⇨ http server started on [::]:1314
```

この状態でブラウザで以下のURLにアクセスすると、swagger uiが表示されると思います。  

http://localhost:1314/swagger/index.html

Try it outから実際にAPIを叩いてみると、APIがたたけることを確認できます。  

# echo v4対応
現時点ではecho v4が出たばかりということもあって、echo-swaggerがv4では使えませんでした。  
echo-swaggerをforkして、echoのバージョンをv4にあげたものが以下のリポジトリにあるので、これを使えばecho v4の環境でもecho-swaggerを使うことが出来ます。  

https://github.com/ken-aio/echo-swagger

# まとめ
go docを書くだけでswaggerが出来てしまう超便利なswagをechoで使う方法でした。  
APIドキュメントを書くのが面倒な人にはおすすめです。  
