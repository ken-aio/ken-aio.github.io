---
title: "CircleCIでHugoのブログを自動公開する"
date: 2019-01-16T16:44:14+09:00
draft: false
keywords: []
description: ""
tags: ["hugo"]
categories: ["others"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
[こちら](https://ken-aio.github.io/post/2019/01/15/hugo-even/) の記事でhugoを使ってこのブログを開設した記事を書きました。  
今回はその続きで、記事を書いたらCircleCIで自動ビルドを行なって新しい記事が公開される仕組みを作ったので、そのやり方を記載します。  

# 構成
運用的には1つのgitリポジトリで構成していて、いくつかのブランチとプルリクで運用します。  

* masterブランチ
  * 公開用の記事を置くブランチ
* blogブランチ
  * 作業のベースブランチ
  * このブランチに新しいコミットが入ったら自動で記事をビルドしてmasterに入れる
* post/xxxxブランチ
  * 記事毎のブランチ
  * 書き終わったらこのブランチからblogブランチにプルリク投げてマージする

という感じです。  
このうち、blogブランチへのマージを検知してビルドしてmasterに入れる仕組みをCircleCIを使って作りました。  

# CircleCIの設定
言わずもがな、有名なSaaSのCIサービスです。    
2.0になってdockerが使えるようになってからはますます便利になりました。  
基本的には Makefile を使うのですが、git pushする設定に少し時間がかかりました。  

## Makefile
CircleCIの設定ではないのですが、今回の要なので先に書いておきます。  
deployの部分だけを切り取ってます。  

```
deploy: ## Deploy posts
	git submodule update -i
	hugo
	mkdir -p tmp && cd tmp && git clone git@github.com:ken-aio/ken-aio.github.io.git
	rm -fr $(GITHUB_DIR)/*
	cp -fr public/* $(GITHUB_DIR)/
	cd $(GITHUB_DIR)/ && git config --local user.name ken-aio && git config --local user.email suguru.akiho@gmail.com
	cd $(GITHUB_DIR)/ && git add . && git commit -m "publish" && git push origin master
	rm -fr $(GITHUB_DIR)
```

ポイントとしては、CircleCIとGithubのSSHキーの設定をするので、pushするためのリポジトリはhttpじゃなくてsshで設定しておくことですね。  
それ以外はビルドしてpushしているだけです。  

## yaml
CircleCIは設定にyamlを使います。  
今回の内容はとてもシンプルで

* git checkout
* hugoインストール
* ビルド
* 成果物をmasterにpush

だけをCircleCIにやってもらってます。  
基本機能として、ブランチへのpushをトリガーにビルドパイプラインが動きます。  

↓が今回設定したyamlです。  
※ hugoのコマンドをキャッシュしてキャッシュがなかったらインストール、にしたかったのですが、これはやれておらず...

```
version: 2
jobs:
  deploy:
    working_directory: ~/workspace
    docker:
      - image: circleci/golang:1.11
    environment:
      - GOCACHE: "/tmp/go/cache"
    steps:
      - checkout
      - add_ssh_keys:
          fingerprints:
            - "ae:a5:cb:56:aa:65:2a:f8:38:c4:2a:62:c1:0e:22:2a"
      - run:
          name: install hugo
          command: |
            go get -v github.com/spf13/hugo
      - run:
          name: deploy
          command: make deploy

workflows:
  version: 2
  build:
    jobs:
      - deploy:
          filters:
            branches:
              only: blog
```

ポイントは `fingerprints` の部分です。  

## git push用の鍵設定
cloneするためのdeployキーはボタン一発とても簡単に作れたのですが、push用の鍵はちょっとだけ設定が必要でした。  
まずはCircleCIのページにて、ビルド設定するリポジトリの設定画面に移動します。  
そこに `SSH Permissions` という項目があるので、この画面を開きます。  
※ `Checkout SSH Keys` ではないので注意  

その画面の右上の `Add SSH Key` でGithubに設定しているSSHのprivate keyを入力します。  
Hostは `github.com` です。  

これで設定は完了です。  
この設定をすると、fingerprintsが表示されるので、それをCircleCIの設定ファイルの fingerprints に設定します。  
※ この設定をするまで、CircleCIのビルドでデプロイする際に権限がないと言われてpushできなくてちょっとハマりました...

これでblogブランチにpushされるとCircleCIで自動ビルドが走ってブログ記事が公開されるようになりました。  

やったね！

# まとめ
CircleCIで自動ブログ公開環境が出来ました。  
こんな素晴らしいサービスが利用出来て、ホントに感謝です！  

それではまたナニカ書いていきます。  
