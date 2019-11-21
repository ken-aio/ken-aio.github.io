---
title: "Go Interface Json"
date: 2019-11-21T18:18:25+09:00
draft: true
keywords: []
description: ""
tags: []
categories: []
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

package main

import (
	"encoding/json"
	"fmt"
)

type MyJSON struct {
	A string `json:"a"`
	B string `json:"b"`
}

var js = `{"a": "test1", "b": "test2"}`

func main() {
	var hoge1 interface{}
	var hoge2 interface{}

	hoge1 = MyJSON{}
	hoge2 = &MyJSON{}

	fmt.Printf("hoge1 = %T\n", hoge1)
	fmt.Printf("hoge2 = %T\n", hoge2)
	fmt.Printf("hoge1 = %T\n", &hoge1)

	hoge11 := hoge1.(MyJSON)
	fmt.Printf("hoge11 = %T\n", &hoge11)

	err := json.Unmarshal([]byte(js), &hoge1)
	fmt.Printf("un 1 = %+v\n", err)
	err = json.Unmarshal([]byte(js), hoge2)
	fmt.Printf("un 2 = %+v\n", err)

	fmt.Printf("hoge1 = %+v\n", hoge1)
	fmt.Printf("hoge2 = %+v\n", hoge2)
}

