---
layout: post
title:  "Chrome 46 の Service Worker の変更点"
date:   2015-12-08 00:00:00
tags: web
---

本記事は Chrome 46 の Service Worker のリリースノートを日本語に翻訳したものです。原文は service-worker-discuss グループで読むことができます。

- [[service-worker-discuss] Release Notes for Service Worker in Chrome 46](https://groups.google.com/a/chromium.org/forum/#!topic/service-worker-discuss/bDt9xrONWwE)

以前のバージョンのリリースノートはこちら。

- [Chrome 45 の Service Worker の変更点](/2015/08/07/service-worker-release-notes-m45)
- [Chrome 44 の Service Worker の変更点](/2015/07/21/service-worker-release-notes-m44)
- [Chrome 43 の Service Worker の変更点](/2015/07/08/service-worker-release-notes-m43)

---

Chrome 46 の Service Worker 関係の変更は次のとおりです。

# 新機能

- Page や Worker に対するリクエスト ([non-subresource requests](https://fetch.spec.whatwg.org/#non-subresource-request)[^client-requests]) に対して CORS レスポンスが返せるようになりました。以前は CORS レスポンスを返した場合はネットワークエラーとなっていました ([bug](https://code.google.com/p/chromium/issues/detail?id=516972))。
- [Cache API] `Cache.addAll()`[^cache-addall] が実装されました ([feature](https://www.chromestatus.com/feature/4922023562182656))。
- [Fetch API] `Request.redirect` が実装されました ([feature](https://www.chromestatus.com/feature/4614142321229824))。
- [Performance API] Performance Timeline API が Worker (Dedicated Worker, Shared Worker, Service Worker) で利用できるようになりました ([feature](https://www.chromestatus.com/feature/6337483654561792))。
- [Performance API] `PerformanceResourceTiming.workerStart` が実装されました ([feature](https://www.chromestatus.com/feature/5767679470206976))。

# API の変更

- `Clients.matchAll()` が MRU order (most-recently-focused order) でクライアントを返すようになりました ([feature](https://www.chromestatus.com/feature/4716607557337088), [discussion](https://github.com/slightlyoff/ServiceWorker/issues/499))。
- `ServiceWorkerRegistration.update()`[^registration-update] が Promise を返すようになりました ([feature](https://www.chromestatus.com/feature/5631681746698240))。
- エスケープされた '/' や '\\' が含まれたスクリプト URL やスコープが `navigator.serviceWorker.register()` に指定された場合に `SecurityError` ではなく `TypeError` で reject するようになりました ([bug](https://code.google.com/p/chromium/issues/detail?id=513622), [discussion](https://github.com/slightlyoff/ServiceWorker/issues/630))。
- [Cache API] Cache API が secure origins (HTTPS もしくは localhost) でのみ使用できるように制限されました ([feature](https://www.chromestatus.com/feature/5740842165731328), [discussion](https://github.com/slightlyoff/ServiceWorker/issues/709))。
- [Fetch API] `Request.context` が削除されました ([feature](https://www.chromestatus.com/feature/5534702526005248), [discussion](https://github.com/whatwg/fetch/issues/93))。

# 改善点

- CORS リクエストの処理が最適化されました。CORS リクエストがネットワークにフォールバックし、かつそのリクエストの URL の Origin がページの Origin と同じ時、そのリクエストはより早く処理されるパスを通るようになりました ([bug](https://code.google.com/p/chromium/issues/detail?id=512764))。

# DevTools 関係の変更

Note: 最新の DevTools を試すために、[Dev channel もしくは Canary](https://www.chromium.org/getting-involved/dev-channel) の使用をおすすめします。

- CORS フォールバックしたレスポンスが DevTools の Network タブ上で "(Service Worker Fallback)" と表示されるようになりました。以前は "(cancelled)" と表示されていました。詳しくはバグトラッカーに添付されたスクリーンショットを参照してください ([bug](https://code.google.com/p/chromium/issues/detail?id=511054))。

# 補足

[^client-requests]: 以前は client requests と呼ばれていましたが、`Request.context` の削除に伴い non-subresource requests という用語になりました ([spec change](https://github.com/whatwg/fetch/commit/d2208faa939998cf56bb08a724cd8d4590afea47?diff=split))。client requests については [Chrome 44 のリリースノート](/2015/07/21/service-worker-release-notes-m44/#fn:client-request)を参照してください。
[^cache-addall]: `Cache.addAll()` については「[Chrome 46 に Cache.addAll() を実装した](/2015/09/02/cache-storage-addall/)」という記事を書きました。
[^registration-update]: `ServiceWorkerRegistration.update()` については「[Service Worker の update()](/2015/06/22/service-worker-update/)」という記事を書きました。
