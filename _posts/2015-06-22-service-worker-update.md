---
layout: post
title: "Service Worker の update()"
date: 2015-06-22 00:00:00
tags: web
---

- **2018/02/15 : 仕様や実装の変更に伴い新たに「[Service Worker スクリプトのインストールと更新処理](/2018/02/15/service-worker-install-and-update-scripts)」という記事を公開しました。本記事の内容は既に古いので、新しい記事を参照するようにしてください。**
- **2015/12/13 : update() とブラウザキャッシュのインタラクションについて仕様変更があったので追記しました。Chrome 48 から適用されます。**
- **2015/12/13 : update() が Promise を返すように仕様変更があったので追記しました。Chrome 46 から適用されます。**

---

Chrome に ServiceWorkerRegistration.update() を実装したのでその紹介。Chrome 45 から使用することができます。

- [[spec] Service Workers - update()](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-registration-update)
- [[blink-dev] Intent-to-Ship](https://groups.google.com/a/chromium.org/forum/#!topic/Blink-dev/bvi8fXqvNhs)
- [Chromium Dashboard - ServiceWorkerRegistration.update()](https://www.chromestatus.com/feature/5663070173003776)

Service Worker は適当なタイミングでスクリプトのアップデートチェックが走ります。例えば Service Worker のコントロール下にあるページを開いたときにチェックします ([Handle Fetch Algorithm](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#on-fetch-request-algorithm) 参照)。このアップデートチェックはスクリプトの Cache-Control ヘッダに従うため、もし頻繁にスクリプトを更新しないのであれば max-age を指定してあげることで不要なチェックを省くことができます。

一方で、もし何らかの理由で max-age に長大な時間が設定された場合は、スクリプトがなかなか更新されないことになります。このようなスクリプトの焼付けを防ぐために、Service Worker では最後の更新から 24 時間以上が経った場合はキャッシュの有無に関わらず必ずアップデートチェックを実行します ([Update Algorithm](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#update-algorithm ) 参照)。

update() はこの「<del>キャッシュを無視する</del>アップデートチェック」をスクリプトから行えるようにしたものです (**2015/12/13 追記: Chrome 48 から update() もスクリプトの Cache-Control ヘッダに従うようになりました。前回更新から 24 時間以上経った場合は今まで通りキャッシュを無視してチェックを行います ([Dashboard](https://www.chromestatus.com/feature/5897293530136576))**)。

例えば次のように使います (2015/12/13 追記: Chrome 46 から update() が Promise を返すようになりました ([Dashboard](https://www.chromestatus.com/feature/5631681746698240)))。

```js
navigator.serviceWorker.getRegistration()
  .then(function(registration) { return registration.update(); })
  .then(function() {
      // The script is updated, or there is no updated script.
    })
  .catch(function(e) {
      // An error occurs during update (eg. Network error, Runtime error).
    });
```

更新があった場合は updatefound イベントが発火します。

```js
registration.addEventListener('updatefound', function() {
  // A new worker is coming!
  console.assert(registration.installing);
});
```

Service Worker の実行コンテキスト上でも使えます。

```js
// sw.js
self.registration.update();
```

ちなみに update() を呼んだからといってすぐに [コントローラー](http://qiita.com/nhiroki/items/eb16b802101153352bba#serviceworker-%E3%81%AB%E3%82%88%E3%82%8B%E3%83%9A%E3%83%BC%E3%82%B8%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%AB%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) である Service Worker が入れ替わるわけではありません。現在のコントローラーによってコントロールされているページが全て閉じられてから入れ替わります ([Service Worker ハッカソンのスライド](https://docs.google.com/presentation/d/1WiL241gQYOSAV6yVlM2_hloX-fDwzWHIZXqWhuEzdX0/pub?start=false&loop=false&delayms=3000&slide=id.g900657643_0_59) 参照)。コントロールされているページがあったとしても強制的にコントローラーを入れ替える [skipWaiting()](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-global-scope-skipwaiting-method) という API もあります。
