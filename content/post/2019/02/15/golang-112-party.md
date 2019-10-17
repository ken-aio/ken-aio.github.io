---
title: "Golang 112 Party"
date: 2019-02-15T19:30:04+09:00
draft: true
keywords: []
description: ""
tags: []
categories: []
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# Golang 1.12 party memo
## go 1.12で何が変わったか @tenten
* ランタイムの変更点
  * GC改善でパフォーマンス
  * メモリアロケーションの改善

* binary-onlyパッケージの最後

* unread
peekで読み込んだ場所を元に戻すことができる関数ができた
=> echoのinoutログで使えるんじゃない？

* replaceAllの追加

* マップの表示がそーと
printlnだけそーとされる

* TLS 1.3サポート
ラウンドトリップが1回少なくなる

* リバースプロキシのwebsocket対応

* go vetがanalysisパッケージを使うようになった
1回だけの共通処理は処理は1回だけ実行される
並列実行できる処理は並列で処理される

# Create a lint tool with analysis @knsh14
go analysisを使うと静的解析が簡単にできるようになる

# go modules
## modulesのモード
goapth mode
これまでと同じくgopathいかに奥

module-aware modeo
どこでもmodulesが使える機能

modulesのキャッシュを使ってCIする方法がブログに書かれてる

replaceでimportする対象に対して、aliasを定義できるようになった
=> githubなどにあげなくてもモジュールの参照ができるようになった

# Go language server
* Language serverは解析を共通でやって、描画を開発ツール側で行う
golsp => goplsに変わった




