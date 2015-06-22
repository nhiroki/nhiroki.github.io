---
layout: post
title: "Service Worker の update()"
date: 2015-06-22 00:00:00
tags: serviceworker
---

Chromium に ServiceWorkerRegistration.update() を実装したのでその紹介。順調に行けばバージョン 45 から使用することができます (Canary ではもう使えます)。

- [[spec] Service Workers - update()](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-registration-update)
- [[blink-dev] Intent-to-Ship](https://groups.google.com/a/chromium.org/forum/#!topic/Blink-dev/bvi8fXqvNhs)
- [Chromium Issue](https://code.google.com/p/chromium/issues/detail?id=450507)

Service Worker は適当なタイミングでスクリプトのアップデートチェックが走ります。具体的には Service Worker のコントロール下にあるリソースに対してリクエストを発行したときです。このアップデートチェックはスクリプトの Cache-Control ヘッダに従うため、もし頻繁にスクリプトを更新しないのであれば max-age を指定してあげることで不要なチェックを省くことができます。

一方で、もし何らかの理由で max-age に長大な時間が設定された場合は、スクリプトがなかなか更新されないことになります。このようなスクリプトの焼付けを防ぐために、Service Worker では最後の更新から 24 時間以上が経った場合はキャッシュの有無に関わらず必ずアップデートチェックを実行します ([Update Algorithm](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#update-algorithm ) 参照)。

update() はこの「キャッシュを無視するアップデートチェック」をスクリプトから行えるようにしたものです ([Note](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-registration-update) 参照)。例えば次のように使います。

{% highlight js %}
navigator.serviceWorker.getRegistration()
  .then(function(registration) { registration.update(); });
{% endhighlight %}

更新があった場合は updatefound イベントが発火します。

{% highlight js %}
registration.addEventListener('updatefound', function() {
  // A new worker is coming!
  console.assert(registration.installing);
});
{% endhighlight %}

Service Worker の実行コンテキスト上でも使えます。

{% highlight js %}
// sw.js
self.registration.update();
{% endhighlight %}

ちなみに update() を呼んだからといってすぐに [コントローラー](http://qiita.com/nhiroki/items/eb16b802101153352bba#serviceworker-%E3%81%AB%E3%82%88%E3%82%8B%E3%83%9A%E3%83%BC%E3%82%B8%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%AB%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) である Service Worker が入れ替わるわけではありません。現在のコントローラーによってコントロールされているページが全て閉じられてから入れ替わります ([Service Worker ハッカソンのスライド](https://docs.google.com/presentation/d/1WiL241gQYOSAV6yVlM2_hloX-fDwzWHIZXqWhuEzdX0/pub?start=false&loop=false&delayms=3000&slide=id.g900657643_0_59) 参照)。コントロールされているページがあったとしても強制的にコントローラーを入れ替える [skipWaiting()](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-global-scope-skipwaiting-method) という API もあります。
