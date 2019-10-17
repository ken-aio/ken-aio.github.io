---
title: "[Golang]テストで特定の処理をモックにしたい場合のインターフェースの使い方"
date: 2019-10-17T13:31:57+09:00
draft: true
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
```
```

# interfaceな変数にDIする

# プロダクションコードとテストコードで実装を切り替える

# まとめ
