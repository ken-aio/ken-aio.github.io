---
title: "[Golang]mapのkeyのちょっとした話"
date: 2019-05-28T23:00:00+09:00
draft: false
keywords: [golang, map]
description: "[Golang]mapのkeyのちょっとした話"
tags: [golang]
categories: [golang]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
Golangのmapのkeyは同一だと評価される場合、同一のkeyとしてみなされます。  
このmapのkeyがどういった場合に同じとみなされるか、時々わからなくなるので、備忘録です。  

# 結論
mapのkeyにstructを使う場合はポインタにしちゃダメだよ、きっと  
(structだけじゃないけどね..)  

# おさらい
mapについておさらいします。  
例えば、以下の場合は同じkeyに対してインクリメントされることになります。  
```
func main() {
	m := map[string]int{}
	m["test"]++
	m["test"]++
	fmt.Println(m)
}
```
結果は以下です。  
```
map[test:2]
```
同じ文字列なら同じkeyとしてみなされます。  
このGolangの仕様はとても直感的だなと思います。  

# keyがstructだったら
これはいわゆるプリミティブ型だけではなく、structでも同じことが言えます。  
例えば、以下の場合は同じstructと判定されます。  
```
type TestStruct struct {
	Str string
	Int int
}

func main() {
	str1 := TestStruct{Str: "test1", Int: 10}
	str2 := TestStruct{Str: "test1", Int: 10}
	fmt.Println("str1 == str2 is ", str1 == str2)
}
```
```
str1 == str2 is  true
```
つまり、同じvalue値を持ったstructは同一であるとみなされます。  
この特性をmapにも適用することが出来ます。  
上記のコードの続きに書きます。  
```
	m := map[TestStruct]int{}
	m[str1]++
	m[str2]++
	fmt.Println(m)
```
これを実行してみると
```
map[{test1 10}:2]
```
変数的には別の変数ですが、同一と判定されるので、mapのkeyとしても同一と判定されてインクリメントされました。  

# ポインタを使ったら
これがポインタだったらどうでしょうか。  
Structを使っている部分をポインタにして実行してみます。  
```
func main() {
	str1 := &TestStruct{Str: "test1", Int: 10}
	str2 := &TestStruct{Str: "test1", Int: 10}
	fmt.Println("str1 == str2 is ", str1 == str2)
	
	m := map[*TestStruct]int{}
	m[str1]++
	m[str2]++
	fmt.Println(m)
}
```
結果は以下です。  
```
str1 == str2 is  false
map[0x40a0e0:1 0x40a0f0:1]
```
見事、別のkeyとしてみなされました。  

# まとめ
ポインタの場合、アドレスが違ったら別物になるので、そりゃ同一とは判定されないよね、とちゃんと考えたら当たり前のことでした。  
Golangで開発している場合、プリミティブ型はそんなにポインタを使わないですが、Structの場合はポインタで操作することが多いと思います。  
このときに何も考えずにポインタのままでmapのkeyに使っちゃうと、中身が一緒でも同じにならないです。  
当たり前のことですが、忘れてしまうことがあったので、備忘録して残しておきます。  

以上でした。
