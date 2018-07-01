---
layout: post
title:  "Chrome 46 に Cache.addAll() を実装した"
date:   2015-09-02 00:00:00
tags: web
---

Chrome 46 に CacheStorage API の Cache.addAll() を実装したのでその紹介です。

 - [MDN - Cache.addAll()](https://developer.mozilla.org/en-US/docs/Web/API/Cache/addAll)
 - [Spec - Cache.addAll()](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#cache)
 - [Chromium Dashboard - Cache.addAll()](https://www.chromestatus.com/feature/4922023562182656)
 - (2015/09/09 追記) [Google Web Updates - Updates to the service worker cache API](https://developers.google.com/web/updates/2015/09/updates-to-cache-api)

Cache.addAll() は Request もしくは URL String の配列を受け取り、それらを fetch した結果を Cache に保存する API です。

```js
caches.open('cachename')
  .then(function(cache) {
      return cache.addAll([
        '/foo.png',             // URL
        new Request('/bar.js')  // Request
      ]);
    })
  .then(function() { /* 全てのレスポンスが保存されると resolve されます */ })
  .catch(e) { /* エラーがあると reject されます */ }
```

機能的には複数の cache.add() を Promise.all() で待ち受けた場合と等価です。

Service Worker Hackathon などでは未実装 API のために [coonsta/cache-polyfill](https://github.com/coonsta/cache-polyfill) の使用を[推奨](https://docs.google.com/presentation/d/1WiL241gQYOSAV6yVlM2_hloX-fDwzWHIZXqWhuEzdX0/pub?start=false&loop=false&delayms=3000&slide=id.g917300647_0_126)していましたが、今回の変更によってこのポリフィルが提供していたすべての API (Cache.add() と Cache.addAll()) が実装されたので、Chrome 46 以降では使用する必要はありません。

ちなみに Chrome 46 ではまだ [Cache.matchAll()](https://developer.mozilla.org/en-US/docs/Web/API/Cache/matchAll) と [CacheQueryOptions](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#cache-query-options-dictionary) を使用できないので注意してください。Cache.matchAll() は既に実装されていますが、使用するにはフラグを有効にする必要があります ([近々デフォルトで有効化される予定です](https://code.google.com/p/chromium/issues/detail?id=523916))。CacheQueryOptions はまだ実装されていません。もしオプションが必要なユースケースがある場合は、是非[バグトラッカー](https://code.google.com/p/chromium/issues/detail?id=426309)にコメントしてください。
