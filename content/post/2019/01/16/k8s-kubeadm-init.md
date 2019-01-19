---
title: "kubeadmを使って簡単にk8sクラスターを立ち上げる"
date: 2019-01-16T23:35:25+09:00
draft: true
keywords: ["kurbernetes","kubeadm"]
description: ""
tags: ["kurbernetes"]
categories: ["kurbernetes"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
今回はkubeadmというコマンドを使ってkurbernetesクラスターを立ち上げる方法をまとめます。  
今回利用する環境は以下です。  

| 項目   | 内容         |
|--------|--------------|
| 環境   | AWS          |
| OS     | ubuntu 16.04 |
| master | t3.medium    |
| node   | t3.medium    |

なお、master nodeのrequirementはメモリ2GB以上、CPU2コア以上となっています。  

# kubeadmとは
kubeadm はkubernetesの公式が出しているkubernetesクラスターを簡単に構築できるコマンドです。  
2018/12/4にGAになりました。  
https://kubernetes.io/blog/2018/12/04/production-ready-kubernetes-cluster-creation-with-kubeadm/

# kubeadmのインストール
まずはkubeadmをインストールします  
※ 詳しくは [公式のドキュメント](https://kubernetes.io/docs/setup/independent/install-kubeadm/) を参照  

# masterの構築

# ネットワークプラグインのインストール

## 名前解決の確認

# worker nodeの追加

# podの起動

# まとめ
