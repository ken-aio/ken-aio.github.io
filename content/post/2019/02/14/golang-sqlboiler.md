---
title: "GolangのORM SQLBoilerを使ってみる - 実装編"
date: 2019-02-14T23:09:03+09:00
draft: true
keywords: ["golang", "ORM", "sqlboiler"]
description: "GolangのORM SQLBoilerを使ってみる - 実装編"
tags: ["golang", "ORM", "sqlboiler"]
categories: ["golang", "ORM", "sqlboiler"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
[GolangのORM SQLBoilerを使ってみる - セットアップ編](https://ken-aio.github.io/post/2019/02/13/golang-sqlboiler/) でセットアップ完了したので、実際にSQLBoilerを使って実装してみたいと思います。  

# 初期化
まずは初期化を行います。  
初期化ではコネクションプールを作ったり、SQLBoiler自体の設定をしたりします。  
今回は発行されたSQLを見てみたいので、debugモードにします。  
なお、簡単化のため、エラーが出たら全てpanicを起こしています。(良い子は真似しないように)  

以下のコードで初期化出来ます。  
```
func initDB() {
	dns := "user=postgres dbname=sampledb host=localhost sslmode=disable connect_timeout=10"
	con, err := sql.Open("postgres", dns)
	if err != nil {
		panic(err)
	}
	// connection pool settings
	con.SetMaxIdleConns(10)
	con.SetMaxOpenConns(10)
	con.SetConnMaxLifetime(300 * time.Second)

	// global connection setting
	boil.SetDB(con)
	boil.DebugMode = true
}
```

# insert
まずはinsertをしてみます。insertは非常に簡単に出来ます。  
insertしたいテーブルのStructインスタンスを作って、Insertの関数を呼ぶだけです。  

```
func insert() {
	user := db.User{Email: null.StringFrom("test@example.com"), PasswordDigest: null.StringFrom("digested-password")}
	fmt.Printf("before user = %+v\n", user)
	user.InsertGP(context.Background(), boil.Infer())
	fmt.Printf("after user = %+v\n", user)
}
```

これを実行すると、以下のようなログが出力されます。  
```
$ go run main.go
before user = {ID:0 Email:{String:test@example.com Valid:true} PasswordDigest:{String:digested-password Valid:true} CreatedAt:0001-01-01 00:00:00 +0000 UTC UpdatedAt:0001-01-01 00:00:00 +0000 UTC R:<nil> L:{}}
INSERT INTO "users" ("email","password_digest","created_at","updated_at") VALUES ($1,$2,$3,$4) RETURNING "id"
[{test@example.com true} {digested-password true} 2019-02-14 14:43:41.253731 +0000 UTC 2019-02-14 14:43:41.253731 +0000 UTC]
after user = {ID:3 Email:{String:test@example.com Valid:true} PasswordDigest:{String:digested-password Valid:true} CreatedAt:2019-02-14 14:43:41.253731 +0000 UTC UpdatedAt:2019-02-14 14:43:41.253731 +0000 UTC R:<nil> L:{}}
```
デフォルトで `created_at` と `updated_at` に現在時刻を入れてくれます。  
もちろん、updateの時は `updated_at` だけが更新されます。  

# update

# delete

# select
## query mod
## 単一テーブル
## join
## load
## 特定の条件をつける

# まとめ
