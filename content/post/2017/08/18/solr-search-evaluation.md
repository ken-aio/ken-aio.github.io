---
title: "Solrと検索のまとめ - 評価編"
date: 2017-08-18T19:30:00+09:00
draft: true
keywords: ["solr"]
description: ""
tags: ["solr"]
categories: ["search"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
[こちらの記事](https://ken-aio.github.io/post/2017/08/18/solr-search-base/) の続編で検索における評価の方法についてまとめます。  
同じく以下の書籍を参考資料にしています。  

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kenaio-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4774189308&linkId=94e5508b8a56fb5f48fb983a4897fa97"></iframe>

# 検索の評価
Solrの検索精度について色々と試してみてもそれがいいのか悪いのか感覚に頼ってしまうと人によっていい・悪いが異なってしまいます。  
そのため、検索の評価も数値で表せる方法で行う必要があります。

### 検索評価の方法
検索の評価には2つの方法があります。  

* オンライン評価
	* 検索サービスが実際にユーザから利用されている状態で行う評価方法
		* いわゆるA/Bテスト
	* 実際のサービスを使って評価できるため、テストデータの用意がなく本番サービスに新手法を投入すればテストができるため、オフライン評価と比較するとテストがやりやすい
	* 反面、本番サービスの検索が劣化する可能性があり、試行錯誤をスピード感をもって行うことは難しい
* オフライン評価
	* 検索サービスがエンドユーザに公開されていない状態で行う評価方法
	* テスト環境をつかってテストができるため、試行錯誤をスピード感をもって行うことができる
	* 反面、テストデータを準備する必要がある、サービスで使う場合に入ってくる様々な要因が考慮できない
		* 例えば検索ユーザのバックグラウンド（高校生が検索するのと社会人が検索する場合で結果に差異がでる、など）や検索する時期（夏だったら海水浴グッズが検索より多く検索され、冬だったらスノーボードグッズがより多く検索される、など）などの要因が検索精度に影響する可能性があるが、考慮できない（想定しかできない）

ここでは検索のオフライン評価の手法についてまとめます。  

# オフライン評価の前提
いずれのオフライン評価でもテストデータが重要になります。  
テストデータが間違っているとそれを用いて行なった評価もすべて間違ってしまうことになります。  
テストデータとしては以下のようなデータを用意します。  

1. テスト対象のドキュメント群
2. テスト対象のクエリ群
3-1. テスト対象クエリを実行した際の正解ドキュメントと不正解ドキュメントの分類（ラベル付け）※ DCG / nDCG以外
3-2. テスト対象クエリを実行した際のドキュメントとの関連度にスコアをつけて分類（DCG / nDCG）

# 11点補完平均適合率
## 適合率 - 再現率曲線
適合率・再現率・ランキングを加味した評価手法です。  
再現率が0.0 〜 1.0までの間を0.1刻みに適合率の値をだします。  
それらの値を2次元グラフのy軸に適合率を、x軸に再現率をプロットしたグラフを作ります。
このグラフのことを「適合率 - 再現率曲線」と呼びます。  
以下のPDFが参考になります。  
[検索システムの評価](http://compsci.world.coocan.jp/OUJ/2012PR/pr_14_a.pdf)  

## 11点補完平均適合率
「再現率 - 適合率曲線」は1つのクエリに対してのグラフでしたが、11点補完平均適合率では全クエリのグラフの各ポイントについての平均をとったグラフです。  
このグラフが右上にある（=再現率をあげても適合率が下がらない）ものほど検索精度が高い、と言えます。  
昔から存在する評価手法のようです。  

## Precision at K
検索結果の上位k件の適合率を計算した結果で評価を行う手法です。  
例えば、上位10件以内に適合ドキュメントが5件ヒットする場合、Precision at 10は 5 / 10 = 0.5となります。  
直感的でわかりやすい数字ですがクエリ固有の特性に結果が左右されやすいです。  

## Average Precision（AP：平均適合率）
期待する検索結果k件の適合ドキュメントがヒットするまでに上位何件までの検索結果が存在するかに着目した評価手法です。  
検索結果に適合する対象たびに適合率を出し、その算術平均をとった値です。  
詳しくは以下の記事が参考になります。  
[Average Precision（平均適合率）とは](http://skellington.blog.so-net.ne.jp/2012-04-10)  
[推薦システムの基本的な評価指標について整理してみた](https://datahotel.io/archives/4778)  

## Mean Average Precision（MAP）
APの値は1つのクエリに対する結果についての結果ですが、それを全てのクエリに対して評価を行い、その算術平均をとった値です。  
詳しくはAPと同じく以下の記事が参考になります。  
[推薦システムの基本的な評価指標について整理してみた](https://datahotel.io/archives/4778)  

以下は2002年のものですが、わかりやすい文章です。  
[検索実験における評価指標としての平均精度の性質](https://www.google.co.jp/url?sa=t&rct=j&q=&esrc=s&source=web&cd=5&cad=rja&uact=8&ved=0ahUKEwi8zsL9sODVAhVExLwKHSfLCBEQFghEMAQ&url=https%3A%2F%2Fipsj.ixsq.nii.ac.jp%2Fej%2Findex.php%3Faction%3Dpages_view_main%26active_action%3Drepository_action_common_download%26item_id%3D17649%26item_no%3D1%26attribute_id%3D1%26file_no%3D1%26page_id%3D13%26block_id%3D8&usg=AFQjCNHJyXilU-lm8rP5zg1vXZySuEmrbw)  

## DCG / nDCG（Discounted Cumulative Gain / Normalized DCG）
検索の評価手法で近年着目されているもののようです。  
DCGは大雑把に書くとランキングi番目までの関連度のスコアを計算する手法です。  
nDCGには2種類の手法が提案されているようです。どのような検索結果を出したいか、によって使い分ける必要がありそうです。  
[予測ランキング評価指標：NDCGの2つの定義と特徴の比較](http://szdr.hatenablog.com/entry/2017/02/24/235539)  

DCGについては以下の記事が参考になります。  
[【アルゴリズム】Discounted Cumulative Gain（DCG）の理論](http://qiita.com/nnahito/items/af4a677bee4e3f5d66d4)  

DCGは1つのクエリに対する評価手法です。  nDCGはDCGに対して全クエリを適用して正規化した評価手法です。  
理想のDCG（Ideal DCG）の並び順を定義した結果を用いて計算します。  

nDCG = DCG / IDCG  

DCG / nDCGについては以下の記事が参考になります。  
[ElasticsearchとKuromojiを使った形態素解析とN-Gramによる検索の適合率と再現率の向上 (3/3)](http://www.atmarkit.co.jp/ait/articles/1507/29/news010_3.html)
[](http://szdr.hatenablog.com/entry/2017/02/24/235539)  

# まとめ
今回はSolrの検索の評価についてまとめました。  
やはり奥が深いですね。  
ただ、サービスをやってる身としてはいくらオフライン評価でいい結果が出ても、本番のサービスをやってみるまでは本当の結果はわからないので、やはり本番サービスに適用して、A/Bテストで良くしていくのが一番いいのかな、という感想です。  
オンライン評価についてもいつかまとめたい気持ちです。  