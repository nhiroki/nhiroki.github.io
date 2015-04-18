---
layout: post
title: "Service Worker の claim() について"
date: 2015-04-18 00:00:00
category: programming
---

Service Worker のスコープとページコントロールについて解説する記事のその 2 です。既に Service Worker の基本について理解していて、かつ、[前回の記事](/2015/02/28/service-worker-scope-and-page-control/)を読んでいることを前提にしています。今回は「まだコントロールされていないページをコントロール状態にする claim()」について紹介します。claim() は Chrome ではバージョン 42 から使用することができます ([リリースノート](https://groups.google.com/a/chromium.org/forum/#!topic/service-worker-discuss/c6qFwC79Q1A))。

#ページコントロールが始まるタイミング (前回の復習)#

前回の記事で「Service Worker がページをコントロールするかどうかは、そのページを開いた時に判断される」と説明しました。次のコードは前回の記事からのコピーで、ページを開いた後に登録された Service Worker が、そのページを直ちに (初回ロード時) にコントロールすることはないことを示しています。

{% highlight js %}
// /scope/will-be-controlled.html
navigator.serviceWorker.register('sw.js', {scope: '/scope/'})
  .then(function(registration) {
      // 登録成功！
      return navigator.serviceWorker.ready;
    })
  .then(function(registration) {
      // アクティベートされたが、この時点では
      // このページ '/scope/will-be-controlled.html' はコントロールされていない。
      assert_true(navigator.serviceWorker.controller == null);
    });
{% endhighlight %}

ちなみに二回目以降のロードでは既に Service Worker が登録されているため、controller は non-null になります。よって、開発中に初回ロード時の挙動をテストするには DevTools などから登録情報を削除する必要があります。また、同じ Service Worker スクリプトとスコープに対して register() を複数回呼んだ場合、登録済みの registration が返ってきます。registration についてはそのうち別の記事を書くかもしれません。

#まだコントロールされていないページをコントロール状態にする#

上記のコードのままだと初回ロード時は Service Worker によってコントロールされません。これを解決するにはページロード時以外にページコントロールを開始させる方法があると良さそうです。それが claim() です。

claim() は Service Worker 側のスクリプトで使います。Service Worker の実行コンテキスト [ServiceWorkerGlobalScope](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-global-scope) は clients というフィールドを持っています。これは現在ブラウザで開いているページのうち、この Service Worker と同じオリジンに属するページ一覧のコンテナとなっています。例えば、次のように使います。

{% highlight js %}
// sw.js
var promise = self.clients.matchAll({includeUncontrolled: true})
  .then(function(clients) {
      // clients は現在開いているページ (client) を保持する。
      // ページの visibilityState を取得したり、postMessage で通信したりできる。
    });
{% endhighlight %}

この clients に claim() は生えているのですが、ここで一旦 claim() は横に置いておき、Client についてもう少し説明します。

Service Worker の仕様では [Service Worker Client](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-client-concept) というモデルを定義しています。これは Service Worker によってコントロールされうる実行コンテキストへのインタフェースとなっていて、Window コンテキストの場合は WindowClient、Worker コンテキスト (SharedWorker など) の場合は WorkerClient というものに関連付けられています。そしてこれらをまとめて [Client](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#client-interface) と呼びます。本記事では以後、コントロール対象のページをクライアントと呼ぶことにします。

さて、話を claim() に戻します。clients.claim() は同一オリジン内のクライアントに対してこの Service Worker がコントローラーになることを要求します。claim() を使用したコードは次のようになります。

{% highlight js %}
// sw.js
self.addEventListener('activate', function(event) {
    event.waitUntil(self.clients.claim());
  });
{% endhighlight %}

claim() は activate された Service Worker 上で呼ぶ必要があります。さもなければ InvalidStateError が返ってきます。ここでは activate イベント内で呼ぶことでそれを保証しています。waitUntil() は引数に渡された promise が resolve されるまでイベントのライフタイムを延長します。これにより activate イベント終了時に claim() の実行が終わっていることを保証します。

clients.claim() は各クライアントに対して、呼び出し元の Service Worker でコントロールできるかどうかを判定するわけですが、その判定条件は[前回の記事](/2015/02/28/service-worker-scope-and-page-control/)で紹介したとおりです。大雑把に言うと次のようになります。

 - クライアントの URL がこの Service Worker のスコープ内に含まれているか
 - クライアントの URL が最長一致のルールに合致しているか

以上の条件に合致したクライアントはこの Service Worker によってコントロールされ始めます。

#コントロールの開始#

Service Worker の activate イベント内で claim() の終了を待ってあげれば、最初のクライアント側のコードは特に変更せずにそのまま使えます。

{% highlight js %}
// /scope/will-be-controlled.html
navigator.serviceWorker.register('sw.js', {scope: '/scope/'})
  .then(function(registration) {
      // activate イベント (claim) の終了を待つ
      return navigator.serviceWorker.ready;
    })
  .then(function() {
      // このクライアント '/scope/will-be-controlled.html' は
      // 初回ロード時からコントロールされる。
      assert_true(navigator.serviceWorker.controller);
    });
{% endhighlight %}

ところで、新たにコントロールされることになったクライアントには controllerchange イベントが発火します。このイベントはロード時からコントロールされている場合は発火しないので注意が必要です。もし ready の代わりに使う場合は navigator.serviceWorker.controller と一緒に Promise.race で待つと良いかもしれないです。

{% highlight js %}
// /scope/will-be-controlled.html
navigator.serviceWorker.register('sw.js', {scope: '/scope/'})
  .then(function(registration) {
      var controller_change_promise = new Promise(function(resolve) {
        navigator.serviceWorker.addEventListener('controllerchange', resolve);
      });
      return Promise.race([navigator.serviceWorker.controller,
                           controller_change_promise]);
    })
  .then(function() {
      // このクライアント '/scope/will-be-controlled.html' は
      // 初回ロード時からコントロールされる。
      assert_true(navigator.serviceWorker.controller);
    });
{% endhighlight %}

#まとめ#

 - Service Worker のコントロール対象をクライアントと呼ぶ
 - Service Worker がクライアントをコントロールするかどうかは、基本的にはそのクライアントをロードした時に判断される
 - claim() を使うとまだコントロールされていないクライアントをコントロール状態にできる
 - claim() は初回ロード時からコントロールさせたい場合に有用

次回は claim() と紛らわしいと評判の skipWaiting() について紹介する予定です。
