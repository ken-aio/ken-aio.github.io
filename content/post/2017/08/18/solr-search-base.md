---
title: "Solrと検索のまとめ - 検索編"
date: 2017-08-18T16:30:00+09:00
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
書籍を読んだので、自分なりにSolrを使った検索についてまとめます。  
なお、この記事については以下の書籍を参考に書いています。  
詳細についてはこちらの書籍に記載されているので、気になる方は読んでみてください。  

<iframe style="width:120px;height:240px;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=000000&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=kenaio-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=4774189308&linkId=94e5508b8a56fb5f48fb983a4897fa97"></iframe>

# 検索の基本的な考え方
## 検索精度とは
検索の精度とは「再現率」と「適合率」を用いて表現されます。  

* 適合率(P)
  * PrecisionのP. 検索結果の内、期待したドキュメント数 / 検索結果の期待するドキュメント数
* 再現率(R)
  * RecallのR. 検索結果の内、期待したドキュメント数 / ドキュメント全体の期待するドキュメント数

日本語だけではわかりづらいので、例を書きます。  
以下のような検索対象のドキュメントがあったとります。  

1. よくわかるSolr
2. Solrを使った検索入門
3. Elasticsearchを使った検索入門
4. ElasticsearchとKibanaを使ったBI入門
5. ElasticsearchとSolrの違い

この時、キーワード「Solr 検索」で検索した場合に検索結果にヒットする文章は以下と期待すると定めたとします。  

2. Solrを使った検索入門  
1. よくわかるSolr  

| ランキング | 検索ヒットドキュメント        | 適合有無 |
|------------|:-----------------------------:|---------:|
| 1          | Solrを使った検索入門          | ○        |
| 2          | Elasticsearchを使った検索入門 | ×        |
| 3          | ElasticsearchとSolrの違い     | ×        |
| 4          | よくわかるSolr                | ○        |

この時、再現率と適合率は以下のようになります。

* 上位1件
	* 適合率：P = 1 / 1 = 1.0
	* 再現率： R = 1 / 2 = 0.5
* 上位2件
	* 適合率：P = 1 / 2 = 0.5
	* 再現率： R = 1 / 2 = 0.5
* 上位3件
	* 適合率：P = 1 / 3 = 0.33
	* 再現率： R = 1 / 2 = 0.5
* 上位4件
	* 適合率：P = 2 / 4 = 0.5
	* 再現率： R = 2 / 2 = 1.0

このように、適合率と再現率はトレードオフの関係にあります。

![適合率と再現率](https://upload.wikimedia.org/wikipedia/ja/5/5a/Precision_and_recall.png "適合率と再現率")  
※ 画像はwikipediaから引用しました
[情報検索](https://ja.wikipedia.org/wiki/%E6%83%85%E5%A0%B1%E6%A4%9C%E7%B4%A2)  

## F-measure（F値）
適合率と再現率の調和平均をとって単一の指標で表すことができる値です。

F-measure = 1 / ( 1/2 * (1/P + 1/R) ) = 2PR / (P + R)

上記検索で再現率100%の場合（上位4件）のF値は以下となります。

F-measure = 2 * 0.5 * 1.0 / (0.5 + 1.0) = 0.666.....

このF値が大きいほど性能の良い検索システムといえます。  

# 検索ランキング
再現率と適合率には順序の話は出てきません。  
検索においては再現率と適合率だけでなく、それをどういう順序で並べるか、ということが重要になってきます。  
例えば、Googleで検索した結果が10ページ目以降に存在する場合、それは検索にヒットしなかったとほぼ同然の意味になるでしょう。  
ユーザが求める文章がより上位のランキングに出てくるようにすることは検索において非常に重要です。  

# Solrにおける再現率と適合率
Solrでは検索をフィルタとトークナイザーを用いて実現します。  
これらの設定でSolrにおける再現率と適合率・検索ランキングを調整していきます。  

日本語検索のトークナイザには主に以下の2種類を組み合わせて使うことが多いようです。  

## 日本語トークナイザ
* JapaneseTokenizerFactory
  * 形態素解析の技術を用いて単語分割を行うトークナイザ。意味のある単位で単語を分割します
  * これをSolrで使うことで適合率の改善が期待されます
  * 例：今日の天気は雨です => 今日 / の / 天気 / は / 雨 / です

## N-gram トークナイザ
* NgramTokenizerFactory
  * ある指定された文字数区切りで単語を分割します。よく用いられるのは2文字ずつに分割するBigramです。
  * これをSolrで使うことで再現率の改善が期待されます
  * 例：今日の天気は雨です => 今日 / 日の / の天 / 天気 / 気は / は雨 / 雨で / です

こちらについては以下の記事が非常によくまとまっているため、参考になります。  
[形態素解析とNgramを併用したハイブリッド検索をSolrで実現する方法](http://tech.vasily.jp/entry/solr_hybrid_search)

# Solrにおける検索ランキング
## Solrのランキングのコア
Solrでは検索にヒットした文章のランキング（並び順）にはscoreを用います。  
Solr6ではscoreの計算はBM25Similarityが用いられます。（正確にはLuceneレベルで変更されたみたい）solr5まではTFIDFSimilarityが用いられていました。  
TFIDFについては以下のブログ記事が参考になります  
[特徴抽出と TF-IDF](http://qiita.com/ynakayama/items/300460aa718363abc85c)  
TFIDFとBM25の違いについては以下のブログ記事が参考になります。  
[確率的情報検索 Okapi BM25 についてまとめた](http://sonickun.hatenablog.com/entry/2014/11/12/122806)  
[BM25 The Next Generation of Lucene Relevance](http://opensourceconnections.com/blog/2015/10/16/bm25-the-next-generation-of-lucene-relevation/)  

以下に簡単にまとめます。

* TF
  * Term Frequency. 特定のドキュメントでの単語の出現頻度のこと
* DF
  * Document Frequency. ドキュメント全体での単語の出現頻度のこと
* IDF
  * Inverse Document Frequency. DFの逆数の対数をとったもの.全ドキュメントの中で珍しい単語はより重要であるという考え方

TFIDFからBM25に変わったのはなぜでしょうか？  
これは個人的な考察ですが、SolrのTFIDFはもともと独自の補正が入っていたようで、近年話題になっている機械学習を用いた検索ランキング手法であるLTRでのブースト値がより有用になるように切り替わったのではないかと考えます。  

## ランキング調整
### Function Query
Solrのランキングアルゴリズムに対してファンクションクエリを使うことでブーストさせることができます。  
例えばECサイトで新着商品をより検索の上位に表示したい、といった場合にはファンクションクエリが有用です。  
これをFreshness Boostと呼びます。具体的には以下の記事が参考になります。  
[Boost Solr documents by age](http://www.solrtutorial.com/boost-documents-by-age.html)  

### Re Rank Query
Function Queryを用いることで検索クエリに対して複雑な計算式を用いてBoostしてランキングを出すことが可能です。  
しかし、複雑な計算式を用いるということはそれだけ計算量がかかり、パフォーマンスの面でデメリットが発生してしまう可能性があります。  
SolrのReRankの機能を用いれば複雑な計算式を適用するドキュメント数を限定することが可能です。  
ReRankでは一度オリジナルの式で検索を行なったのち、上位指定件数に対して再度、ReRankQueryを適用し、ランキングを変更します。  
例えば、上位200件に対してfreshness boostの式を適用する、といったことが可能です。  

# まとめ
以上、Solrの検索について色々と調べたことをまとめました。  
検索は奥が深いですね。  
