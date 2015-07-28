---
layout: post
title: "Chrome 44 の Service Worker の変更点"
date: 2015-07-21 00:00:00
tags: serviceworker
---

本記事は Chrome 44 の Service Worker のリリースノートを日本語に意訳したものです。原文は service-worker-discuss グループで読むことができます。

- [[service-worker-discuss] Release Notes for Service Worker in Chrome 44](https://groups.google.com/a/chromium.org/forum/#!topic/service-worker-discuss/C9LHhAcz7mw)

以前のバージョンのリリースノートはこちら。

- [Chrome 43 の Service Worker の変更点](/2015/07/08/service-worker-release-notes-m43/)

(2015/07/28 追記) [@FalkenMatto](https://twitter.com/FalkenMatto) さん (原文の投稿者) からコメントをいただき、一部加筆修正しました。ありがとうございます。

---

Chrome 44 の Service Worker 関係の変更は次のとおりです。

#新機能#

- `Request.context` がサポートされました ([Bug](https://code.google.com/p/chromium/issues/detail?id=455116), [Spec](https://fetch.spec.whatwg.org/#dom-request-context))。
- `Cache.add()` が実装されました ([Dashboard](https://www.chromestatus.com/feature/5673980799221760), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#cache-add))。

#API の変更#

- `FetchEvent` が `ExtendableEvent`[^extendable-event] を継承するようになり、コンストラクタが追加されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=479536), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#fetch-event-section))。
- セキュリティ上の理由に、client requests[^client-request] に対して opaque レスポンスを返すことができなくなりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=474914), [Spec Discussion](https://github.com/slightlyoff/ServiceWorker/issues/590))。
- install イベントのインタフェースが `InstallEvent` から `ExtendableEvent` に変更され、`InstallEvent` インタフェースが削除されました ([Bug](http://code.google.com/p/chromium/issues/detail?id=470032), [Spec Discussion](https://github.com/slightlyoff/ServiceWorker/issues/661))。

#改善点#

- ブラウザのコンテンツ設定で Service Worker を許可しているかどうか、"Cookies set by this page" UI[^cookies-ui] 上で確認できるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=419284))。
- Service Worker がスタートアップに失敗するバグが修正されました。また、スタートアップ失敗によってメインリソースリクエストに対する `FetchEvent` がディスパッチできなかった場合は適切に[^network-fallback]ネットワークにフォールバックするように改善されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=448003))。

#DevTools 関係の変更#

Note: 最新の DevTools を試すために、[Dev channel もしくは Canary](https://www.chromium.org/getting-involved/dev-channel) の使用をおすすめします。

- `--unsafely-treat-insecure-origin-as-secure` コマンドラインオプションが追加されました。Service Worker のような [Powerful Features](https://www.google.com/url?q=https%3A%2F%2Fw3c.github.io%2Fwebappsec%2Fspecs%2Fpowerfulfeatures%2F&sa=D&sntz=1&usg=AFQjCNG2JU3mZRk1D6C5Nh2qlu5OXWBVLw) は Secure Contexts (例: HTTPS) を必要としますが、このオプションで指定されたオリジンではセキュリティチェックをバイパスします。これによりローカル環境での開発が行いやすくなりました。

{% highlight sh %}
$ ./chrome --user-data-dir=/tmp/foo --unsafely-treat-insecure-origin-as-secure=http://your.insecure.site
{% endhighlight %}

- Cache Storage のエントリを個別に消せるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=474455))。
- Cache Storage が Service Worker のフレームだけでなく、すべてのフレームから inspect できるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=439389))。
- non-incognito ウィンドウ上の Service Worker でコンソール出力した内容が incognito ウィンドウのコンソールに表示されてしまうバグが修正されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=488241))。
- `waitUntil()` や `respondWith()` で発生した例外が Service Worker のコンソールに出力されるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=359423))。
- Service Worker を使ったアプリの開発効率向上を目指し、現在 "Service Worker explorer UI" の開発に取り組んでいます。この機能はまだ experimental で、現在 UX の改良が行われています。こちらの[スライド](https://docs.google.com/presentation/d/1DKu4RZigLvM5XUq3ovsgffQBIHrro5-pii4qEJuyvrQ/edit?usp=sharing)を参考に試用していただき、是非フィードバックをお寄せください。

#バグフィックス#

- Service Worker が STOPPING 状態でスタックしてしまうバグが修正されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=499646))。
- バグフィックスの全リストは[こちら](https://code.google.com/p/chromium/issues/list?can=1&q=Cr%3DBlink-ServiceWorker+m%3D44+status%3AFixed%2CVerified&colspec=ID+Pri+M+ReleaseBlock+Cr+Status+Owner+Summary+OS+Modified&x=m&y=releaseblock&cells=tiles)で確認できます。

---

#訳者補足#

[^extendable-event]: `ExtendableEvent` はそれの持つ `waitUntil()` に任意の Promise を渡すことで、その Promise が settled (resolved or rejected) になるまでイベントのライフタイムを延長することができるイベントです。
[^client-request]: client requests とは `Request.context` が navigation 関係 (form, frame, hyperlink etc) もしくは worker (serviceworker, sharedworker, worker) であるようなものを指します。詳しくは [Fetch の仕様](https://fetch.spec.whatwg.org/#requests)を参照してください。
[^cookies-ui]: URL バーの鍵もしくはファイルアイコンをクリックした時に表示される UI です。
[^network-fallback]: (2015/07/28 追記) メインリソースリクエストがネットワークにフォールバックした場合は、その後のサブリソースリクエストもすべて Service Worker を介さずにネットワークにフォールバックします。サブリソースリクエスト時に `FetchEvent` がディスパッチできなかった場合はネットワークにフォールバックせず、エラーレスポンス (500 "Service Worker Response Error") を返します。これにより、同一ドキュメント内で Service Worker にインターセプトされるリクエストとされないリクエストが混在することはありません。
