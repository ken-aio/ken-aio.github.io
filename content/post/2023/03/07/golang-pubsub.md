---
title: "[Golang]Google Cloud PubSubでdocker container内でstreaming pullができない問題の解決方法"
date: 2023-03-06T10:30:00+09:00
draft: false
keywords: [golang, pubsub, gcp]
description: "[Golang]Google Cloud PubSubでdocker container内でstreaming pullができない問題の解決方法"
tags: [golang]
categories: [golang]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
GolangでCloud pubsubのsubscriberを実装した時のことです。  
ローカルでgo runコマンドで実行すると正常にsubするのですが、docker container上だとどうやってもsubしてくれない現象が発生しました。  

なお、subscriberはstreaming pull方式です。  
（詳細は [サブスクリプション タイプを選択する](https://cloud.google.com/pubsub/docs/subscriber?hl=ja) を参照）

Googleで調べてもジャストな記事がヒットしなかったのでここにメモしておきます。  

# 結論
docker containerで ca-certificates をちゃんとアップデートしましょう。  

# 発生した現象
サンプルコードをそのまま動かすようなコードをdocker container上で起動しました。  
以下は記事執筆時点の [こちら](https://cloud.google.com/pubsub/docs/pull?hl=ja#go_2) のドキュメントのコードそのままです。  

```
import (
        "context"
        "fmt"
        "io"
        "sync/atomic"
        "time"

        "cloud.google.com/go/pubsub"
)

func pullMsgs(w io.Writer, projectID, subID string) error {
        // projectID := "my-project-id"
        // subID := "my-sub"
        ctx := context.Background()
        client, err := pubsub.NewClient(ctx, projectID)
        if err != nil {
                return fmt.Errorf("pubsub.NewClient: %v", err)
        }
        defer client.Close()

        sub := client.Subscription(subID)

        // Receive messages for 10 seconds, which simplifies testing.
        // Comment this out in production, since `Receive` should
        // be used as a long running operation.
        ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
        defer cancel()

        var received int32
        err = sub.Receive(ctx, func(_ context.Context, msg *pubsub.Message) {
                fmt.Fprintf(w, "Got message: %q\n", string(msg.Data))
                atomic.AddInt32(&received, 1)
                msg.Ack()
        })
        if err != nil {
                return fmt.Errorf("sub.Receive: %v", err)
        }
        fmt.Fprintf(w, "Received %d messages\n", received)

        return nil
}
```

これを以下のようなDockerfileでビルドしてdocker runしてもpullされませんでした。  
```
FROM    golang:1.20 AS build
WORKDIR /go/src/github.com/ken-aio/pubsub-subscriber
COPY    go.mod go.sum /go/src/github.com/ken-aio/pubsub-subscriber/
RUN     go mod download
ADD     . /go/src/github.com/ken-aio/lcms-pubsub-subscriber
RUN     go build

FROM debian:bullseye
RUN cp /usr/share/zoneinfo/Japan /etc/localtime

COPY --from=build /go/src/github.com/ken-aio/pubsub-subscriber/dist/pubsub-subscriber /pubsub-subscriber

CMD /pubsub-subscriber
```

# 解決
golangのcloud pubsubのstreaming pullにはgRPCのstreamingが使われています。  
調査したところ、以下の環境変数を設定してから起動するとgRPCのログが出力されることがわかりました。  

```
export GRPC_GO_LOG_SEVERITY_LEVEL="INFO"
```

この設定をしてからsubscriberを起動すると、以下のエラーが確認できました。  

```
2023/03/03 12:30:26 WARNING: [core] [Channel #11 SubChannel #12] grpc: addrConn.createTransport failed to connect to {
  "Addr": "pubsub.googleapis.com:443",
  "ServerName": "pubsub.googleapis.com:443",
  "Attributes": null,
  "BalancerAttributes": null,
  "Type": 0,
  "Metadata": null
}. Err: connection error: desc = "transport: authentication handshake failed: tls: failed to verify certificate: x509: certificate signed by unknown authority"
```

ということで、TLSの接続でエラーが出ていました。  

`debian:bullseye` のCA証明書が足りなかったのが原因でした。  

```
RUN apt update && apt -y upgrade && apt install -y ca-certificates
```

こちらをDockerfileに追記してビルドし直したところ、無事にsubscribeできるようになりました。  

