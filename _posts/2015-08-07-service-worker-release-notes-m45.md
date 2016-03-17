---
layout: post
title: "Chrome 45 の Service Worker の変更点"
date: 2015-08-07 00:00:00
tags: serviceworker
---

本記事は Chrome 45 の Service Worker のリリースノートを日本語に意訳したものです。原文は service-worker-discuss グループで読むことができます。

- [[service-worker-discuss] Release Notes for Service Worker in Chrome 45](https://groups.google.com/a/chromium.org/forum/#!topic/service-worker-discuss/PNU3UhNoxU4)

以前のバージョンのリリースノートはこちら。

- [Chrome 44 の Service Worker の変更点](/2015/07/21/service-worker-release-notes-m44)
- [Chrome 43 の Service Worker の変更点](/2015/07/08/service-worker-release-notes-m43)


今回も草稿を [@FalkenMatto](https://twitter.com/FalkenMatto) さん (原文の投稿者) にレビューしてもらいました。ありがとうございます。

---

Chrome 45 の Service Worker 関係の変更は次のとおりです。

# 新機能

- `ServiceWorkerRegistration.update()`[^registration-update] が実装されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=450507), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#service-worker-registration-update))。
- `Client.id` が実装されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=504222), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#client-id))。
- 呼び出し元のオリジンに属する全ての ServiceWorkerRegistration を返す `navigator.serviceWorker.getRegistrations()` が実装されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=478382), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#navigator-service-worker-getRegistrations))。

# API の変更

- `Client.postMessage()` で送られたメッセージが `Window` ではなく `navigator.serviceWorker` にディスパッチされるようになりました ([Feature](https://www.chromestatus.com/feature/5163630974730240), [Demo](https://googlechrome.github.io/samples/service-worker/post-message/index.html))。
- "failing to activate" の概念が削除され、いったん `activate` イベントがディスパッチされた Service Worker は必ず active 状態になるようになりました。`waitUntil()` でアクティベーション処理を遅延させることができますが、これを `waitUntil(Promise.reject())` しても `waitUntil(Promise.resolve())` した場合と同じ挙動になります[^failing-to-activate] ([Bug](https://code.google.com/p/chromium/issues/detail?id=480050), [Spec Discussion](https://github.com/slightlyoff/ServiceWorker/issues/659))。
- `ServiceWorker.terminate()` が削除されました (以前は `terminate()` が呼ばれると `InvalidAccessError` 例外を投げていました) ([Bug](https://code.google.com/p/chromium/issues/detail?id=502934))。
- `WindowClient.frameType` が `Client.frameType` と重複していたため、削除されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=506736), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#window-client-interface))。
- `Clients.openWindow()` に不正な URL が渡された場合、`SyntaxError` ではなく `TypeError` で reject されるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=506071), [Spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#clients-openwindow))。
- `FetchEvent.cancellable` のデフォルト値が修正されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=501227))。
- (Push API) `PushSubscription.subscriptionid` が削除されました。今後は `PushSubscription.endpoint` を使用してください ([Feature](https://www.chromestatus.com/feature/5283829761703936), [Doc](https://developer.mozilla.org/en-US/docs/Web/API/PushSubscription/subscriptionId))。
- (Push API) `gcm_user_visible_only` が削除されました。今後は `userVisibleOnly` を使用してください ([Feature](https://www.chromestatus.com/feature/5778950739460096), [Doc](https://developer.mozilla.org/en-US/docs/Web/API/PushManager/subscribe))。

# 改善点

- Service Worker がナビゲーション時以外にも更新されるようになりました。具体的には Service Worker 起動時に前回の更新確認から 24 時間以上経っている場合に更新確認が行われます。これにより Service Worker を Push Notifications のためだけに使っている場合も適切に更新されるようになります ([Bug](https://code.google.com/p/chromium/issues/detail?id=477598))。
- Service Worker のスクリプトを格納しているバックエンドの実装が [BlockFile DiskCache](https://www.chromium.org/developers/design-documents/network-stack/disk-cache) から、より安定性の高い [SimpleCache](https://www.chromium.org/developers/design-documents/network-stack/disk-cache/very-simple-backend) に切り替わりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=487482))。
- (Chrome 44 以降) Service Worker のイベントが "[chrome://net-internals](https://sites.google.com/a/chromium.org/dev/for-testers/providing-network-details)" にロギングされるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=499143))。
- Service Worker の更新確認時、スクリプトに変更がなかった場合はディスクに余計な書き込みを行わないようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=457013))。

# DevTools 関係の変更

Note: 最新の DevTools を試すために、[Dev channel もしくは Canary](https://www.chromium.org/getting-involved/dev-channel) の使用をおすすめします。

- DevTools のコンソールのコンテキストの初期値が Service Worker からページに変更されました ([Bug](https://code.google.com/p/chromium/issues/detail?id=497721))。
- Service Worker を使ったアプリの開発効率向上を目指し、現在 "Service Worker explorer UI" の開発に取り組んでいます。この機能はまだ experimental で、現在 UX の改良が行われています。こちらの[スライド](https://docs.google.com/presentation/d/1DKu4RZigLvM5XUq3ovsgffQBIHrro5-pii4qEJuyvrQ/edit?usp=sharing)を参考に試用していただき、是非フィードバックをお寄せください。

# バグフィックス

- Service Worker が `iframe[sandbox]` を尊重するようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=486308))。
- Service Worker のアイドルタイマーが他のイベントと同じようにメッセージイベントでもリセットされるようになりました ([Bug](https://code.google.com/p/chromium/issues/detail?id=498121))。
- バグフィックスの全リストは[こちら](https://code.google.com/p/chromium/issues/list?can=1&q=Cr%3DBlink-ServiceWorker+m%3D45+status%3AFixed%2CVerified&colspec=ID+Pri+M+ReleaseBlock+Cr+Status+Owner+Summary+OS+Modified&x=m&y=releaseblock&cells=tiles)で確認できます。

---

# 訳者補足

[^registration-update]: `update()` については「[Service Worker の update()](/2015/06/22/service-worker-update)」で解説を書きました。
[^failing-to-activate]: "failing to activate" の挙動は「[Service Worker の Registration](/2015/07/05/service-worker-registration)」でも紹介しました。
