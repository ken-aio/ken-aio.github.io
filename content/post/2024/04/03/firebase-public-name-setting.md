---
title: "Firebaseの公開名を設定する方法"
date: 2024-04-03T10:30:06+09:00
draft: true
keywords: ["firebase"]
description: "firebase"
tags: ["firebase"]
categories: ["firebase"]
author: "ken-aio"
---

<a href="http://b.hatena.ne.jp/entry/" class="hatena-bookmark-button" data-hatena-bookmark-layout="vertical-normal" data-hatena-bookmark-lang="ja" title="このエントリーをはてなブックマークに追加"><img src="https://b.st-hatena.com/images/entry-button/button-only@2x.png" alt="このエントリーをはてなブックマークに追加" width="20" height="20" style="border: none;" /></a><script type="text/javascript" src="https://b.st-hatena.com/js/bookmark_button.js" charset="utf-8" async="async"></script>
<a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button" data-show-count="false">Tweet</a><script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# はじめに
Firebase Authenticationを使っていて、メールをユーザに送信したいケースがありました。

メールを送信する際にメールテンプレート内に `%APP_NAME%` の記述をすると該当のFirebaseプロジェクトのアプリケーション名を設定できるようです。

https://support.google.com/firebase/answer/7000714?hl=ja

しかし、Firebaseコンソールからアプリケーション名を設定する方法が分からなかったので調べてみました。

以下画像の `公開設定` がどうしても表示されませんでした。
<img src="../firebase-console.png" alt="firebase-console" width="600">

# 調べたこと
まず調べると、Firebaseコンソールの「プロジェクトの設定」から「全般」タブにて `公開名` を設定するような情報がいくつか見つかりました。
しかし、私のFirebaseコンソールには `公開名` の設定項目が見当たりませんでした。
色々と調べてもどうやって設定するのかの情報がなく、FirebaseのHelpからGoogleにお問合せをすることにしました。

# 返答
問い合わせの翌日、早速以下のメール文章が返ってきました。

```
Could you try to set your public settings through Google Cloud Console? You may follow the steps below:

1. Go to Cloud Console Credentials.
2. Select your project.
3. Click Edit app.
4. Under the "Application name" section, set your public-facing name. or
5. Select your email in the drop down under "Support email Shown on consent screen for user support".
6. Click Save.

Additionally, I also suggest to verify your "Application logo", make sure that it points the URL to a valid png, jpg, bmp, or single frame gif, less than 1MB.

If the issues persists, provide the HAR file generated from the browser logs when the issue occurred to check if there's any underlying errors on that page.
```

設定がFirebaseコンソール上ではなく、GoogleCloudConsole上で行うようです。

早速、この手順で設定を行っていました。

# 結論
こちらの手順でGoogleCloudから設定をこなうと、Firebaseコンソール上でも無事に `公開名` の設定ができるようになりました。
この設定を行い、無事に目的のメールテンプレートの `%APP_NAME%` にアプリケーション名を設定することができました。

Googleにお問合せをして、翌日に丁寧な返答がもらえてとても助かりました。
同じようにお困りの方がいたら参考になれば幸いです。
