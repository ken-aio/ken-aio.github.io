---
title: "GolangのORM SQLBoilerを使ってみる - セットアップ編"
date: 2019-02-13T23:00:00+09:00
draft: true
keywords: ["golang", "sqlboiler"]
description: ""
tags: ["golang", "sqlboiler"]
categories: ["golang", "ORM"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
GolangのORMである [SQLBoiler](https://github.com/volatiletech/sqlboiler) を紹介します。  
今回はSQLBoilerをセットアップするところまでやってみます。  

# SQLBoilerとは
DBのスキーマ情報を読み取って、そこからstructやそれらのレシーバを自動生成してくれるORMです。  
自動生成の対象にはFK成約やUnique成約を考慮して、one to one や one to manyの関数を自動生成してくる素敵なORMです。  
リフレクションなどを使っていないため、他のORMと比較しても動作が早いようです。ベンチマーク結果は [公式サイト](https://github.com/volatiletech/sqlboiler#benchmarks) を参照ください。  
また、エラーや使い方など、何か困ったことがあっても自動生成されたコードを読めば大体わかるのもいいところです。  

同じようにスキーマ情報からstructなどを自動生成するORMに [XORM](https://github.com/go-xorm) があります。  
XORMも少し使ったことがあるのですが、生成される対象の関数などはSQLBoilerの方が圧倒的に多く、便利だと感じました。(私見)  

# 使ってみる
## インストール
まずはsqlboilerのコマンドをインストールします。  
```
$ go get -u -t github.com/volatiletech/sqlboiler
$ sqlboiler --version
SQLBoiler v3.2.0
```

また使うRDBMS用のライブラリも必要です。  
今回のサンプルではPostgreSQLを使います。  
```
$ go get github.com/volatiletech/sqlboiler/drivers/sqlboiler-psql
```

## プロジェクト設定する
sqlboilerのコマンドをインストールしたら、次に接続先の設定を行います。  
まずはプロジェクトのroot dirに以下のようなtomlファイルを作ります。  
```
pkgname="db"
output="app/db"
add-global-variants=true
add-panic-variants=true
[psql]
  dbname="sampledb"
  host="localhost"
  port=5432
  user="postgres"
  pass=""
  sslmode="disable"
```
それぞれ名前の通りの設定ですが、2つだけ見慣れないパラメータがあると思います。  
SQLBoilerのDBとのコネクションはgolang標準のdatabase/sqlのコネクションを使います。  
このコネクションはアプリケーション起動時にdatabase/sqlのコネクションプールを使うことも出来るし、実際にSQLを発行するタイミングで別のコネクションを渡すことも出来ます。  
また、SQL実行時に何かしらのエラーが発生した場合、 `err` を返す方法とその場で `panic` を起こすことが出来ます。

* add-global-variants
  * これはグローバルに設定したコネクションを使ってSQLを発行するオプションを自動生成するか、のフラグです
* add-panic-variants
  * これはSQLでエラーが発生した場合にpanicを起こすかどうかのオプションを自動生成するか、のフラグです。
  * プロダクションコードではSQLエラーで直接panicを起こすことはあまりないかと思いますが、テストコードだとエラーハンドリングせずにpanicさせてしまうことはよくやります。

これらは有用なオプションなので、自動生成の対象に入れています。

## 対象のDB
次に自動生成しますが、対象のDBを作っておきます。  
今回は以下のような簡単なテーブル構成を想定します。  

```
---------            -----------            ----------
| Users | 1 ... 0..* | Members | 1..* ... 1 | Groups |
---------            -----------            ----------
```

DDLは以下を使います。  

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
```

## 自動生成する
まずはpostgresにDBとTableを作ります。  
```
$ createdb sampledb
$ psql -U postgres -f ddl.sql sampledb
CREATE DATABASE
CREATE TABLE
CREATE TABLE
CREATE TABLE
ALTER TABLE
ALTER TABLE
```

そして、自動生成  
```
$ sqlboiler psql
```
成功していれば何も表示されず、 `app/db` のディレクトリが出来ていてその中に生成されたファイルが存在すると思います。  

# まとめ
とりあえず今回はSQLBoilerのセットアップをして自動生成するところまでやってみました。  
次回は生成されたものを使ってSQLを発行してみようと思います。  

今回作ったサンプルは以下のリポジトリに入れておきます。  
https://github.com/ken-aio/go-sqlboiler-sample
