---
title: "[Golang]Restyで手軽にAPIリクエストを送る"
date: 2019-02-07T23:00:00+09:00
draft: false
keywords: ["golang", "resty"]
description: "[Golang]Restyで手軽にAPIリクエストを送る"
tags: ["golang", "resty", "echo"]
categories: ["golang", "API"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


# はじめに
今はマイクロサービスアーキテクチャでシステムを作るのが定石になってきている時代です。  
最近はJSONのAPIだけではなく、GraphQLやgRPCなどの連携方法が登場していますが、JSONのREST APIを使っているシステムもまだまだ多いかと思います。  

GolangでJSON形式のAPIリクエストを送ってレスポンスを受け取る場合、 `net/http` の `http.Client` あたりを使うのが定石かと思います。  
これをラップしてより使いやすくしてくれているライブラリが `Resty` です。  

# Resty
Restyの機能は [githubのREADME](https://github.com/go-resty/resty#features) を見ると一覧化されています。  
この中でもより有用と思うのは  

* APIに失敗した場合のリトライが非常に簡単にかける(Exponential Backoffアルゴリズム)
* request / responseのパラメータがmapやstructで行える
* requestの前後にhookを仕掛けることができる
* テストが簡単に出来る

あたりかと思います。  
これらの機能があるのでAPI呼び出しを非常に簡単に行うことができます。  

# 使ってみる
早速使ってみます。  
サンプルコードとしてサーバとRestyを使ったクライアントを実装します。  

## 簡単なAPIを叩いてみる
### サーバ
サンプルのサーバは `Echo` を使って実装します。  
Echoについては [過去の記事](https://ken-aio.github.io/tags/echo/) あたりを参照してみてください。  

まずはとても簡単なServerAPIを実装します。
```
	e.GET("/", hello)
	e.POST("/", postHello)
	e.PUT("/", putHello)
	e.DELETE("/", deleteHello)
...
func hello(c echo.Context) error {
	log.Println("hello action")
	return c.JSON(http.StatusOK, map[string]string{"hello": "world"})
}

func postHello(c echo.Context) error {
	return c.JSON(http.StatusOK, map[string]string{"hello": "post"})
}

func putHello(c echo.Context) error {
	return c.JSON(http.StatusOK, map[string]string{"hello": "put"})
}

func deleteHello(c echo.Context) error {
	return c.JSON(http.StatusOK, map[string]string{"hello": "delete"})
}
```

起動してcurlを実行してみると以下のような結果が得られます。  

```
$ curl -X GET localhost:1314
{"hello":"world"}
$ curl -X POST localhost:1314
{"hello":"post"}
$ curl -X PUT localhost:1314
{"hello":"put"}
$ curl -X DELETE localhost:1314
{"hello":"delete"}
```

### クライアント
簡単に実装してみます。  
以下のコードだけで、上の `curl` と同じことが実現出来ます。  
```
func main() {
	resp, _ := resty.R().Get("http://localhost:1314")
	fmt.Printf("Get resp = %+v\n", resp)
	resp, _ = resty.R().Post("http://localhost:1314")
	fmt.Printf("Post resp = %+v\n", resp)
	resp, _ = resty.R().Put("http://localhost:1314")
	fmt.Printf("Put resp = %+v\n", resp)
	resp, _ = resty.R().Delete("http://localhost:1314")
	fmt.Printf("Delete resp = %+v\n", resp)
}
```

実行結果は  
```
$ go run client/main.go
Get resp = {"hello":"world"}
Post resp = {"hello":"post"}
Put resp = {"hello":"put"}
Delete resp = {"hello":"delete"}
```

です。ばっちりAPIを叩けていますね。  

## 高度な機能を試す
これだけだとつまらないので、もう少し高度な機能を試してみます。  
今回試すのは

* structへのbind
* リトライ

をやってみます。  

### サーバ
まずはstructのbindの確認のため、テストサーバを実装します。  
```
// Req is request body
type Req struct {
	Query string `json:"query"`
}
...
	e.GET("/param", getParam)
	e.POST("/param", postParam)
...
func getParam(c echo.Context) error {
	r := &Req{}
	if err := c.Bind(r); err != nil {
		return err
	}
	return c.JSON(http.StatusOK, map[string]interface{}{"success": true, "message": "good get", "param": r.Query})
}

func postParam(c echo.Context) error {
	r := &Req{}
	if err := c.Bind(r); err != nil {
		return err
	}
	return c.JSON(http.StatusOK, map[string]interface{}{"success": true, "message": "good post", "param": r.Query})
}
```

### クライアント
queryパラメータとbodyはmapやstructを渡すとそのままqueryパラメータやbodyとしてrequestを渡してくれます。  
```
func structBindTest() {
	resp, err := resty.
		R().
		SetQueryParam("query", "this is query param").
		Get("http://localhost:1314/param")
	if err != nil {
		log.Printf("get err: %+v", err)
	}
	fmt.Printf("Get resp = %+v\n", resp)

	resp, err = resty.
		R().
		SetBody(&Req{Query: "this is body param"}).
		Post("http://localhost:1314/param")
	if err != nil {
		log.Printf("get err: %+v", err)
	}
	fmt.Printf("Get resp = %+v\n", resp)
}
```

実行して見ると以下の結果が得られます。  

```
$ go run client/main.go
Get resp = {"message":"good get","param":"this is query param","success":true}
Get resp = {"message":"good post","param":"this is body param","success":true}
```

### Backoff確認
Restyは [Exponential Backoffアルゴリズム](https://en.wikipedia.org/wiki/Exponential_backoff) でretryを行うことが出来ます。  

Exponential Backoffアルゴリズムは要するにリトライ間隔を少しずつ伸ばしていって、最後に諦める、というようなアルゴリズムです。  
詳しくは以下のブログが参考になります。  

* [AWSユーザーは必ず覚えておきたいExponential Backoffアルゴリズムとは何か](https://yoshidashingo.hatenablog.com/entry/2014/08/17/135017)

ここではクライアントだけを実装します。  

```
func retryTest() {
	log.Println("start")
	retryNum := 5
	resty.
		SetRetryCount(retryNum).
		SetRetryWaitTime(1 * time.Second).
		SetRetryMaxWaitTime(5 * time.Second).
		R().
		Get("http://dummyurl")
	log.Println("end")
}
```

実行してみます。  
```
$ go run client/main.go
2019/02/07 23:26:42 start
RESTY 2019/02/07 23:26:42 ERROR Get http://dummyurl: dial tcp: lookup dummyurl: no such host, Attempt 1
RESTY 2019/02/07 23:26:43 ERROR Get http://dummyurl: dial tcp: lookup dummyurl: no such host, Attempt 2
RESTY 2019/02/07 23:26:44 ERROR Get http://dummyurl: dial tcp: lookup dummyurl: no such host, Attempt 3
RESTY 2019/02/07 23:26:47 ERROR Get http://dummyurl: dial tcp: lookup dummyurl: no such host, Attempt 4
RESTY 2019/02/07 23:26:51 ERROR Get http://dummyurl: dial tcp: lookup dummyurl: no such host, Attempt 5
2019/02/07 23:26:56 end
```

ログを見てみると、リトライ間隔が少しずつ伸びていって、最後は5秒のタイムアウトで終わっていることが確認できます。  

# まとめ
Restyを使うとただリクエストを送ってレスポンスを受け取るだけではなく、様々な付加要素を得ることが出来ます。  
今回は記載していないですが、hook機能もなかなか便利です。  
例えば、共通の認証をhookさせて認証が有効な場合はビジネスロジックに関係なく認証機構をhookにいれる、などで使えます。  

マイクロサービスでJSON APIを使う場合はRestyの利用を検討してみてください。

今回のサンプルは以下においています。

https://github.com/ken-aio/go-echo-sample/releases/tag/v4
