---
layout: post
title: "omniauth-facebook による認証でモバイル用ログインページを使う方法"
date: 2012-03-04 18:19:00 +09:00
tags: web
image: /images/profile.png
---

omniauth-facebook を使って OAuth2 認証をするときに Facebook のログインページをモバイル用にする方法 (デフォルトだと PC 用の大きなログイン画面になる)。環境は Rails 3.1.1, devise 1.5.2, omniauth-facebook 1.1.0。

変更する方法は簡単。認証ページへのリンクに display パラメータを与えれば OK。例えば,

```html
<a href="/auth/facebook">Login with Facebook</a>
```

の場合は、

```html
<a href="/auth/facebook?display=touch">Login with Facebook</a>
```

にする。display の他の値は [Dialoge Overview - facebook DEVELOPERS](http://developers.facebook.com/docs/reference/dialogs/#display "Dialoge Overview - facebook DEVELOPERS") に載っている (page, popup, iframe, touch, wap がある)。
