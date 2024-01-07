---
layout: post
title: "Chromium では Prefetch や Prerender を総称して Speculative Loading と呼ぶことになった"
date: 2024-01-07 00:00:00 +09:00
tags: web
image: /images/tamagawa-nikaryo-kamigawara-zeki.webp
---

Chromium では Prefetch や Prerender といった[投機的なリソースローディング機能](/2021/05/06/resource-loading-apis)を総称して **Preloading** と呼んでいました。しかし、Preloading という名称は既に広く使われており、特に具体的な API である `<link rel=preload>` や Service Worker の Navigation Preload などと被ってしまうため、評判がよくありませんでした。

<blockquote class="twitter-tweet" data-dnt="true"><p lang="en" dir="ltr">I&#39;ve complained before that &quot;preload&quot; is a heavily overused term in frontend/<a href="https://twitter.com/hashtag/webperf?src=hash&amp;ref_src=twsrc%5Etfw">#webperf</a>, with many conflicting meanings and nuances.<br><br>Now Google will make it even worse by using &quot;preloading&quot; for the upcoming revival of prefetch/prerender: <a href="https://t.co/ZzA7Oy4dBv">https://t.co/ZzA7Oy4dBv</a><br><br>🙄🙄🙄🙄🙄 <a href="https://t.co/TvlO99wUon">pic.twitter.com/TvlO99wUon</a></p>&mdash; Robin Marx (@programmingart) <a href="https://twitter.com/programmingart/status/1692103834691145920?ref_src=twsrc%5Etfw">August 17, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

また、[以前の記事](/2021/05/06/resource-loading-apis)でも紹介した通り、`Pre* APIs` という呼び方もありますが、すべての投機的なリソースローディング機能が Pre プレフィックスを持っているわけではありません（例えば、[Speculation Rules API](https://developer.chrome.com/docs/web-platform/prerender-pages)）し、API ではなく内部実装の最適化機構に対しては使いにくい名前でした。また、ブログ記事などで `Pre*` と表記しても読者にはその意味が伝わりにくいという問題もありました。

そこで、開発チーム内での議論の結果、**投機的なリソースローディングを総称して Speculative Loading と呼ぶことになりました**。この名称はウェブ標準として定められたものではなく、Chromium 開発者が独自に使用している用語になります。用語が馴染まないケースがあればまた違う呼び名に変える可能性もありますが、少なくとも今後 Chromium ではこの用語を使用していきます。

# 具体的な変更

昨年、[DevTools が Speculation Rules をサポート](https://developer.chrome.com/blog/debugging-speculation-rules)しました。この際パネル名などに Preloads という言葉が使われていましたが、名称変更に伴って Speculative loads や Speculations といった文言に[修正されました](https://chromium-review.googlesource.com/c/devtools/devtools-frontend/+/4974805)。リンク先の記事では古い表記になっていますが、いずれ更新されると思います。

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr"><a href="https://twitter.com/programmingart?ref_src=twsrc%5Etfw">@programmingart</a> you may be pleased with the latest change in Chrome Canary: <a href="https://t.co/yBmSauqwkX">pic.twitter.com/yBmSauqwkX</a></p>&mdash; Barry Pollard (@tunetheweb) <a href="https://twitter.com/tunetheweb/status/1718899373168341106?ref_src=twsrc%5Etfw">October 30, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

[MDN の Speculation Rules API](https://developer.mozilla.org/en-US/docs/Web/API/Speculation_Rules_API) のページでは既に新しい表記が使われています。