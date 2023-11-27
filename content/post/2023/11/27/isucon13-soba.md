---
title: "isucon13に参加しました"
date: 2023-11-27T19:57:46+09:00
draft: false
keywords: [isucon, isucon13]
description: "isucon13に参加しました"
tags: [isucon]
categories: [isucon]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
11/25開催のisucon13に会社の同僚とsobaチームとして参加しました。  
isucon9から参加していたみたいなので今年は5回目の参加でした。  

5回目の参加にして初めて30位以内の26位に入れたので記録を残しておきます。  
[isucon13の結果はこちら。](https://isucon.net/archives/57993937.html)

- 参加者 : 会社の同僚と3人で参加(ken-aio, K A, KURIYO)
- 言語 : golang
- 最終スコア : 58,370

TOP30のチームには
```
スポンサー企業ノベルティ詰め合わせセット・ISUCON13オリジナルTシャツ・ネームカード・認定証をお送りします。
```
とのことなので、楽しみです。

# 準備
申し込みが確定できた9月頃から本番まで大体週１で1~2時間くらいのペースで練習しました。  
練習は [private-isu](https://github.com/catatsuy/private-isu) を使わせていただきました。  

# 感想
今回はDNSが問題として出題されたのがまずはびっくりポイントでした。  
参加したメンバーは普段の業務ではDNSサーバに触れる機会がないため、「これは...」という気持ちになっていました。  

最初に「DNSサーバに手を出すと火傷しそうなので、やることがなくなったらDNSと向き合おう」というのは3人で話していました。  

結果、アプリケーションの改善も終わらないくらいだったので競技時間内ではDNSサーバと向き合う時間は取れませんでした。  

N+1, 画像, 統計取得を改善 + 3台に負荷分散をして最終スコアとなりました。  

# 大体の時系列

## ~10:00
Cloud Formation実行したり、開発環境整えたり、監視ツールを入れたり、マニュアル読んだり、最初にやることをやる。  

## 10:16
golangのコードをサーバから落としてきて最初のcommmit  

## 10:30頃~
最初にベンチを実行、大体3,600点くらいだった。  

nginxのアクセスログとmysqlのslow logをみて最初にやることを決定。  
以下の分担で改善を進めることになった。  

- ken-aio : /iconの取得にアクセスログで総合時間が多かったのでそこの改善を進める
- K A : statisticsが圧倒的に遅かったので改善を進める
- KURIYO : slow queryでindexが足りないテーブルにindexを貼る

## 12:00頃
indexを貼る。  
8,000点くらいまで上昇。  
statisticsの改善がSQLメインになりそうだったのでSQLが得意なKURIYOにバトンタッチ。  
K AはlivecommentのN+1の改善に着手。  

## 13:00頃
/iconについて以下の改善を実施。  

- MySQLからimageのバイナリを削除
- MySQLにimageのsha256を保存
- imageはgolangが動いてるサーバのローカルに保存
- リクエストとsha256の結果が同じ場合は304を返すように変更

大体10,000点くらいになる。  
それでも /icon に一番時間がかかっていたので引き続き改善を対応。  
このタイミングで [fgprof](https://github.com/felixge/fgprof) を導入して改善ポイントの分析を実施。  

## 14:00頃
/icon の画像をサーバのローカルに保存するのをやめてsha256の値とバイナリを全てgolangのメモリに保存するように変更。  
大体12,000点くらい。  
次にN+1問題と/registerが遅い問題の改善に着手。  
golangの外部コマンド呼び出してpower-dnsにレコードを登録する処理があったのでそれの改善を試みた。  

## 15:00頃
KURIYOのstatisticsの改善が完了。20,000点くらいまでスコアが上昇。  
KURIYOは無駄なtransaction外しの改善の対応を進める。  

## 16:00頃
N+1を改善しつつpower-dnsをローカルコマンドからAPI経由に登録処理を変更。  
API経由にしたのでサーバの処理を分散するように変更を開始。  
以下の構成を目指した。  

- サーバ1 : nginx, golang
- サーバ2 : power-dns, mysql(power-dns用)
- サーバ3 : mysql(アプリケーション用)

## 16:30頃
golang -> power-dnsがAPIでどうしても叩けなかったので作戦を変更して以下の構成に変更。  

- サーバ1 : nginx, golang, power-dns
- サーバ2 : mysql(power-dns用)
- サーバ3 : mysql(アプリケーション用)

この構成にして大体45,000点くらいまで上昇。  

## 17:00頃
K AのN+1の対応が完了。 大体53,000点くらいまで上昇。  

## 17:30頃
KURIYOのtransaction外しを入れる。大体55,000点くらいまで上昇。  

## 17:50頃
最後にnginx, mysqlのログを停止、各種ツールの停止等を実施。最後の計測が間に合わずに終わる。  

## 最終スコア
58,000点でfinish

# 最後に
去年に引き続き今年もDNSというびっくり要素がありました。(去年はDBにSQLiteが使われていたのがびっくり)  
今年も楽しく参加させていただいてあっという間の8時間でした。運営の皆様、ありがとうございました。  
来年は20位以内を目指します。  

