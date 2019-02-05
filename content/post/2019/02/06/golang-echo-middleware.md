---
title: "[Golang]EchoのMiddlewareを実装する"
date: 2019-02-06T00:00:00+09:00
draft: false
keywords: ["golang", "echo", "middleware"]
description: "[Golang]EchoのMiddlewareを実装する"
tags: ["golang", "echo"]
categories: ["golang", "echo"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
golangのWeb  FWのechoを使って独自のmiddle wareを作る方法について書きます。  

# echoが用意してるmiddleware
echoは標準で様々なmiddlewareを用意してくれています。  
詳しくは [echoの公式サイト](https://echo.labstack.com/middleware) を見てみてください  
代表的なものとしては

* [Logger](https://echo.labstack.com/middleware/logger)
* [Recover](https://echo.labstack.com/middleware/recover)

あたりで、echoを使うならとりあえず入れておいて損はないmiddlewareかと思います。  
使い方は、アプリケーションを起動する際にechoのUseを呼ぶだけ、です。  

```
e := echo.New()
e.Use(middleware.Recover())
e.Use(middleware.Logger())
```

以下、LoggerとRecoverについて簡単に紹介します。  

## Logger
いわゆるアクセスログのようなリクエスト単位のログを出力してくれます。  
フォーマットもいじれて便利です。  
デフォルトのフォーマットは以下のように設定されています。  
[ソースコード](https://github.com/labstack/echo/blob/master/middleware/logger.go)

```
Format: `{"time":"${time_rfc3339_nano}","id":"${id}","remote_ip":"${remote_ip}",` +
	`"host":"${host}","method":"${method}","uri":"${uri}","user_agent":"${user_agent}",` +
	`"status":${status},"error":"${error}","latency":${latency},"latency_human":"${latency_human}"`
```

これ以外にもログ項目を追加する方法が提供されています。詳細は [ソースコードのコメント](https://github.com/labstack/echo/blob/master/middleware/logger.go#L23) を参照するのが良いかと思います。  
この中でも特に便利だなと思ったのが、http headerやquery、formのデータをログに出力できることです。  
ログ出力の実装は↓のようになっているので、それに合わせてフォーマット定義すればその通りにログが出力されます。  

```
switch {
case strings.HasPrefix(tag, "header:"):
	return buf.Write([]byte(c.Request().Header.Get(tag[7:])))
case strings.HasPrefix(tag, "query:"):
	return buf.Write([]byte(c.QueryParam(tag[6:])))
case strings.HasPrefix(tag, "form:"):
	return buf.Write([]byte(c.FormValue(tag[5:])))
case strings.HasPrefix(tag, "cookie:"):
	cookie, err := c.Cookie(tag[7:])
	if err == nil {
		return buf.Write([]byte(cookie.Value))
	}
}
```

## Recover
アプリケーションのどこかで予期せずにpanicを起こしてしまっても、サーバは落とさずにエラーレスポンスを返せるようにリカバリーするmiddlewareです。  
nil pointerなどのRuntimeエラーはどうしても発生してしまうので、APIサーバとして使うのであれば必須なmiddlewareかなと思います。  
[詳細はこちら](https://echo.labstack.com/middleware/recover)

# middlewareを実装する
[公式ドキュメント](https://echo.labstack.com/cookbook/middleware) を参照すると丁寧にmiddlewareの作り方が書かれています。  
こちらを参考にmiddlewareを作ってみます。  
middlewareは `echo.HandlerFunc` を返す関数を作ればよいだけ、です。  

```
func myMiddleware(next echo.HandlerFunc) echo.HandlerFunc {
	return func(c echo.Context) error {
		log.Println("before action")
		if err := next(c); err != nil {
			c.Error(err)
		}
		log.Println("after action")
		return nil
	}
}
```

`next(c)` で実際のアプリの処理が実行されるアプリの処理の前後に処理をはさみたいときには `next(c)` の前後に処理を書けばよいです。  
これを定義した上で、echoを初期化する際に Use してあげれば自分で作ったmiddlewareが有効になります。  

```
e.Use(myMiddleware)
```

処理にもログを仕込みます。  
```
	e.GET("/", hello)
...
func hello(c echo.Context) error {
	log.Println("hello action")
	return c.JSON(http.StatusOK, map[string]string{"hello": "world"})
}
```

echoを起動して、curlコマンドを実行してみると、以下のログが出力されます。  
```
$ curl localhost:1314
```
```
2019/02/05 23:31:38 before action
2019/02/05 23:31:38 hello action
2019/02/05 23:31:38 after action
{"time":"2019-02-05T23:31:38.618186+09:00","id":"","remote_ip":"::1","host":"localhost:1314","method":"GET","uri":"/","user_agent":"curl/7.54.0","status":200,"error":"","latency":416885,"latency_human":"416.885µs","bytes_in":0,"bytes_out":18}
```

これでアプリの処理の前後に処理を挟み込むことが出来ました。  
Loggerミドルウェアを使っているのでアクセスログも出力されています。  

# Loggerミドルウェアをカスタマイズする
最後にLoggerミドルウェアを少しカスタマイズしてみます。  
`e.Use(middleware.Logger())` としていた部分を以下のように書き換えます。  

```
e.Use(middleware.LoggerWithConfig(middleware.LoggerConfig{
	Format: `${time_rfc3339_nano} ${host} ${method} ${uri} ${status} ${header:my-header}` + "\n",
}))
```

この状態でアプリを起動してcurlを打ってみます。そしたらログには以下のように出力されます。  
```
2019/02/05 23:43:17 before action
2019/02/05 23:43:17 hello action
2019/02/05 23:43:17 after action
2019-02-05T23:43:17.211349+09:00 localhost:1314 GET / 200
```

また、独自のheaderを指定すると、それがログに出力されます。  
```
$ curl localhost:1314 -H "my-header:echo sample header"
```
```
2019-02-05T23:45:24.95393+09:00 localhost:1314 GET / 200 echo sample header
```

このように、システム全体としてログとして出力したいheader/query/formなどがあればそれをアクセスログ内に残すことが出来ます。  
使いようによってはとても便利に使うことが出来るかと思います。  

# まとめ
今回はechoのmiddlewareを実装してみました。  
色々と便利な使い方が出来るので、echoを使う場合は使ってみると良いかと思います。  
作ったサンプルは以下のリポジトリにおいています。  

https://github.com/ken-aio/go-echo-sample/tree/v3
