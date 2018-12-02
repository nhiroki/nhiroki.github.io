---
layout: post
title: "Data URI Scheme"
date: 2011-07-01 01:48:00 +09:00
tags: web
image: /images/profile.png
---

Data URI Scheme というものを教えてもらった。画像ファイルを base64 エンコードしたものを HTML ファイルや JS ファイル内に埋め込める技術だそうだ。これを使えば余計なコネクションを貼らずに済むから、画像読み込みのオーバヘッドを軽減できそう。

```html
<img src="data:image/png;base64,***" />
```

**参考**

- [Data URI scheme - Wikipedia, the free encyclopedia](http://en.wikipedia.org/wiki/Data_URI_scheme "Data URI scheme - Wikipedia, the free encyclopedia")
