---
layout: post
title: "JavaScript 処理系 V8 とレンダリングエンジン Blink のアーキテクチャ"
date: 2021-09-04 00:00:00 +09:00
tags: web
image: /images/profile.png
---

プログラミングおよびプログラミング言語ワークショップ（PPL）が開催したサマースクール「[JavaScript 処理系と Chrome ブラウザの実装技術](http://ppl.jssst.or.jp/index.php?ss2021)」にて、JavaScript 処理系 V8 とレンダリングエンジン Blink のアーキテクチャについて話をしてきました。

<div class="youtube"><iframe src="https://docs.google.com/presentation/d/e/2PACX-1vTbELnS3VWyK6sxxdTwcMWTNouiWm1wgOXBa_4214YOcz5coRTZW04U54DKk7jE2mIb5A31C4kYAxyN/embed?start=false&loop=false&delayms=3000" frameborder="0" width="960" height="569" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe></div>

本発表では、主にブラウザ JavaScript に関する歴史や最新動向に関する概説と、ウェブ特有の実行モデルをどのようにブラウザアーキテクチャへ落とし込んでいるか紹介しました（[スライドへのリンク](https://docs.google.com/presentation/d/e/2PACX-1vTbELnS3VWyK6sxxdTwcMWTNouiWm1wgOXBa_4214YOcz5coRTZW04U54DKk7jE2mIb5A31C4kYAxyN/pub)）

> Chrome ブラウザでは JavaScript 処理系として V8 を、そしてそれを駆動するレンダリングエンジンとして Blink を組み込んでいます。本発表では Chrome がウェブ特有の実行モデルやセキュリティモデルを守りながらどのように V8 と Blink を協調させているのか、そのアーキテクチャを紹介します。
>
> JavaScript は汎用的なプログラミング言語として仕様策定されています。一方で、ブラウザを主たる用途として発展しているのは間違いありません。本講演を通してブラウザから見た言語処理系事情について理解を深めていただければ幸いです。

ウェブの進化とウェブブラウザ開発については[以前の発表](/2020/12/04/web-and-browser)も参考になると思います。これらの発表を通して、ウェブブラウザ開発に少しでも興味を持っていただければ幸いです。