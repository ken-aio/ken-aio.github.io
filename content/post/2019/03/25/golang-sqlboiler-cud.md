---
title: "GolangのORM SQLBoilerを使ってみる - 実装編(Create/Update/Delete)"
date: 2019-03-25T23:00:00+09:00
draft: false
keywords: ["golang", "ORM", "sqlboiler"]
description: "GolangのORM SQLBoilerを使ってみる - 実装編(Create/Update/Delete)"
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

ここで出てくる `null.String` はnot null制約がついていないカラムについてnullで覆うことでnull safeな状態になっています。  
https://github.com/volatiletech/null  
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
次にupdateしてみます。  
updateも簡単で、entity structのupdateを呼ぶのみです。  
```
func update() {
	user := db.User{ID: 1}
	user.Email = null.StringFrom("update@example.com")
	user.UpdateGP(context.Background(), boil.Infer())
}
```

実行してみるとupdateが発行されているログが確認できます。  
```
$ go run main.go
UPDATE "users" SET "email"=$1,"password_digest"=$2,"updated_at"=$3 WHERE "id"=$4
[{update@example.com true} { false} 2019-02-21 14:13:19.8155 +0000 UTC 1]
```

# delete
続いてdeleteしてみます。  
deleteもupdateのようにentity structのdeleteを呼びます。  
```
func delete() {
	user := db.User{ID: 1}
	user.DeleteGP(context.Background())
}
```

実行するとdeleteが発行されています。  
```
$ go run main.go
DELETE FROM "users" WHERE "id"=$1
1
```

# トランザクション処理
トランザクション処理も簡単にできます。  
以下がサンプルコードです。(簡単のためにエラーは無視しています)  
```
func insertTx() {
	ctx := context.Background()
	tx, err := boil.BeginTx(ctx, nil)
	if err != nil {
		panic(err)
	}
	user := db.User{Email: null.StringFrom("test@example.com"), PasswordDigest: null.StringFrom("digested-password")}
	fmt.Printf("before user = %+v\n", user)
	err = user.Insert(ctx, tx, boil.Infer())
	if err != nil {
		tx.Rollback()
		panic(err)
	}
	tx.Commit()
	fmt.Printf("after user = %+v\n", user)
}
```

実行すると以下のログが確認できます。  
```
$ go run main.go
before user = {ID:0 Email:{String:test@example.com Valid:true} PasswordDigest:{String:digested-password Valid:true} CreatedAt:0001-01-01 00:00:00 +0000 UTC UpdatedAt:0001-01-01 00:00:00 +0000 UTC R:<nil> L:{}}
INSERT INTO "users" ("email","password_digest","created_at","updated_at") VALUES ($1,$2,$3,$4) RETURNING "id"
[{test@example.com true} {digested-password true} 2019-03-25 13:52:29.148263 +0000 UTC 2019-03-25 13:52:29.148263 +0000 UTC]
after user = {ID:1 Email:{String:test@example.com Valid:true} PasswordDigest:{String:digested-password Valid:true} CreatedAt:2019-03-25 13:52:29.148263 +0000 UTC UpdatedAt:2019-03-25 13:52:29.148263 +0000 UTC R:<nil> L:{}}
```

# まとめ
これでSQLBoilerを使って更新系の処理を行うことができました。  
自動生成機能があるので、色々と楽にできる部分が多いですね。  
次回は多彩なselectをやってみようと思います。  
今回作ったサンプルは以下のリポジトリに入れてあります。  
https://github.com/ken-aio/go-sqlboiler-sample
