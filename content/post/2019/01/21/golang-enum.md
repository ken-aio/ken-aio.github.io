---
title: "GolangでEnumを作る"
date: 2019-01-21T23:29:51+09:00
draft: false
keywords: ["golang", "enum"]
description: ""
tags: ["golang", "enum"]
categories: ["golang"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
GolangでJavaのEnumっぽい機能が欲しいなと思って調べた結果を残しておきます。  
今回やりたかったことは

* Enum自体はGolangの文法に沿ってcamel caseになっている
* Enum変換元はstringでsnake caseになっている

Webサービスのquery parameterはsnake caseだけど、golangのロジックに入ってくるタイミングではcamel caseになっていてほしい、という感じです。  

# 調べたこと
ググってみると、やはり同じことを思う人は結構いるみたいでした。  
その中でも以下については詳しく見てみました。  

1. [Go言語 - enumを定義する](https://blog.y-yuki.net/entry/2017/05/09/000000)
2. [GoLangでJavaのenumっぽいライブラリ作った話](https://journal.lampetty.net/entry/introducing-goenum)
3. [Re: GoLangでJavaのenumっぽいライブラリ作った話](https://mattn.kaoriya.net/software/lang/go/20141208093852.htm)
4. [4 iota enum examples](https://yourbasic.org/golang/iota/)

1は要件を満たせそうでした。が、enumを作るたびにswitch caseで変換していくのは流石に面倒だな...ということで別の方法を探しました。  

2はiotaがあるのにstructを作るのもなーと思ってやめました。  

3は公式サポートの自動生成でいいと思ったのですが、得られるstringがcamel caseのままだったので、今回の要件には合いませんでした。  

そして最後に出会ったのが4でした。今回の要件的にはこれが一番いいな、ということでこれを採用することにしました。  

以下4の記事から引用  
```
type Suite int

const (
    Spades Suite = iota
    Hearts
    Diamonds
    Clubs
)
```
```
func (s Suite) String() string {
    return [...]string{"Spades", "Hearts", "Diamonds", "Clubs"}[s]
}
```

# 今回採用したEnum
Enum => stringには簡単に変換することが出来ました。  
しかし、string => Enumの変換をうまくすることが出来ませんでした...  
試行錯誤した末、最終的には以下の形に落ち着きました。  

https://play.golang.org/p/Bvc6neemVw9

```
package main

import "fmt"

type Weekday int

const (
	Unknown Weekday = iota
	Sunday
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Saturday
)

func NewWeekday(strWeekday string) Weekday {
	for i, name := range Sunday.Names() {
		if strWeekday == name {
			return Weekday(i)
		}
	}
	return Unknown
}

func (e Weekday) Names() []string {
	return []string{
		"unknown",
		"sunday",
		"monday",
		"tuesday",
		"wednesday",
		"thursday",
		"friday",
		"saturday",
	}
}

func (e Weekday) String() string {
	return e.Names()[e]
}

func switchWeekday(weekday Weekday) {
	switch(weekday) {
	case Monday:
		fmt.Println("Hello monday!!")
	case Friday:
		fmt.Println("Hello friday!!")
	case Unknown:
		fmt.Println("Unknown weekday....")
	default:
		fmt.Println("not found weekday...")
	}
}

func main() {
	fmt.Printf("monday = %+v\n", Monday)

	input := "friday"
	weekday := NewWeekday(input)
	switchWeekday(weekday)

	input = "nothing"
	weekday = NewWeekday(input)
	switchWeekday(weekday)
}
```

ポイント  

* EnumのタイプにUnknownを定義して見つからない場合はUnknownを返すようにした
* NewWeekday でstringをinputにWeekday型のenumを返す
* String()でEnumの文字を取得できる

# まとめ
これで今回の要件を満たせるEnumを定義することが出来ました！  
ただ、やはりGolangでEnumを定義するのは面倒ですね...  

引き続きナニカ書いていきます♪
