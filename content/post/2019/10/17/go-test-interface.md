---
title: "[Golang]テストで特定の処理をモックにしたい場合のインターフェースの使い方"
date: 2019-10-17T13:31:57+09:00
draft: false
keywords: [golang, test, interface]
description: "[Golang]テストで特定の処理をモックにしたい場合のインターフェースの使い方"
tags: [golang]
categories: [golang]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
最近は会社でGolangの利用が少しずつ広がっています。  
そんな中、一人の同僚から、「Golangでプロダクションコードでは期待通りの動きをさせて、テストだとモックにする場合のGolangのやり方を教えて欲しい。」という要望がありました。  
せっかくなので、ブログにまとめてみます。  
対象はオブジェクト思考な言語の経験はあるけどGolangの経験は浅いような方です。  

# 結論
簡単に書くと以下のことをやればプロダクションコードとテストコードで動作を分けることができます。  

* 動作を分けたい対象をinterfaceを使って抽象化する
* プロダクションコードでは対象のinterfaceにプロダクション用の実装をDIする
* テストコードでは対象のinterfaceにテスト用の実装をDIする

ということです。  
知っている人は「そんなの当たり前じゃん！」と思うかもしれませんが、この記事ではこれをGolangでやるにはどうしたらよいのか、具体的なコードを用いて紹介していこうと思います。  

# DIとDIコンテナ
まずはDIとDIコンテナについて整理します。  
Java界隈の人がDIという単語を見ると、DIコンテナのことを思い浮かべてしまう人が多いかもしれません。（かく言う自分もDI = DIコンテナと思っていた時期はありました...）  

例えば、JavaのSpringを使うとDIしたい対象に特定のアノテーションをつけると、DIコンテナがよきに計らって特定のタイミングでDIしてくれます。  
そのため、アプリケーションのコードでは対象のオブジェクトの初期化などを考えずに利用することが可能となります。  
SpringのDIサンプル。 `@Autowired` というのがDIコンテナにDIしてね、という目印になります。  
```
@Component
public class UserServiceImpl implements UserService {
    @Autowired
    private UserRepository userRepository;
}
```
このようにDIコンテナを使っていると、上記の `userRepository` はアプリケーションコード中ではどこでも初期化していないのに利用することが可能になります。  
これを見るとDI(コンテナ)ってブラックボックスでよくわらかない...というイメージを持ってしまうのもしょうがないのかなと思ったりします。  

ちょっと前置きが長くなりましたが、DIとDIコンテナは以下のように分類できると理解しています。(もし間違ってたらご指摘ください)  

* DI ・・・ 特定の変数に対して何かしらの具体的なオブジェクトを初期化された状態で渡すこと
* DIコンテナ ・・・ DIを自前（アプリケーションコード中で）行うことなく、DIコンテナから欲しいオブジェクトが初期化された状態で受け取ることを出来るようにする仕組み

つまり、DIコンテナがなくてもDIをすることは出来ます。（面倒になるけど）  

# Golangでのinterfaceの使い方
次に簡単にinterfaceについて紹介しておこうかと思います。  
知ってるよ！と言う人は読み飛ばしちゃってください。  

まずは超基礎、interfaceの定義の仕方です。  
このようにメソッドと引数と返却値の形を決めるものです。  
```
type MyInterface interface {
	Method(args []string) (string, error)
}
```

次にinterfaceについて具体的な実装をしてみます。  
```
type ImplA struct{}
func (a *ImplA) Method(args []string) (string, error) {
	return fmt.Sprintf("I am A"), nil
}

type ImplB struct{}
func (a *ImplB) Method(args []string) (string, error) {
	return fmt.Sprintf("I am B"), nil
}
```
このようにGolangではどのinterfaceの実装をするか、明示的に宣言することなくそのinterfaceを満たしていれば実装されているとみなされます。  
これらのチェックはコンパイル時に行われるため実行は高速になります。  

さて、実際にこのinterfaceの実装を使ってみます。
```
func main() {
	var a, b MyInterface
	a = &ImplA{}
	b = &ImplB{}
	fmt.Println(a.Method([]string{}))
	fmt.Println(b.Method([]string{}))
}
```
実行してみます。  
```
I am A <nil>
I am B <nil>
```
どちらも `MyInterface` 型ですが、結果はそれぞれ具体的な実装である A と B の結果が出力されましたね。  

# interfaceな変数にDIする
次に上記で作ったinterfaceに対してDIをしてみようと思います。  
今回は簡単なサンプルにするため、関数の引数に対してDIしてみます。  
まずはprintする簡単な関数を定義します。  
```
func PrintAny(any MyInterface) {
	msg, err := any.Method([]string{})
	if err != nil {
		return
	}
	fmt.Println("any message is ", msg)
}
```
ポイントは引数を具体的なstructではなくinterfaceで受け取っているところです。  
このinterfaceに対してDIしてみます。  
```
	var a, b MyInterface
	a = &ImplA{}
	b = &ImplB{}
	PrintAny(a) // AをDI!!
	PrintAny(b) // BをDI!!
```
最初の `PrintAny` は A を DI しています。2番目は B を DI しています。  
「え、これだけ？」と思うかもしれませんが、これも立派なDIになります。  
このように実装すると何が嬉しいのかというと、 `PrintAny` 関数の `MyInterface` が関数内部でインスタンスを生成していないことで実装を入れ替えることができることです。  
ただ、今回のサンプルでは実装を入れ替えることにあまりメリットを感じれないかと思います。  
次にDIのメリットがあるようなサンプルを記載します。  

