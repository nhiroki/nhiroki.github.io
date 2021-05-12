---
layout: post
title: "Service Worker の Registration"
date: 2015-07-05 00:00:00
tags: web
---

今回は Service Worker の中で最も重要なオブジェクトであろう ServiceWorkerRegistration について詳しく紹介します。なお、この記事は 2015/07/05 時点での仕様に基づいており、Chrome 45 で動作確認を行っています。

ServiceWorkerRegistration はその名の通り Service Worker の登録情報を表しています。[スコープ](/2015/02/28/service-worker-scope-and-page-control/)によって一意に特定することができ、そのスコープに関連した [ServiceWorker オブジェクト](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-obj)のセットを持っています。この辺りのことはコンセプトとして[仕様に明記](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#service-worker-registration-concept)されています。

ServiceWorkerRegistration オブジェクトの IDL は次のように定義されています。

```
[Exposed=(Window,Worker)]
interface ServiceWorkerRegistration : EventTarget {
  [Unforgeable, SameObject] readonly attribute ServiceWorker? installing;
  [Unforgeable, SameObject] readonly attribute ServiceWorker? waiting;
  [Unforgeable, SameObject] readonly attribute ServiceWorker? active;

  readonly attribute USVString scope;

  void update();
  [NewObject] Promise<boolean> unregister();

  attribute EventHandler onupdatefound;
};
```

ServiceWorker オブジェクトについても言及するので、そちらの IDL も引用しておきます。

```
[Exposed=(Window,Worker)]
interface ServiceWorker : EventTarget {
  readonly attribute USVString scriptURL;
  readonly attribute ServiceWorkerState state;
  readonly attribute DOMString id;
  void postMessage(any message, optional sequence<Transferable> transfer);

  attribute EventHandler onstatechange;
};
ServiceWorker implements AbstractWorker;

enum ServiceWorkerState {
  "installing",
  "installed",
  "activating",
  "activated",
  "redundant"
};
```

本記事では、一般的な意味での Service Worker を SW、ServiceWorkerRegistration インタフェースを Registration と表記し、ServiceWorker インタフェースについては省略せずに表記します。

# Registration の登録・取得・更新・抹消

Registration の登録には register() を使用します。次の例では "https://example.com/foo/bar" をキーとした Registration を登録します。

```js
// On https://example.com/index.html
navigator.serviceWorker.register('/sw.js', {scope: '/foo/bar'})
  .then(function(registration) { ... });
```

登録済みの Registration は getRegistration() もしくは getRegistrations() で取得することができます。getRegistrations() は Chrome ではバージョン 45 から使用することができます ([crbug](http://crbug.com/478382))。

```js
// 現在のページをスコープに含む Registration を返す
navigator.getRegistration().then(function(registration) { ... });
// 'https://example.com/hoge/fuga' をスコープに含む Registration を返す
navigator.getRegistration('/hoge/fuga/').then(function(registration) { ... });
// このオリジンに属する全ての Registration を返す
navigator.getRegistrations().then(function(registrations) { ... });
```

.ready は現在のページをスコープに含む Registration の .active がセットされた時に resolve される promise です ([別記事参照](http://qiita.com/nhiroki/items/eb16b802101153352bba#%E3%82%B3%E3%83%B3%E3%83%88%E3%83%AD%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B%E3%82%BF%E3%82%A4%E3%83%9F%E3%83%B3%E3%82%B0))。

```js
navigator.serviceWorker.ready.then(function(registration) {
    console.assert(registration.active);
  });
```

SW の実行コンテキスト上では自身が所属している Registration を self.registration で取得することができます。

```js
// On https://example.com/sw.js
var registration = self.registration;
```

Registration に紐付いた SW を更新するには update() を使います ([別記事参照](/2015/06/22/service-worker-update))。

```js
registration.update();
```

Registration の抹消には unregister() を使います。

```js
// 成功すると true を返す。既に抹消済みの場合は false を返す
registration.unregister().then(function(result) { ... });
```

# Registration と ServiceWorker の状態遷移

Registration には installing, waiting, active の三つの ServiceWorker オブジェクトが関連付けられています。

```
interface ServiceWorkerRegistration : EventTarget {
  [Unforgeable, SameObject] readonly attribute ServiceWorker? installing;
  [Unforgeable, SameObject] readonly attribute ServiceWorker? waiting;
  [Unforgeable, SameObject] readonly attribute ServiceWorker? active;
  // 以下省略
};
```

次の図は ServiceWorker オブジェクトの状態 ([ServiceWorker.state](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-state)) の遷移と Registration のフィールドの対応関係を表しています。

[![ServiceWorkerRegistration](/images/service-worker-registration.png)](/images/service-worker-registration.png)

.installing は register() もしくは update() から install イベント完了までの installing 状態の ServiceWorker オブジェクトです。新しい SW が .installing にセットされると Registration の updatefound イベントが発火します ([別記事参照](/2015/06/22/service-worker-update))。これを使うと、新しい SW が検出されたらとりあえず postMessage を送る、といったことができます。

```js
registration.addEventListener('updatefound', function(e) {
    e.currentTarget.installing.postMessage('Hello, new installing worker!');
  });
```

.waiting は install イベント完了から activate イベント開始までの installed 状態の ServiceWorker オブジェクトです。三つのフィールドの中で .waiting が一番分かりにくいかもしれません。register() や update() などによって新しい SW がインストールされたとしても、その時点でページをコントロールしている SW がいる場合はすぐに入れ替えることができません。というのも、ページコントロール中に SW が入れ替わってしまうとアプリケーションの処理に不整合が生じる可能性があるからです。そこで、現在アクティブな SW がコントロールしているページがすべて閉じられて安全に SW を入れ替えできるようになるまでは、新しくインストールされた SW は installed 状態で待機することになります。これが waiting と呼ばれる理由です。

ちなみにコントロールされているページの有無に関わらず、一気に .active (activated 状態) に遷移させる方法として [skipWaiting()](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/index.html#service-worker-global-scope-skipwaiting) があります。これを使うと、例えば「バグのある SW スクリプトを配信してしまったのですぐに SW を更新したいけど、ユーザがタブを開きっぱなしにしているせいで SW がいつまでも入れ替わらない」といった事態を避けることができます。

```js
// On https://example.com/sw.js
self.addEventListener('install', function(event) {
    // install イベント終了後、コントロールされているページの有無に関わらず、
    // すぐに activate イベントが発火する
    event.waitUntil(skipWaiting());
  });
```

.active は activate イベント開始以降の activating もしくは activated 状態の ServiceWorker オブジェクトです。この状態になるとページをコントロールすることができます。ちなみに activating 状態の SW は activate イベントが reject されたとしても activated 状態に遷移し、コントローラーになるので注意が必要です ([Spec Issue](https://github.com/slightlyoff/ServiceWorker/issues/659#issuecomment-95244473))。

```js
// On https://example.com/sw.js
self.addEventListener('activate', function(event) {
    // SW は activated 状態になる
    event.waitUntil(Promise.resolve());
  });

self.addEventListener('activate', function(event) {
    // reject されても SW は activated 状態になる (resolve の場合と等価)
    event.waitUntil(Promise.reject());
  });
```

# Registration の状態確認

Registration の状態は Chrome の場合は DevTools 上で確認することができます。ただし、この機能は本記事公開の時点では experimental な扱いのため、デフォルトでは使用することができません。利用方法については次のスライドを参考にしてください。

[Debugging ServiceWorker (horo@) - Google スライド](https://docs.google.com/presentation/d/1DKu4RZigLvM5XUq3ovsgffQBIHrro5-pii4qEJuyvrQ/edit?usp=sharing)

# まとめ

今回は ServiceWorkerRegistration の概要とそれに関係した API、そして ServiceWorker オブジェクトの状態遷移との対応関係について説明しました。navigator.serviceWorker.register() や getRegistration() が SW のエントリーポイントになるのに対し、Registration は SW の状態を表す中心的なオブジェクトになります。これらを抑えておくことで、SW のモデルをより深く理解できるようになるはずです。
