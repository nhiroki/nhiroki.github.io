---
layout: post
title: "WFH で Audio Worklets の使用率が増えている話"
date: 2020-05-12 00:00:00 +09:00
tags: web text
image: /images/profile.png
---

COVID-19 の流行によってオンライン会議システムが急速に普及しています。これに伴い、会議システムの音声処理などで使われている Audio Worklets API の使用率がここ数ヶ月で急激に増えています[^tweets] [^zoom]。

次のグラフは Chrome 上でのページロードに対する Audio Worklets API の使用率を表しています[^addmodule]。元データは [Chrome Platform Status](https://www.chromestatus.com/metrics/feature/timeline/popularity/2261) で見ることができます。

![Audio Worklets API の使用率](/images/usage-of-audio-worklets-graph.png)

Audio Worklets API は本記事執筆時点で Chromium ベースのウェブブラウザ (Chrome や Edge など) と Firefox で使うことができます。 Firefox は元々バージョン 77 でローンチする予定だったものを、サービスプロバイダからの要請を受けて[バージョン 76 に前倒ししてローンチ](https://hacks.mozilla.org/2020/05/firefox-76-audio-worklets-and-other-tricks/)したようです。前述の通り、WFH (Work From Home) へのシフトによってウェブにおける音声処理の必要性が高まったことが背景にあります。

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">&quot;Nightly is now 77, but there is demand for this feature including<br>some WFH benefits from one service provider, and so we are looking to uplift the pref switch to 76.&quot;</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1250346721541894145?ref_src=twsrc%5Etfw">April 15, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Chromium においては、チームメイトと私で 2016 年から 2018 年頃に Worklets の共通基盤 (スレッディングやモジュールスクリプトの実行処理) を[設計実装](https://docs.google.com/document/d/1RIMCo_xejzvm0BlJdhAg2nfpdOE7xy_ijHVXk6kPHws/edit?usp=sharing)しました。並行して別のチームが Audio Worklets の API レイヤーを共通基盤の上に実装して [Chrome 66 でローンチ](https://www.chromestatus.com/feature/4588498229133312)しました。この辺りの経緯は以前「[就職して 6 年過ぎた](/2018/04/06/six-years-reflection)」という記事に書きました。

ローンチしてからつい最近まで Chromium でしか実装されておらず、その認知度も低かったため使用率は微々たるものでした。上のグラフの平たい部分ですね。それが音声処理需要の増加によって使用率が一気に増加したのが今です。

当時これを担当したのは技術的に面白そうだったからというだけの理由でしたが、それが社会活動を支える部品として広く使われるようになったことにとても驚いていますし、このような状況になる前に必要な機能を揃えられたのはとても幸運だったと思います。もしブラウザ開発を通して誰かの一助になり、困難な状況にある社会を良くすることに少しでも貢献できているのならばとても嬉しいことです。

# 注釈

[^tweets]: 本記事は [Twitter に投稿した内容](https://twitter.com/nhiroki_/status/1258397865673551872)を後から見直せるように記事にしたものです。
[^zoom]: たとえば Zoom の Web クライアントで使われています ([source](https://devforum.zoom.us/t/firefox-computer-audio-not-supported-dont-lie-to-me/14597))
[^addmodule]: `AudioWorklet#addModule()` を一度でも呼ぶと Audio Worklets API を使用しているとみなします。