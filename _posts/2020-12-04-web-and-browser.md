---
layout: post
title: "ウェブの進化とウェブブラウザ開発の最前線"
date: 2020-12-04 00:00:00 +09:00
tags: web
image: /images/profile.jpg
---

学部 3, 4 年生向けの特別講義で『[ウェブの進化とウェブブラウザ開発の最前線](https://docs.google.com/presentation/d/e/2PACX-1vQA7761ZtEk8uqs6wQ3sOsXo2B-IGpsvRHHftseFDoPTcE4Jq0TPCuX92GrLeL0D4wkJNhUk0jVxE3V/pub?start=false&loop=false&delayms=3000)』というタイトルで話をしてきました。

<div class="youtube"><iframe src="https://docs.google.com/presentation/d/e/2PACX-1vQA7761ZtEk8uqs6wQ3sOsXo2B-IGpsvRHHftseFDoPTcE4Jq0TPCuX92GrLeL0D4wkJNhUk0jVxE3V/embed?start=false&loop=false&delayms=3000" frameborder="0" width="960" height="569" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe></div>

ウェブの進化の歴史を知ることで現在のトレンドについて理解し、またウェブブラウザというグローバルで大規模なソフトウェアの開発の一端を垣間見ることで、ウェブやウェブブラウザの開発に少しでも興味を持ってくれたら良いなぁという気持ちで話をしてきました。

なお歴史観については私の事実誤認も含まれると思うので、間違いを見つけたら教えて下さい :-)

# 追記 (随時)

たくさんの反応を頂きありがとうございます！次回同じような資料を作るときの参考にできるよう、ここにメモしていきます。ウェブは無限に話せる話題があって楽しいですね！

## ウェブ以前のハイパーテキストの歴史も取り入れるべきでは？

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ありがとうございます！おっしゃるとおりで、ウェブの進化史と言いつつウェブが公開されてからの話しかしていないので片手落ちになってますね。ザナドゥ計画の話とかできると面白そうです。ひとまずブログの方にウェブ以前の話を追記し、スライドを改訂する際にそちらにも取り入れてみます。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1335034990023778304?ref_src=twsrc%5Etfw">December 5, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## レンダリングエンジンの話が抜けているのでは？

90 分 1 コマという授業時間の制約によりレンダリングエンジンの話題は泣く泣くごっそり削ることにしました。本来なら各ブラウザのレンダリングエンジン採用の変遷やブラウザ戦争といった話を取り入れるべきだと思います。特に Chromium が WebKit を Blink へ fork した件は開発者としてもろに影響を受けたので、その話もそのうちどこかに書けると良いな。レンダリングエンジンのアーキテクチャについては既存の資料がいっぱいあるので、学生さんへの任意の課題として最後に付け加えています。

## Flash の話も欲しかった

これは聴講されていた学生さんからも質問がありました。Flash の話もうまくまとめられると良かったのですが、いかんせん Flash 周りの知見が浅いので念入りに調べないと自分ではうまく書けなさそうです。Flash が登場した経緯、Apple が Flash サポートをやめることにした話、そしてブラウザ全体でサポートをやめることになった話を書くと良いのかな？Flash の生み出した文化についても書くと楽しそうだけど、それは脱線し過ぎかもしれない。関連して Java アプレット、ActiveX や Silverlight の話もできると良さそう。

ブラウザプラグインという観点から見ると、NPAPI / PPAPI → NaCl / asm.js → Web Assembly という変遷についても面白い話題です。

## その他歴史上の話で面白そうなもの

- Net News
- FTP
- WaSP (The Web Standard Project)
- HTML5 策定の話
- W3C と WHATWG の話
- Facebook が HTML5 を採用しようとした話
- Extensions によるブラウザの機能拡張
- (何かあれば教えてください)