---
layout: post
title:  "Chromium ソースコード珍百景"
date: 2017-12-26 00:00:00 +09:00
tags: codereading web
image: /images/chromium-funny-sourcecode-fine-day.jpg
---

これは [Chromium Browser アドベントカレンダー](https://qiita.com/advent-calendar/2017/chromium) **26 日目**の記事です。穴が空いたときの代理投稿のために用意してあったのですが無事に使わずに済んだので 26 日目として公開します。

本記事では Chromium のソースコードを眺めていて見かけた珍百景を紹介します。実用性は皆無です。他に見つけたら随時追加します。

ちなみに Chromium のソースコードの読み方については[本アドベントカレンダー 1 日目の記事](https://nhiroki.jp/2017/12/01/chromium-sourcecode)で紹介しました。みなさんも珍百景を見つけたらこっそり教えてください :D

# 天気の良い日

天気が良い日は陽気な気分でコードを書けますよね。

![天気の良い日](/images/chromium-funny-sourcecode-fine-day.jpg)

Web Platform Tests で使用するサブドメインは自由に設定できるのですが、その一つが「天気の良い日」になっています。理由は不明。なぜかフランス語のサブドメインも設定されていて、そっちはどうやら「生徒」という意味らしいです。こちらも理由は不明。

# pen pineapple apple pen

![pen pineapple](/images/chromium-funny-sourcecode-pen-pineapple.jpg)

テストのダミーレスポンスのカスタムヘッダーに仕込まれた pen pineapple apple pen。ボディはなぜか Hello world。

# Kotori Otonashi

テストに突然現れる Kotori Otonashi。一体誰なんだ！？

![Kotori Otonashi](/images/chromium-funny-sourcecode-kotori-otonashi.jpg)

コードレビューアーの「I am ok with anime characters in our tests.」というコメントがじわじわ来ます ([レビューページ](https://bugs.webkit.org/show_bug.cgi?id=87352))。

![Kotori Otonashi](/images/chromium-funny-sourcecode-kotori-otonashi-review.jpg)

WebKit 時代のパッチですが、Blink フォーク後もしっかり残っています。