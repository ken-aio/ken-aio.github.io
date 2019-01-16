---
title: "Hugo + Even でGithubブログを作りました"
date: 2019-01-15T16:40:05+09:00
draft: false
keywords: ["ブログ","hugo","even"]
description: ""
tags: ["hugo"]
categories: ["others"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに  
心機一転、ブログを始めるにあたって使ったのが  

* [github pages](https://pages.github.com/)
* [hugo](https://gohugo.io/)

でした。  
理由としては簡単で、ブログの運用をやりたくないので、楽な方法を探してて見つかったのがこの組み合わせでした。  
はてブとかでも良かったのですが、せっかくなのでちょっとは技術的な方がよかったというのもあり。  

hugoは色々なテーマが公開されていて、好きなテーマをインストールしたらすぐに静的なサイトが作れるという優れものでした。  

テーマはファーストインプレッションで

* [Even](https://themes.gohugo.io/hugo-theme-even/)

を選びました。  

本記事ではこのブログの作成手順を記載します。  

# Github pages
Github pagesはgithubのサービスで静的サイトのホスティングが出来る機能です。  
やり方は簡単、公式ページの手順に沿って作っていけばそれだけで自分のgithubアカウントの静的サイトが完成します。  
詳しくは [Github Pages公式サイト](https://pages.github.com/) を参照してください。  

# hugo
hugoは静的サイトのジェネレータです。  
マークダウン形式で内容を書くとそれの静的サイトをビルドしてくれます。  
Golang製というのもあり、非常に軽いです。  
また、preview用の開発サーバも内包されています。  
開発サーバも更新を自動検知してブラウザを自動reloadしてくれるので、非常に簡単にブログを始めることができます。  
便利なのでとてもオススメです。  
  
hugoは非常に簡単に使い始められます。  
macの場合はbrewでインストールできます。  

```
$ brew install hugo
$ hugo version
Hugo Static Site Generator v0.53/extended darwin/amd64 BuildDate: unknown
```

だけです。  
...書くまでもないですねw  

## hugoの初期化
まずはhugoプロジェクトを初期化します。  


```
$ hugo new site {your-site-name}
$ cd {your-site-name}
$ git init
```

すると、サイト名が出来るので、そこに移動します。  

# Even
hugoにはブログだけでなく、様々なテーマが公開されています。  
公開されているテーマは [こちらから](https://themes.gohugo.io/) 参照できます。  
用途に合わせてテーマの選択ができます。  
今回はこのブログのテーマで使った [Even](https://github.com/olOwOlo/hugo-theme-even) の始め方について記載します。  

## Evenテーマのインストール
まずはテーマをインストールします。  
テーマをインストールするには `git submodule` を使います。  

```
$ git submodule add https://github.com/olOwOlo/hugo-theme-even.git themes/even
```

これでEvenがインストールされます。  

## Evenの設定
submoduleを追加しただけだとEvenを使うことはできません。  
詳しくは [Even公式ドキュメントのREADME](https://github.com/olOwOlo/hugo-theme-even) を参照するのがよいです。  
ここでは今回のブログを作成した際の手順を記載します。  

まずは設定のテンプレートをコピーしてきます。  

```
$ cp themes/even/exampleSite/config.toml .
```

これをベースに自分のサイトの設定をしていきます。  
色々とカスタマイズのポイントがありますが、最低限、↓の設定を変更すればよいです。  

```
baseURL = "https://ken-aio.github.io/"
title = "ken-aio's blog"
[params]
  logoTitle = "ken-aio's blog"
[params.social]
  # この辺は全てデフォルトになっているので、適宜、自分のアカウントに変更する
```

# 最初のページを作る
設定が出来たので、最初のページを作ってみます。  
ページを作るのも簡単、hugoコマンドで作れます。  

```
$ hugo new post/helloworld.md
```

あとはhelloworld.mdをカスタマイズしていけばブログが出来ます。  
ローカルでの確認サーバの起動もhugoコマンドで出来ます。

```
$ hugo server -wD
```

`-wD` はwatch と 自動ビルド です。  
これで localhost:1313 にアクセスするとブログが出来ていると思います。  

# Github pagesにデプロイ
最後に本番用のビルドをしてmasterブランチにpushするとGithub Pagesでブログが公開されます。  
※ mdファイル内の `draft` を `false` に設定しないと public には反映されないので注意. (これにちょっとハマりました...w)

```
$ hugo
$ cp -fr public/* /path/to/github_pages_repo/
```

やったね！  

# 自動化するMakefile
本番ビルドとデプロイを自動化するMakefileを↓を元に作りました。(感謝！)  

https://github.com/uqichi/blog/blob/master/Makefile

今回作った [Makefile](https://github.com/ken-aio/ken-aio.github.io/blob/blog/Makefile) の内容です。  
blogを書くブランチが `blog` ブランチ、 github pagesに公開するのが `master` ブランチです。  

```
# from https://github.com/uqichi/blog/blob/master/Makefile
POSTS       := $(wildcard content/post/*.md)
FILE_DIR    := `date +'%Y/%m/%d'`
GITHUB_DIR  := "tmp/ken-aio.github.io"

.DEFAULT_GOAL := help

new: ## Add new post
	@read -p "Enter post name: " f; \
	if [ -z $${f} ]; then echo "file name is empty. so exit"; exit 1; \
	else FILE="post/$(FILE_DIR)/$${f}.md"; \
	fi; \
	hugo new $${FILE}

edit: ## Edit specific post
	@nvim `ls -d $(POSTS) | peco`

deploy: ## Deploy posts
	hugo
	cd tmp && git clone https://github.com/ken-aio/ken-aio.github.io.git
	rm -fr $(GITHUB_DIR)/*
	cp -fr public/* $(GITHUB_DIR)/
	cd $(GITHUB_DIR)/ && git config --local user.name ken-aio && git config --local user.email suguru.akiho@gmail.com
	cd $(GITHUB_DIR)/ && git add . && git commit -m "publish" && git push origin master
	rm -fr $(GITHUB_DIR)

server: ## Run local server
	@hugo server -wD

help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

# Aliases
n:  new
e:  edit
s:  server
```

# まとめ
こんな感じでブログが立ち上がりました。  
作業時間は1〜2時間でした。これでSSL対応のサイトができてしまうなんて、便利な世の中ですね。  

それでは引き続きナニカ書いていきます。  
