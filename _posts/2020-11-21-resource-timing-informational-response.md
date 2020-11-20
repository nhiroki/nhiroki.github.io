---
layout: post
title: "Resource Timing と HTTP ステータスコード 1xx"
date: 2020-11-21 00:00:00 +09:00
tags: web
image: /images/profile.png
---

要約：Resource Timing の定義する responseStart イベントは informational response (1xx) をレスポンスの開始点として扱うので、1xx を返すアプリケーションで responseStart を記録する場合は注意が必要。

# Resource Timing

[Resource Timing 仕様](https://w3c.github.io/resource-timing/)はリソース読み込みに関する様々なイベントのタイムスタンプを取得するための JavaScript API を定義している。本記事執筆時点では以下のイベントが定義されている。各イベントの詳細は [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Resource_Timing_API/Using_the_Resource_Timing_API) などを見てください。

```
[Exposed=(Window,Worker)]
interface PerformanceResourceTiming : PerformanceEntry {
    readonly        attribute DOMString           initiatorType;
    readonly        attribute DOMString           nextHopProtocol;
    readonly        attribute DOMHighResTimeStamp workerStart;
    readonly        attribute DOMHighResTimeStamp redirectStart;
    readonly        attribute DOMHighResTimeStamp redirectEnd;
    readonly        attribute DOMHighResTimeStamp fetchStart;
    readonly        attribute DOMHighResTimeStamp domainLookupStart;
    readonly        attribute DOMHighResTimeStamp domainLookupEnd;
    readonly        attribute DOMHighResTimeStamp connectStart;
    readonly        attribute DOMHighResTimeStamp connectEnd;
    readonly        attribute DOMHighResTimeStamp secureConnectionStart;
    readonly        attribute DOMHighResTimeStamp requestStart;
    readonly        attribute DOMHighResTimeStamp responseStart;
    readonly        attribute DOMHighResTimeStamp responseEnd;
    readonly        attribute unsigned long long  transferSize;
    readonly        attribute unsigned long long  encodedBodySize;
    readonly        attribute unsigned long long  decodedBodySize;
    [Default] object toJSON();
};
```

各イベントはざっくり以下のようにマッピングされる (図は Resource Timing の仕様より引用)

![表紙](/images/resource-timing-informational-response.webp)

今回はこのうちの responseStart に関する話。responseStart は HTTP レスポンスヘッダを受信して処理し始めたタイミングを表す。これは TTFB (Time To First Byte) とも呼ばれる。仕様では次のように定義されている。

> The time immediately after the user agent's HTTP parser receives the first byte of the response (e.g. frame header bytes for HTTP/2, or response status line for HTTP/1.x) from relevant application caches, or from local resources or from the server, if the resource passes the timing allow check algorithm.
>
> ― [4.3 The PerformanceResourceTiming Interface - responseStart](https://w3c.github.io/resource-timing/#dom-performanceresourcetiming-responsestart)

# Informational Response (1xx)

HTTP では最終的なレスポンスを返す前に補足情報を載せたレスポンスを返すことができる。このレスポンスには HTTP ステータスコード 1xx が使われる。1xx のレスポンスは [informational response](https://tools.ietf.org/html/rfc7231#section-6.2) と呼ばれ、現在以下のものが定義されている。

- 100 Continue : リクエストを受信中で、完了次第処理を継続することをクライアントに通知する。
- 101 Switching Protocols : プロトコルの切り替えをクライアントに要求する。WebSocket などで使われる。
- 102 Processing : リクエストが処理中であることをクライアントに通知する。
- 103 Early Hints : preload などのためのヒントをクライアントに与える。現時点ではまだどのブラウザも実装していない。

Informational response は最終 response (これを non-informational response と呼ぶことにする) に先立って送られてくる。例えば、

```
1xx --> 1xx --> ... --> 1xx --> 200
```

みたくなる。Informational response については[一つ前の記事](/2020/11/16/chromium-http-stream-parser)でも触れたのでそちらも見てください。

# responseStart と Informational Response

informational response (1xx) を受信した場合、responseStart はそれの first byte をレスポンスの開始点として扱う。つまり

```
1xx --> 1xx --> ... --> 1xx --> <responseStart> 200 <responseEnd>
```

ではなく

```
<responseStart> 1xx --> 1xx --> ... --> 1xx --> 200 <responseEnd>
```

みたくなる。仕様には以下のように記載されている。

> NOTE
>
> ... snip ...
>
> For fetches composed of multiple requests (e.g. preflights, authentication challenge-response, redirects, and so on), the reported responseStart value is that of the last request. **In the case where more than one response is available for a request, due to an Informational 1xx response, the reported responseStart value is that of the first response to the last request.**
>
> ― [4.3 The PerformanceResourceTiming Interface - responseStart](https://w3c.github.io/resource-timing/#dom-performanceresourcetiming-responsestart)

以上より、もし 1xx を返すアプリケーションで responseStart を記録する場合は注意が必要になる。200 の start と end を記録しているつもりが 1xx を含んだ時間を記録しているかもしれない。現在の Resource Timing 仕様には non-informational response の first byte の時間を取る方法は定義されていません。

ただ現状 1xx を使っていてさらに Resource Timing でメトリクスを取っているケースはほとんどないだろうし、これが問題になることはまずないと思います。一方、もし今後 103 Early Hints が実装されて広く使われるようになったら問題になるかもしれません。