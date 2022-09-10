---
layout: post
title: "Service Worker の FetchEvent がネットワークフォールバックした場合の仕様を読み解く"
date: 2022-09-10 00:00:00 +09:00
tags: web
image: /images/profile.png
---

Service Worker はブラウザの発行した HTTP リクエストをフックする FetchEvent を提供している。FetchEvent の持つ `respondWith()` に対して Response オブジェクトで resolve される promise を渡すことで、任意のレスポンスを返すことができる。例えば、リクエスト URL とは異なる URL に対してフェッチした結果を返したり、Cache Storage API にキャッシュしてあったレスポンスを返したりできる。

```js
onfetch = e => {
  e.responseWith(fetch("https://another.example.com"));
}
```

また FetchEvent で `respondWith()` を呼ばずに return した場合は、そのリクエストは通常のネットワークリクエストとして処理される。これをネットワークフォールバックと呼んでいる。

```js
onfetch = e => {
  // respondWith() is not called. This results in a regular network request.
}
```

FetchEvent は Service Worker の機能なので、その挙動は [Service Worker 仕様](https://w3c.github.io/ServiceWorker/)で定義されている。一方フェッチ処理は [Fetch 仕様](https://fetch.spec.whatwg.org/)にて定義されている。

とあるイシューの対応でこれら仕様がどのように連携してネットワークフォールバックを行うのか調査する必要があり、本記事はそれをメモしたものです。本記事で引用した仕様文面は 2022 年 9 月 10 日現在のものであり、最新のものとは異なる可能性があります。

# Fetch Event

Fetch Event の処理は Service Worker 仕様の [Handle Fetch](https://w3c.github.io/ServiceWorker/#handle-fetch) アルゴリズムで定義されている。Service Worker でコントロールされたページで何かしらのフェッチリクエストが発生すると、その過程でこのアルゴリズムが呼ばれる。

まず大事なのが step 4 で `response` 変数を初期化しているところ。正常系においてはこれがこのアルゴリズムの返り値となる。返り値は、正常な response、network error response, null のどれかになる。ちなみにネットワークフォールバック時は null になる。


```
4. Let response be null.
```

Step 23 までは FetchEvent をディスパッチするための前処理で、step 24.3 の中で実際に FetchEvent がディスパッチされる。

```
24.3
  1. Let e be the result of creating an event with FetchEvent.
  (skip)
  12. Dispatch e at activeWorker’s global object.
```

Step 24.3.14 と 24.3.15 で FetchEvent の respond-with entered flag と wait to respond flag がセットされているかを確認している。これらフラグにより FetchEvent 内で `respondWith()` が呼ばれたかが分かる。呼ばれた場合は step 24.3.15.3 で `response` 変数に対して potential response をセットする。この potential response は `respondWith()` に渡した promise が resolve されたタイミングで中身が確定する。


```
24.3
  14. If e’s respond-with entered flag is set, set respondWithEntered to true.
  15. If e’s wait to respond flag is set, then:
    1. Wait until e’s wait to respond flag is unset.
    2. If e’s respond-with error flag is set, set handleFetchFailed to true.
    3. Else, set response to e’s potential response.
```

そして step 30 でこの `response` 変数を return する。

```
30. Return response.
```

以上が `respondWith()` を呼んで、何らかのレスポンスを返した場合の処理となる。ネットワークフォールバックの場合は `respondWith()` を呼ばないので、step 24.3.15 のサブステップは実行されず、`response` 変数は null のままになる。また、step 27 で `respondWithEntered` が false のままになっているため、step 27.3 で null を return する。

```
27. If respondWithEntered is false, then:
  1. If eventCanceled is true, then:
    1. If eventHandled is not null, then reject eventHandled with a "NetworkError" DOMException in workerRealm.
    2. Return a network error.
  2. If eventHandled is not null, then resolve eventHandled.
  3. Return null.
```

以上がネットワークフォールバックした場合の処理になる。

# Fetch

次に Handle Fetch アルゴリズムの呼び出し元を見ていく。これは Fetch 仕様の [HTTP fetch](https://fetch.spec.whatwg.org/#http-fetch) アルゴリズムにある。このアルゴリズムはその名の通り、HTTP を使ったリソースフェッチをするときに呼ばれる。このアルゴリズムの step 5.4 で Service Worker の Handle Fetch アルゴリズムを呼び出し、その結果を `response` 変数に入れる。前述の通り、ネットワークフォールバックした場合はこれが null になる。

```
5. If request’s service-workers mode is "all", then:
  (skip)
  4. Set response to the result of invoking handle fetch for requestForServiceWorker, with fetchParams’s controller and fetchParams’s cross-origin isolated capability. [HTML] [SW]
  (skip)
```

Step 6 で `response` が null の場合には step 6.3 でネットワークフェッチが行われ、そのレスポンスが step 9 で return される。

```
6. If response is null, then:
  (skip)
  3. Set response and actualResponse to the result of running HTTP-network-or-cache fetch given fetchParams.
  (skip)
(skip)
9. Return response.
```

# まとめ

本記事では Service Worker の FetchEvent がネットワークフォールバックした場合の処理について仕様を追ってみた。Service Worker 仕様は FetchEvent がどのように response を返すかだけを定義していて、その後のフォールバック処理は Fetch 仕様で定義されていることが分かった。このように仕様同士のシームレスな連携を確認するのは、仕様読みの醍醐味の一つですね。