# プロダクションコードとテストコードで実装を切り替える
ここでは以下のようなサンプルを考えます。  

* 現在時刻を取得
* Unix時間に変換
* Unix時間が偶数なら `Even` を、奇数なら `Odd` を返す関数を定義する

この問題を単純に解くのであれば以下のようなコードになります。  
```
func UnixTimeSample() string {
	unixNow := time.Now().Unix()
	if unixNow%2 != 0 { // !!
		return "Even"
	} else {
		return "Odd"
	}
}
```
簡単ですね。次にこの `UnixTimeSample` の関数をテストしてみましょう。  
```
func TestUnixTimeSample(t *testing.T) {
	expected := "Odd"
	result := UnixTimeSample()
	if result != expected {
		t.Errorf("I am not %s", expected)
	}
	fmt.Println("result is", result)
}
```
しかし、このテストは `green` になったり `red` になったりします。  
```
$ go test
result is Even
PASS
ok      github.com/ken-aio/go-interface-sample  0.017s
$ go test
result is Odd
--- FAIL: TestUnixTimeSample (0.00s)
    main_test.go:12: I am not Odd
FAIL
exit status 1
FAIL    github.com/ken-aio/go-interface-sample  0.010s
```
まあ当たり前ですよね、現在時刻に依存しているので。  
このように外部の要素に依存するものを関数内部で生成してしまうとユニットテストがうまくできなくなってしまいます。  
そこで、DIパターンを使って外から外部依存する対象を注入してしまえばユニットテストでロジックのテストを行うことができるようになります。  
リファクタリングしてみましょう。今回は対象が時間を扱っています。そこで、 `TimeManager` を導入してみます。  
```
type ITimeManager interface {
	Now() time.Time
}
```
現在時刻を取得する `ITimeManager` というインターフェースを定義しました。  
次にプロダクションコードを実装してみます。  
```
type TimeMenager struct {}
func (t *TimeManager) Now() time.Time {
	return time.Now()
}
```
プロダクションコードでは標準の現在時刻を取得する関数を呼び出しているだけです。  
`UnixTimeSample` で使っている現在時刻取得を `TimeManager` 経由で行うようにしてみます。  
```
func UnixTimeSample(timeManager ITimeManager) string {
	unixNow := timeManager.Now().Unix()
	if unixNow%2 != 0 { // !!
		return "Even"
	} else {
		return "Odd"
	}
}
```
メソッド引数で受け取るようにしました。  
そして、最後にメイン関数で `TimeManager` をDIします。  
```
func main() {
	fmt.Println(UnixTimeSample(&TimeManager{}))
}
```
試しに実行してみましょう。時間に依存するのでタイミングによってそれぞれ表示が変わりますね。  
```
$ go run main.go
Even
$ go run main.go
Odd
```
これでプロダクションコードは挙動を変えることなく `TimeManager` の導入ができました。  
次にテストコードでロジックのテストをしてみます。  
まずは `TimeManager` のモックを導入します。  
```
type MockTimeManager struct {
	MockTime *time.Time
}

func (t *MockTimeManager) Now() time.Time {
	if t.MockTime == nil {
		return time.Now()
	}
	return *t.MockTime
}
```
この `MockTimeManager` では外から好きな時間をNowとして取得可能なようにしています。  
これを使ってテストを実行してみます。テストはテーブルドリブンテストの手法で作ってみます。  
```
func TestUnixTimeSample(t *testing.T) {
	cases := []struct {
		t        time.Time
		expected string
	}{
		{t: time.Unix(1419933531, 0), expected: "Odd"},
		{t: time.Unix(1419933530, 0), expected: "Even"},
	}
	mockTimeManager := &MockTimeManager{}

	for _, c := range cases {
		mockTimeManager.MockTime = &c.t
		result := UnixTimeSample(mockTimeManager)
		if result != c.expected {
			t.Errorf("expected is %s but I am %s", c.expected, result)
		}
	}
}
```
これを実行してみると、見事にfailしました。  
```
$ go test
--- FAIL: TestUnixTimeSample (0.00s)
    main_test.go:22: expected is Odd but I am Even
    main_test.go:22: expected is Even but I am Odd
FAIL
exit status 1
FAIL    github.com/ken-aio/go-interface-sample  0.010s
```

# まとめ
今回は対象としてわかりやすいと思う時間についてGolangのinterfaceを使ってプロダクションコードとテストコードで実装を入れ替えるDIのやり方を紹介しました。  
DIについて理解できたでしょうか？  
実際にやってみると意外と簡単だった思うのではないでしょうか。  
今回のサンプルは以下のリポジトリにコードを置いています。  
https://github.com/ken-aio/go-interface-sample

もしDIの対象が増えて自前で管理するのが辛くなってきた時はDIコンテナの出番です。  
GolangではDI管理についてはGoogleが作っている `wire` というライブラリを使っている例をよく見かけます。（自分では使ったことはありません...）  
https://github.com/google/wire  
比較的大きなプロジェクトで色々な場面でDIを使う必要が出てきた場合は導入の検討をしてみると良いかもしれません。  

本記事が誰かしらの役に立てば幸いです。  
ではまたいつか  
