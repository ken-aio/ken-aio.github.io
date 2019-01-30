---
title: "GolangとEchoでお手軽APIサーバ"
date: 2019-01-30T21:26:32+09:00
draft: true
keywords: ["golang","echo"]
description: "GolangとEchoでお手軽にAPIサーバを作ってみます"
tags: ["golang","echo"]
categories: ["golang"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
golangとechoを使って簡単にAPIサーバを作る方法をまとめます。  

# 動機
golangでAPIサーバを作りたい場合、どんな選択をするでしょうか  
golangは標準のnet/httpがしっかりしているので、そもそもFWが不要、という議論もよく見かけます。  
しかし、net/httpを触ってみてやっぱりFW使った方がいいなと感じました。  

* リクエストのパースが面倒
* ミドルウェアの組み込みが面倒

など、net/httpも便利ですがやっぱりある程度は自分で書かないといけないと感じました。  
その結果、結局オレオレFWが出来てしまう予感がして、FWを探しました。  
その中でechoに出会いました。  

# Echoとは
golangのWeb FWです。  
一応HTMLもはけるのですが、APIサーバとしてのユースケースが多いんじゃないかなと感じてます。  
公式によると、有名なFWのginよりもパフォーマンスがよいとのこと。  

# API作ってみる
まずはhello worldを返すAPIを実装してみます。  

## インストール

## hello world

## routingを分割

## ロジックを分割

## リクエストのバリデーション

# まとめ
今回は簡単な使い方についてまとめてみました。  
今度、もう少し踏み込んだ使い方もまとめてみたいと思います。  
