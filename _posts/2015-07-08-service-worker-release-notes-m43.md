---
layout: post
title: "Chrome 43 の Service Worker の変更点"
date: 2015-07-08 00:00:00
tags: serviceworker
---

本記事は Chrome 43 の Service Worker のリリースノートを日本語に意訳したものです。原文は service-worker-discuss グループで読むことができます。

- [[service-worker-discuss] Release Notes for Service Worker in Chrome 43](https://groups.google.com/a/chromium.org/forum/#!topic/service-worker-discuss/hHkLjDpwWiU)

草稿を [@FalkenMatto](https://twitter.com/FalkenMatto) さん (原文の投稿者) にレビューしてもらいました。ありがとうございます。

---

リリースノートを復活させます！2015 年 5 月後半に stable リリースされた Chrome 43 の Service Worker 関係の変更は次の通りです。なお、Chrome 44 は今月後半のリリースが予定されており、またリリースノートを公開する予定です。

#新機能#

- `Clients.matchAll()` をサポートしました。オプショナル引数によってマッチするクライアント[^client]の種類を指定することができ、Shared Worker のような non-Window クライアントにも対応しました ([Bug](https://code.google.com/p/chromium/issues/detail?id=460415), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#clients-getall))。
- `Clients.openWindow()` で cross-origin URL を開くことができるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=457187), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#clients-openwindow))。
- CacheStorage API が Window や Service Worker 以外の Worker 上でも使えるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=439389), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#cache-storage))。注意: 現在 HTTP 上でも使用可能ですが、HTTPS 上でのみ使えるように制限がかけられる可能性があります ([Spec Discussion](https://github.com/slightlyoff/ServiceWorker/issues/709))。

#API の変更#

- `Clients.getAll()` が削除されました。代わりに `Clients.matchAll()` を使用してください ([Bug](https://code.google.com/p/chromium/issues/detail?id=451334), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#clients-getall))。

#改善#

- データベースアクセスの排除により、`Clients.claim()` がより高速に処理されるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=454250))。

#DevTools 関係の変更#

Note: 最新の DevTools を試すために、[Dev channel もしくは Canary](https://www.chromium.org/getting-involved/dev-channel) の使用をおすすめします。

- UA のオーバライドやネットワーク帯域の制限などが行えるデバイスエミュレーション機能が Service Worker と一緒に使えるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=444820))。
- ドキュメントとそれをコントロールする Service Worker を同じ DevTools ウィンドウ上で操作できるようになりました。コンソールのドロップダウンリストからドキュメントと Service Worker のコンテキストを切り替えることができます。
- DevTools の Sources タブの左側のペインに Service Worker のエントリが表示されるようになりました。
- ドキュメントのコンソールに Service Worker から出力したメッセージが表示されるようになりました。
- Service Worker を使ったアプリの開発効率向上を目指し、現在 "Service Worker explorer UI" の開発に取り組んでいます。この機能はまだ experimental で、現在 UX の改良が行われています。こちらの[スライド](https://docs.google.com/presentation/d/1DKu4RZigLvM5XUq3ovsgffQBIHrro5-pii4qEJuyvrQ/edit?usp=sharing)を参考に試用していただき、是非フィードバックをお寄せください。

#バグフィックス#

- `navigator.serviceWorker.register()` に渡されたスクリプト URL とスコープは ServiceWorkerContainer のあるドキュメントの URL を基準にパス解決されていましたが、呼び出し元ドキュメントの URL を基準にパス解決されるように修正されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=449422))。
- `navigator.serviceWorker.ready` が shift-reload で開かれたページでも適切に resolve されるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=462529))。
- 同一の Service Worker を指す複数の Javascript オブジェクト (ServiceWorker オブジェクト) の状態が適切に同期されていなかった問題が修正されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=459457))。
- `navigator.serviceWorker.ready` はページロード時のスコープマッチ[^scope-match]で選ばれた ServiceWorkerRegistration の `.active` で resolve されていましたが、`.ready` にアクセスした時にスコープマッチしたもので resolve されるように修正されました。これにより、ページロード後に作られた ServiceWorkerRegistration のスコープの方がより長く一致する場合はそちらで resolve されるようになります。
- バグフィックスの全リストは[こちら](https://code.google.com/p/chromium/issues/list?can=1&q=Cr%3DBlink-ServiceWorker+m%3D43&colspec=ID+Pri+M+Stars+ReleaseBlock+Cr+Status+Owner+Summary+OS+Modified&cells=tiles)で確認できます。

---

#訳者補足#

[^client]: クライアントについては[こちらの記事](/2015/04/18/service-worker-claim/)で解説を書きました。
[^scope-match]: ServiceWorkerRegistration のスコープマッチについては[こちらの記事](/2015/02/28/service-worker-scope-and-page-control/)で解説を書きました。
