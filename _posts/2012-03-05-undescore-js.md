---
layout: post
title: "underscore.js の template のデリミタを変更する"
date: 2012-03-05 05:30:00 +09:00
tags: web
image: /images/profile.png
---

注) この記事は [Rails with Underscore.js Templates](http://stackoverflow.com/questions/7514922/rails-with-underscore-js-templates "Rails with Underscore.js Templates") の内容を日本語で要約したものです。

underscore.js を rails で使うと、template のデリミタ `<%= hoge =>` が erb のそれとかぶってしまいうまく動かない。そこで、template で使うデリミタを別のものに変更する。例えば、`{% raw %}{{= hoge }}{% endraw %}` に変更する場合は、あらかじめ次のコードを実行させる。

```js
_.templateSettings = {
    interpolate: /\{\{\=(.+?)\}\}/g,
    evaluate: /\{\{(.+?)\}\}/g
};
```
