---
title: "[Golang]cobraを使って本格的なCLIを作る"
date: 2019-01-27T22:00:00+09:00
draft: false
keywords: []
description: "GolangとcobraでCLIを作る方法です"
tags: ["golang", "cli", "cobra"]
categories: ["golang"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
GolangでCLIツールを作りたくて、ググった結果、cobraを使うことに決めました。  
使ってみた結果、非常によかったです。  
本格的なコマンドを非常に簡単に実装することができました。  
cobraの使い方についてはすでに色々な情報がありますが、自分のためにもまとめておきます。  

# なんでcobra
cobraはメジャーなCLIツールで採用されていたのが一番の理由です。  
このブログを作ってるhugoもcobraを使っていました。  
hugoのコードも参考にさせていただきました。  

* [kurbernetes （kubectl）](https://github.com/kubernetes/kubernetes/tree/master/pkg/kubectl)
* [hugo](https://github.com/gohugoio/hugo)

# cobraの始め方
cobraを始めるにあたり、以下の記事を参考にさせていただきました。  

* [Golangのコマンドライブラリcobraを使って少しうまく実装する](https://qiita.com/tkit/items/3cdeafcde2bd98612428)
* [GolangでwebサービスのAPIを叩くCLIツールを作ろう](https://qiita.com/minamijoyo/items/cfd22e9e6d3581c5d81f)

今回は簡単なサンプルとして、helloコマンドを実装してみます。  

## cobraのインストール
まずはインストールします。...とはいえ、go getするだけです。  
```
$ go get -u github.com/spf13/cobra/cobra
```

## セットアップ
cobraには `add` と `init` の2つのコマンドがあります。  

* init ･･･ cobraプロジェクトの初期化
* add ･･･ 新しいコマンドの追加

### cobraの初期化
まずは初期化をします。initするだけです。  
```
$ cobra init hello
Your Cobra application is ready at
/path/to/go/src/hello

Give it a try by going there and running `go run main.go`.
Add commands to it by running `cobra add [cmdname]`.
```
initすると `GOPATH` にコマンドのプロジェクトが初期化されます。  
プロジェクトは↓の構成です。基本的にはcmdの中にコマンドを実装していきます。  
```
$ tree
.
├── LICENSE
├── cmd
│   └── root.go
└── main.go
```

### helloコマンドの実装
まずは `hello` と打つと `world` と表示するコマンドを作ります。  
`cmd/root.go` を編集します。参考サイトを参考にして、最初の状態からは少しリファクタしています。  

rootCmdの実装  
```
func newRootCmd() *cobra.Command {
	return &cobra.Command{
		Use:   "hello",
		Short: "This is hello command",
		Long:  `This is hello comand long long description`,
		RunE: func(cmd *cobra.Command, args []string) error {
			fmt.Println("world")
			return nil
		},
	}
}
```

rootCmdの初期化
```
var rootCmd = newRootCmd()
```

コマンドのエントリーポイントとなる `Execute` は特に変更していません。  
この状態で実行してみます。  
```
$ go run main.go
world
```
無事にworldと表示されました。  

## サブコマンドの実装
次にサブコマンドを実装してみます。  
`hello world ken` とコマンドを打つと `ハローkenさん` と答えるようにしてみます。  
まずはサブコマンドを追加します。  
```
$ cobra add world
```
すると、cmdの中に `world.go` が出来ます。  

自動生成されたコードにまずはバリデーションを追加してみます。  
引数の名前は必須にしたいので、引数が1より小さい場合はエラーにします。  
引数のチェックは `Args` を使うと行えます。  

```
	Args: func(cmd *cobra.Command, args []string) error {
		if len(args) < 1 {
			return errors.New("requires name")
		}
		return nil
	},
```

実行してみます。  

```
$ go run main.go world ken
world called
$ go run main.go world
Error: requires name
Usage:
  hello world [flags]

Flags:
  -h, --help   help for world

Global Flags:
      --config string   config file (default is $HOME/.hello.yaml)

requires name
exit status 1
```

正しく実行すると、デフォルトのメッセージが表示されます。  
バリデーションに引っかかるとエラーメッセージが表示されて、そのままhelpが出てきます。  
今回、特にhelp関連は何も変更していないのですが、デフォルトでこのhelpメッセージを作ってくれます。  

次にコマンドを実装します。  
デフォルトでは `Run` というFieldに実装するようになっているのですが、エラーが出ても返すことが出来ないので `RunE` を実装するほうがよさそうです。  

```
	RunE: func(cmd *cobra.Command, args []string) error {
		fmt.Printf("ハロー%sさん", args[0])
		return nil
	},
```

実行すると、想定どおりに実行されています。  

```
$ go run main.go world ken
ハローkenさん
```

これでサブコマンドも実装することが出来ました。  

# まとめ
このようにcobraを使えばこれまで使っていたようなコマンドラインツールが簡単に出来てしまいます。  
CLI使ったことない人も是非、試してみて下さい。  
