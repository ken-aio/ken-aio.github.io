---
title: "Kubernetesで使っているすべてのNodeportを取得する"
date: 2019-05-29T22:56:10+09:00
draft: true
keywords: [kubernetes, nodeport]
description: ""
tags: [kubernetes]
categories: [kubernetes]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
備忘録です。  

普段はクラウドのkubernetesではなく、自前でインストールしたkubernetesを使っています。  
自前Kubernetesを使っていると、開発環境から気軽に外部からserviceにアクセスするためにNodePortを使うことがあります。  
NodePortはk8sクラスター内でユニークである必要があるので、どのポートが使われているか知っておく必要があります。

そこで、ワンライナーで使っているNodePortのport番号を取得するコマンドを作りました。

# コマンド
```

```

# まとめ
誰か使うことあるのかな？  
良かったら使って上げてください。  

では。
