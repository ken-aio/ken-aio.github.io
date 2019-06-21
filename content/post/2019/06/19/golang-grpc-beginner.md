---
title: "Golang Grpc Beginner"
date: 2019-06-19T15:21:15+09:00
draft: true
keywords: []
description: ""
tags: []
categories: []
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
今更ながらですが、Golangを使ってgRPCサーバを立ててみました。  
この記事ではセットアップから立ち上げる方法についてまとめました。  

# 諸々のインストール
最初にgRPCを使うためのライブラリ群をインストールします。  
必要なライブラリは以下です。  

* protoc
* protoc-go-gen
* grpc

まずはgrpcをインストールします。  
```
$ go get -u google.golang.org/grpc
```

protocはprotoファイルの基本的なコンパイラのようです。  
Macの場合はhomebrewでインストールできます.  
```
$ brew install protobuf
```

protobug以外はプラグインとして提供されています。  
それぞれ、golangのgenerator, documentのgenerator, grpcのgeneratorです。  
いずれも `go get` のコマンドでインストールします。  
```
$ go get -u github.com/golang/protobuf/protoc-gen-go
```

# protoの定義
まずはprotocを定義します。  

# generate
protoからgolangのコードを自動生成します。  

# 起動とテスト
簡単に動作確認するには `grpcurl` のコマンドを使うのがいいかと思います。  

# まとめ
GolangでgRPCができました。  
この感じだとAPI IFの定義をprotoでかけて型もあってtype safeにもなりパフォーマンスも向上するため、メリットが大きいと感じました。  
ただ、clientもgRPCに対応する必要があるので、そちらも考慮する必要はありますね。  

# 参考
https://qiita.com/marnie_ms4/items/4582a1a0db363fe246f3
