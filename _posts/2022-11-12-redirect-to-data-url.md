---
layout: post
title: "fetch() が data URL へリダイレクトしたときの仕様を読み解く"
date: 2022-11-12 00:00:00 +09:00
tags: web
image: /images/profile.png
---

`fetch()` はリクエストが data URL にリダイレクトした場合はネットワークエラーを返す。これが仕様でどのように定まっているのか調べたのでメモとして残す。なお、参照している仕様は 2022 年 11 月 12 日現在のものであり、その後変更されている可能性があるので注意されたし。

# Fetch アルゴリズム

`fetch()` の仕様は [Fetch Standard](https://fetch.spec.whatwg.org/) で規定されている。`fetch()` のエントリーポイントは [fetch(input, init) (#dom-global-fetch)](https://fetch.spec.whatwg.org/#dom-global-fetch) アルゴリズムである。このアルゴリズムでは request をセットアップし、ステップ 12 で fetch (#concept-fetch) を呼ぶ。

> The fetch(input, init) method steps are:
> 
> - 12\. Set controller to the result of calling fetch given request and processResponse given response being these steps:

[fetch (#concept-fetch)](https://fetch.spec.whatwg.org/#concept-fetch) アルゴリズムは fetch を行うためのパラメータ (fetchParams) をセットアップし、ステップ 17 で main fetch アルゴリズムを呼ぶ。

> - 17\. Run main fetch given fetchParams.

[main fetch](https://fetch.spec.whatwg.org/#concept-main-fetch) アルゴリズムはリクエストのスキームやモードによってこの先のリクエストアルゴリズムを切り替える。リダイレクト前のリクエストが same-origin かつ CORS リクエストではない場合、ステップ 11 で scheme fetch アルゴリズムを呼ぶ。なお、直接 data URL に対してリクエストした場合もこの分岐で scheme fetch アルゴリズムを呼ぶ。

> To main fetch, given a fetch params fetchParams and an optional boolean recursive (default false), run these steps:
>
> - 11\. If response is null, then set response to the result of running the steps corresponding to the first matching statement:
>   - request’s current URL’s origin is same origin with request’s origin, and request’s response tainting is "basic"
>   - request’s current URL’s scheme is "data"
>   - request’s mode is "navigate" or "websocket"
>     - 1\. Set request’s response tainting to "basic".
>     - 2\. Return the result of running scheme fetch given fetchParams.

main fetch アルゴリズムに付記された NOTE は面白い。読み込まれた Documents や Web Workers はそれを生成した URL の origin を自身の origin として設定し、same-origin policy のチェックなどに用いる。しかし data URL の場合は origin を持たないため、data URL から生成された Documents や Web Workers は unique opaque origin になる。詳しくは以前書いた「[environment settings object の origin の仕様を追う](/2020/02/16/spec-environment-settings-object-origin)」を参照されたい。

> NOTE: HTML assigns any documents and workers created from URLs whose scheme is "data" a unique opaque origin. Service workers can only be created from URLs whose scheme is an HTTP(S) scheme. [HTML] [SW]

[scheme fetch](https://fetch.spec.whatwg.org/#concept-scheme-fetch) アルゴリズムはリクエスト URL の scheme に応じてリクエスト処理を行う。現在このアルゴリズムでは "about", "blob", "data", "file", HTTP(S) に対する振り分けを行っている。リダイレクト前のリクエストは HTTP(S) であるため、今回は HTTP fetch アルゴリズムを呼ぶ。

> To scheme fetch, given a fetch params fetchParams:
>
> - 3\. Switch on request’s current URL’s scheme and run the associated steps:
>   - "HTTP(S) scheme"
>     - Return the result of running HTTP fetch given fetchParams.

[HTTP fetch](https://fetch.spec.whatwg.org/#concept-http-fetch) アルゴリズムは HTTP(S) scheme に対するリクエスト処理を担う。ステップ 6.3 で [HTTP-network-or-cache fetch](https://fetch.spec.whatwg.org/#concept-http-network-or-cache-fetch) アルゴリズムを呼び、実際の処理を行う。今回はこの先の処理は重要ではないので深追いしない。HTTP-network-or-cache fetch アルゴリズムの結果は actualResponse 変数に格納される。ステップ 8 で actualResponse がリダイレクトだった場合を処理しており、リダイレクトモードによって違う挙動をする。今回はリダイレクトモードが "follow" の場合を見ていく。この場合 HTTP-redirect fetch アルゴリズムを呼ぶ。

> To HTTP fetch, given a fetch params fetchParams and an optional boolean makeCORSPreflight (default false), run these steps:
>
> - 6\. If response is null, then:
>   - 6.3. Set response and actualResponse to the result of running HTTP-network-or-cache fetch given fetchParams.
> - 8\. If actualResponse’s status is a redirect status, then:
Switch on request’s redirect mode:
>   - "error"
>     - Set response to a network error.
>   - "manual"
>     - Set response to an opaque-redirect filtered response whose internal response is actualResponse. If request’s mode is "navigate", then set fetchParams’s controller’s next manual redirect steps to run HTTP-redirect fetch given fetchParams and response.
>   - "follow"
>     - Set response to the result of running HTTP-redirect fetch given fetchParams and response.
> - 9\. Return response.

なお HTTP fetch  アルゴリズムはステップ 5 で Service Worker に対してネットワークリクエストをインターセプトさせる処理も行っており、これも面白い。詳しくは「[Service Worker の FetchEvent がネットワークフォールバックした場合の仕様を読み解く](/2022/09/10/service-worker-fetch-event-network-fallback)」という記事を参照されたい。

> To HTTP fetch, given a fetch params fetchParams and an optional boolean makeCORSPreflight (default false), run these steps:
>
> - 5\. If request’s service-workers mode is "all", then:
>   - 4\. Set response to the result of invoking handle fetch for requestForServiceWorker, with fetchParams’s controller and fetchParams’s cross-origin isolated capability. [HTML] [SW]

[HTTP-redirect fetch](https://fetch.spec.whatwg.org/#concept-http-redirect-fetch) アルゴリズムはリダイレクト処理を行う。ステップ 3 ではリダイレクトレスポンスに指定された Location ヘッダをパースし、locationURL 変数に格納する。もし locationURL が null や failure だった場合は network error を返す。そして最も大事なのはステップ 6 で、locationURL の scheme が HTTP(S) でなかった場合に network error を返す。data URL はもちろん HTTP(S) ではないので、ここで network error が返ることになる。ようやく今回の主題であった「リクエストが data URL にリダイレクトした場合はネットワークエラーを返す」が定義されている箇所にたどり着くことができた。

> To HTTP-redirect fetch, given a fetch params fetchParams and a response response, run these steps:
>
> 3\. Let locationURL be actualResponse’s location URL given request’s current URL’s fragment.
> 4\. If locationURL is null, then return response.
> 5\. If locationURL is failure, then return a network error.
> 6\. If locationURL’s scheme is not an HTTP(S) scheme, then return a network error.

なお、locationURL が HTTP(S) であった場合は、fetchParams をリダイレクト先に再設定した上で main fetch アルゴリズムを呼び、一連の処理をやり直す。

> To HTTP-redirect fetch, given a fetch params fetchParams and a response response, run these steps:
>
> 19\. Return the result of running main fetch given fetchParams and true.

# まとめ

本記事では `fetch()` リクエストが data URL にリダイレクトした場合にネットワークエラーを返す挙動が仕様でどのように定義されているのか追った。なおこの挙動は Web Platform Tests (WPT) の [wpt /fetch/api/redirect/redirect-to-dataurl.any.*](https://wpt.fyi/results/fetch/api/redirect?label=master&label=experimental&aligned&view=subtest) でテストされている。