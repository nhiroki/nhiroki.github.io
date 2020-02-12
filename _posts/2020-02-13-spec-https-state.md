---
layout: post
title: "HTTPS state の仕様を追う"
date: 2020-02-13 00:00:00 +09:00
tags: web
image: /images/profile.png
---

HTTPS state とそれが non-http(s) scheme fetch でどのように振る舞うのか仕様を調べたメモです。

この記事は 2020 年 2 月 12 日時点の各種仕様を元に記述しており、現時点では参照している仕様が更新されていたり、リンクが切れている可能性があります。ご了承ください。

# HTTPS state in Fetch standard

[Fetch standard](https://fetch.spec.whatwg.org/) は HTTPS state value という属性値を定義している。HTTPS state value は { "none", "deprecated", "modern" } のどれかを取ることができる。

> An HTTPS state value is "none", "deprecated", or "modern".
>
> [https://fetch.spec.whatwg.org/#concept-https-state-value](https://fetch.spec.whatwg.org/#concept-https-state-value)

同じく Fetch standard が定義する response は HTTPS state というフィールドを持ち、これは前述の HTTPS state value を保持する。初期値は "none" になる。

> A response has an associated HTTPS state (an HTTPS state value). Unless stated otherwise, it is "none".
>
> [https://fetch.spec.whatwg.org/#concept-response-https-state](https://fetch.spec.whatwg.org/#concept-response-https-state)

この属性値は response の配送方法によって決まる。HTTPS で response が配送されれば "modern" になる。ただし、TLS が使用するハッシュ関数や暗号スイートなどが deprecated なものの場合、response の HTTPS state value は "deprecated" になる可能性がある。どのような場合に "deprecated" になるかは Fetch standard は意図して厳密には定義していない。

> A response delivered over HTTPS will typically have its HTTPS state set to "modern". A user agent can use "deprecated" in a transition period. E.g., while removing support for a hash function, weak cipher suites, certificates for an "Internal Name", or certificates with an overly long validity period. How exactly a user agent can use "deprecated" is not defined by this specification.
>
> [https://fetch.spec.whatwg.org/#concept-https-state-value](https://fetch.spec.whatwg.org/#concept-https-state-value)

# HTTPS state in HTML standard

[HTML standard](https://html.spec.whatwg.org/multipage/) 側では environmental settings object が HTTPS state を持つと定義している。

> An environment settings object is an environment that additionally specifies algorithms for:
>
> **An HTTPS state**: An HTTPS state value representing the security properties of the network channel used to deliver the resource with which the environment settings object is associated.
>
> [https://html.spec.whatwg.org/multipage/webappapis.html#https-state](https://html.spec.whatwg.org/multipage/webappapis.html#https-state)

Environmental settings object は主に Document の場合と Worker の場合があるが、以降は Worker の場合を見ていく。Worker の environmental settings object の HTTPS state は以下のように定義されている。

> Let settings object be a new environment settings object whose algorithms are defined as follows:
>
> **The HTTPS state**: Return worker global scope's HTTPS state.
>
> [https://html.spec.whatwg.org/multipage/workers.html#script-settings-for-workers:https-state](https://html.spec.whatwg.org/multipage/workers.html#script-settings-for-workers:https-state)

worker global scope の HTTP state を返していることが分かる。worker global scope の HTTP state は次のように定義されている。

> A WorkerGlobalScope object has an associated HTTPS state (an HTTPS state value). It is initially "none".
>
> [https://html.spec.whatwg.org/multipage/workers.html#concept-workerglobalscope-https-state](https://html.spec.whatwg.org/multipage/workers.html#concept-workerglobalscope-https-state)

これは top-level worker script fetch 後の worker global scope の初期化時に次のように設定されている。

> In both cases, to perform the fetch given request, perform the following steps if the is top-level flag is set:
> 1. Set request's reserved client to inside settings.
> 2. Fetch request, and asynchronously wait to run the remaining steps as part of fetch's process response for the response response.
> 3. Set worker global scope's url to response's url.
> 4. **Set worker global scope's HTTPS state to response's HTTPS state.**
> 5. ...
>
> [https://html.spec.whatwg.org/multipage/workers.html#worker-processing-model:concept-workerglobalscope-https-state](https://html.spec.whatwg.org/multipage/workers.html#worker-processing-model:concept-workerglobalscope-https-state)

要するに top-level worker script に対応する response が持つ HTTPS state を worker の environmental settings object に設定している。以上より、Fetch standard の定義する response HTTPS state がどのように HTML standard の定義する environmental settings object の HTTPS state に関連付けられるかが分かった。

environmental settings object がその初期化時に response の HTTPS state を受け継ぐということは Fetch standard 側にも記載がある。

> An environment settings object typically derives its HTTPS state from a response.
>
> [https://fetch.spec.whatwg.org/#concept-https-state-value](https://fetch.spec.whatwg.org/#concept-https-state-value)

# HTTPS state in Mixed Content standard

HTTPS state は mixed content のチェックに使われる。これは [Mixed Content standard](https://w3c.github.io/webappsec-mixed-content/) で定義されている。

まず unauthenticated response というコンセプトが次のように定義されている。

> **unauthenticated response**
>
> We know a posteriori that a response (response) is unauthenticated if either of the following statements is false:
> - response’s url is a priori authenticated.
> - If response’s url’s scheme is "https" or "wss", response’s HTTPS state is "modern".
>
> [https://w3c.github.io/webappsec-mixed-content/#unauthenticated-response](https://w3c.github.io/webappsec-mixed-content/#unauthenticated-response)

これをつかって response が mixed content であるとは次のように定義されている。

> **mixed content**
>
> A response is mixed content if it is an unauthenticated response, and the context responsible for loading it requires prohibits mixed security contexts.
>
> [https://w3c.github.io/webappsec-mixed-content/#mixed-content](https://w3c.github.io/webappsec-mixed-content/#mixed-content)

"the context responsible for loading it requires probihits mixed security contexts" の部分は次のように定義されている。

> **5.1. Does settings prohibit mixed security contexts?**
>
> Both documents and workers have environment settings objects which may be examined according to the following algorithm in order to determine whether they restrict mixed content. This algorithm returns "Prohibits Mixed Security Contexts" or "Does Not Prohibit Mixed Security Contexts", as appropriate.
>
> Given an environment settings object (settings):
>
> 1. If settings’ HTTPS state is not "none", then return "Prohibits Mixed Security Contexts".
> 2. If settings has a responsible document document, then:
>     1. While document has an embedding document:
>         1. Set document to document’s embedding document.
>         2. Let embedder settings be document’s global object's relevant settings object.
>         3. If embedder settings’ HTTPS state is not "None", then return "Prohibits Mixed Security Contexts".
> 3. Return "Does Not Restrict Mixed Security Contexts".
>
> [https://w3c.github.io/webappsec-mixed-content/#categorize-settings-object](https://w3c.github.io/webappsec-mixed-content/#categorize-settings-object)

# Data URL のような http(s) scheme を持たない場合

Data URL は ```data:[<mime type>][;base64],<base64 data>``` のような構造を持つ URL で、Base64 形式でエンコードしたデータを URL 内に埋め込むことができる。これに対するリクエストはネットワークに送られる代わりにブラウザ内で ```<base64 data>``` 部分をデコードして、あたかもネットワークから response が送られてきたかのように処理を行う。

このとき http や https といった scheme は使われないため、response から HTTPS state value を決めることができない。そこで Data URL に対してリクエストを投げた environment settings object (a.k.a. fetch client settings object) の HTTPS state が使われる。これは Fetch standard の Scheme fetch アルゴリズムに定義されている。

> **4.2. Scheme fetch**
>
> To perform a scheme fetch using request, switch on request’s current URL’s scheme, and run the associated steps:
> - "data"
>   1. Let dataURLStruct be the result of running the data: URL processor on request’s current URL.
>   2. If dataURLStruct is failure, then return a network error.
>   3. Return a response whose status message is `OK`, header list consist of a single header whose name is `Content-Type` and value is dataURLStruct’s MIME type, serialized, body is dataURLStruct’s body, and **HTTPS state is request’s client’s HTTPS state if request’s client is non-null**.
> 
> [https://fetch.spec.whatwg.org/#scheme-fetch](https://fetch.spec.whatwg.org/#scheme-fetch)

これはつまり Data URL を使って iframe や worker を作った場合は親のコンテキスト (environmental settings object) の HTTPS state を引き継ぐことを意味する。

# まとめ

- 仕様により HTTPS state という属性値が定義されている。
  - Fetch standard は HTTPS state の具体的な値を定義し、response がそれを持つと定めている。
  - HTML standard は environmental settings object が HTTPS state を持ち、それは response のものを引き継ぐと定義している。
  - Mixed Content standard は mixed content のチェックのために HTTPS state を使っている。
- Data URL のような non-http(s) scheme fetch では fetch client settings object の HTTPS state が response の HTTPS state となる。
  - Data URL から作られた iframe や worker は親コンテキスト (environmental settings object) の HTTPS State を引き継ぐ。