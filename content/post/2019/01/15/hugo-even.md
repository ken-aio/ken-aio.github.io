---
title: "Hugo + Even テーマでブログを作りました"
date: 2019-01-15T16:40:05+09:00
draft: true
keywords: ["ブログ","hugo","even"]
description: ""
tags: []
categories: []
author: "ken-aio"
---

# はじめに  
心機一転、ブログを始めるにあたって使ったのが  

todo link
* [github pages][https://pages.github.com/]
* hugo

でした。  
理由としては簡単で、ブログの運用をやりたくないので、楽な方法を探してて見つかったのがこの組み合わせでした。  
はてブとかでも良かったのですが、せっかくなのでちょっとは技術的な感じがよかったので。  

hugoは色々なテーマが公開されていて、好きなテーマをインストールしたらすぐに静的なサイトが作れるという優れものでした。  

テーマはファーストインプレッションで

todo link
* even

を選びました。

本記事ではこのブログの作成手順を記載します。  

# github pages

# hugo
hugoのインストールは非常に簡単です。  
macの場合はbrewでインストールできます。  

```bash
$ brew install hugo
$ hugo version
Hugo Static Site Generator v0.53/extended darwin/amd64 BuildDate: unknown
```

だけです。  
...書くまでもないですねw  

# even
hugoにはブログだけでなく、様々なテーマが公開されています。  
公開されているテーマは [こちらから](https://themes.gohugo.io/) 参照できます。  
用途に合わせてテーマの選択ができます。  
今回はこのブログのテーマで使った [Even][https://github.com/olOwOlo/hugo-theme-even] の始め方について記載します。  

# まとめ
こんな感じでブログが立ち上がりました。  
作業時間は1〜2時間でした。これでSSL対応のサイトができてしまうなんて、便利な世の中ですね。  

それでは引き続きナニカ書いていきます。
