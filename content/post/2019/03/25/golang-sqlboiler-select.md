---
title: "GolangのORM SQLBoilerを使ってみる - 実装編(Select)"
date: 2019-03-25T23:07:18+09:00
draft: true
keywords: ["golang", "ORM", "sqlboiler"]
description: "GolangのORM SQLBoilerを使ってみる - 実装編(Select)"
tags: ["golang", "ORM", "sqlboiler"]
categories: ["golang", "ORM", "sqlboiler"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
* [GolangのORM SQLBoilerを使ってみる - セットアップ編](https://ken-aio.github.io/post/2019/02/13/golang-sqlboiler/)
* [GolangのORM SQLBoilerを使ってみる - 実装編(Create/Update/Delete)](https://ken-aio.github.io/post/2019/03/25/golang-sqlboiler-cud/)

でセットアップ完了と更新系ができしたので、続いてselect系について書いていこうと思います。  
SQLBoilerのselect系はかなりのことを行うことができます。  

# Query Mod
SQLBoilerを使うにあたってはQuery Modを理解することが大事です。  
Query ModはSQLBoiler独自の概念で、SQLの条件をtype safeに定義できる仕組みです。  
Query Modは `qm` パッケージに属しています。  

例えば、where区を書きたい場合は `qm.Where("user.id = ?", userID)` などのように定義できます。  
そして、Query Modで条件を定義した上で最後にFinisherを使います。  
FinisherがSQLの基本的な形を決定します。  
Finisherにはいくつかの種類がありますが、よく使うのは `All` (select *) `One` (select * ... limit 1) `Count` (select count(*)) あたりかと思います。  
全容については以下をご参照ください。  
https://github.com/volatiletech/sqlboiler#finishers

実際にSQLを発行してみますが、いずれも↓のDDLで作ったテーブルを使っています。  
```
CREATE TABLE groups
(
	id serial NOT NULL,
	name text NOT NULL,
	description text NOT NULL,
	created_at timestamp NOT NULL,
	updated_at timestamp NOT NULL,
	PRIMARY KEY (id)
) WITHOUT OIDS;


CREATE TABLE group_members
(
	id bigserial NOT NULL,
	user_id int NOT NULL,
	group_id int NOT NULL,
	role text NOT NULL,
	created_at timestamp NOT NULL,
	updated_at timestamp NOT NULL,
	PRIMARY KEY (id),
	CONSTRAINT UQ_group_members_user_group UNIQUE (user_id, group_id)
) WITHOUT OIDS;

CREATE TABLE users
(
	id serial NOT NULL,
	email text UNIQUE,
	password_digest text,
	created_at timestamp NOT NULL,
	updated_at timestamp NOT NULL,
	PRIMARY KEY (id)
) WITHOUT OIDS;

ALTER TABLE group_members
	ADD CONSTRAINT FK_group_members_groups FOREIGN KEY (group_id)
	REFERENCES groups (id)
	ON UPDATE RESTRICT
	ON DELETE RESTRICT
;

ALTER TABLE group_members
	ADD CONSTRAINT FK_group_members_users FOREIGN KEY (user_id)
	REFERENCES users (id)
	ON UPDATE RESTRICT
	ON DELETE RESTRICT
;
```

# 単一テーブル
## テーブル全件取得
まずは簡単な全件取得です。ユーザを全件取得してい見ます。  
```
users := db.Users().AllGP(context.Background())
fmt.Printf("users = %+v\n", users)
```
```
$ go run main.go
SELECT * FROM "users";
[]
users = [0xc000164100]
```
全件なので、Allを使っています。sliceで返ってきていることがわかりますね。  
末尾のGとPはそれぞれ以下の意味となります。  

* G ・・・ GlobalなDBコネクションを使う。トランザクション制御をしたい場合、Beginした時の `sql.Tx` を使います。  
* P ・・・ エラーが発生したら `err` を返すのではなく、panicを起こす。productionで使う場合は基本的にPは使わないほうがよいです。  

## 1件取得
続いて1件取得してみます。  
```
user := db.Users().OneGP(context.Background())
fmt.Printf("user = %+v\n", user)
```
```
SELECT * FROM "users" LIMIT 1;
[]
user = &{ID:1 Email:{String:test@example.com Valid:true} PasswordDigest:{String:digested-password Valid:true} CreatedAt:2019-03-25 13:52:29.148263 +0000 +0000 UpdatedAt:2019-03-25 13:52:29.
148263 +0000 +0000 R:<nil> L:{}}
```
Allとの違いとしては、 `One` を指定するとSQLに `limit 1` が追加されています。  
また、 `All` のときはsliceで返ってきていたのがOneだとstructで返ってきました。  

# Join
続いてjoinしてみます。memberに所属しているuserだけを取得します。  
```
users := db.Users(qm.InnerJoin("group_members on group_members.user_id = users.id")).AllGP(context.Background())
fmt.Printf("users = %+v\n", users)
```
```
$ go run main.go
SELECT "users".* FROM "users" INNER JOIN group_members on group_members.user_id = users.id;
[]
users = [0xc000184100]
```
ちゃんとJoinできました。  

## joinしたデータを取得
joinしたデータを取得するのは少し面倒です。取得したいentityを含んだstructを作る必要があります。  
```
type userMember struct {
	db.User        `boil:",bind"`
	db.GroupMember `boil:",bind"`
}
var mem userMember
db.Users(qm.Select("users.*, group_members.*"), qm.InnerJoin("group_members on group_members.user_id = users.id")).BindG(context.Background(), &mem)
fmt.Printf("mem = %+v\n", mem)
```
```
$ go run main.go
SELECT users.*, group_members.* FROM "users" INNER JOIN group_members on group_members.user_
id = users.id;
[]
mem = {User:{ID:1 Email:{String:test@example.com Valid:true} PasswordDigest:{String:digested-password Valid:true} CreatedAt:2019-03-28 23:41:47.305423 +0000 +0000 UpdatedAt:2019-03-2823:41:47.305423 +0000 +0000 R:<nil> L:{}} GroupMember:{ID:2 UserID:1 GroupID:1 Role:admin CreatedAt:2019-03-25 13:52:29.148263 +0000 +0000 UpdatedAt:0001-01-01 00:00:00 +0000 UTC R:<nil> L:{}}}
```
joinして一括でデータをガツッと取得したい場合なんかは良さそうです。

# eager loading

# 特定の条件をつける

# まとめ

