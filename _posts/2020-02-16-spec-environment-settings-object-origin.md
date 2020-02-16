---
layout: post
title: "environment settings object の origin の仕様を追う"
date: 2020-02-16 00:00:00 +09:00
tags: web
image: /images/profile.png
---

iframe や worker といった実行コンテキスト (environment settings object) が持つ origin がどのように決められるのか仕様を調べたメモです。長文でかつ難解なので、結論だけ知りたい方はまとめを読んでください。

この記事は 2020 年 2 月 16 日時点の各種仕様を元に記述しており、現時点では参照している仕様が更新されていたり、リンクが切れている可能性があります。ご了承ください。

**目次**

- [はじめに](#はじめに)
- [iframe の場合 (about:blank)](#iframe-の場合-aboutblank)
- [iframe の場合 (navigate)](#iframe-の場合-navigate)
- [Web Worker (Dedicated Worker / Shared Worker) の場合](#web-worker-dedicated-worker--shared-worker-の場合)
- [Service Worker の場合](#service-worker-の場合)
- [Worklet の場合](#worklet-の場合)
- [まとめと雑感](#まとめと雑感)

# はじめに

[HTML standard](https://html.spec.whatwg.org/multipage/) では iframe や worker といった実行コンテキストを抽象化したコンセプトの一つ[^one-of-concepts]として environment settings object を定義しています。この environment settings object は origin を持ち、これが [Same Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) のようなセキュリティチェックに使われます。

[^one-of-concepts]: "抽象化したコンセプトの一つ" とわざと曖昧な表現をしています。HTML standard では実行コンテキストにほぼ 1 対 1 対応するコンセプトが複数定義されていて複雑な状況になっており、私もちゃんと把握できていません。興味がある方はぜひ調べて教えて下さい。

> An **environment settings object** is an environment that additionally specifies algorithms for:
>
> - **An origin**: An origin used in security checks.
>
> [https://html.spec.whatwg.org/multipage/webappapis.html#concept-settings-object-origin](https://html.spec.whatwg.org/multipage/webappapis.html#concept-settings-object-origin)

この origin はその実行コンテキストのソースとなる URL (例えば ```iframe.src```) によって決定される気がしますが、果たして本当にそうでしょうか？本記事では environment settings object の origin がどのように決められているのか仕様を追ってみます。

# iframe の場合 (about:blank)

まずは iframe の場合を見ていきます。iframe は src に URL を指定してそれをドキュメントに追加するときに新しく nested browsing context というものを作ります。browsing context が何なのかは気にしなくて大丈夫です。

> When an iframe element element is inserted into a document whose browsing context is non-null, the user agent must run these steps:
>
> 1: **Create a new nested browsing context** for element.
>
> 2: Process the iframe attributes for the "first time".
>
> [https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element)

**create a new nested browsing context** アルゴリズムは次のように定義されています。Step 1 の **creating a new browsing context** が怪しいのでさらに見ていきます。

> To **create a new nested browsing context**, given an element element:
>
> 1: Let browsingContext be the result of **creating a new browsing context** with element's node document and element's node document's browsing context's top-level browsing context's group.
>
> [https://html.spec.whatwg.org/multipage/browsers.html#creating-a-new-nested-browsing-context](https://html.spec.whatwg.org/multipage/browsers.html#creating-a-new-nested-browsing-context)

**create a new browsing context** アルゴリズムは次のように定義されています。Step 4 で **determining the origin** アルゴリズムを呼んで、その結果を origin 変数に設定していることが分かります。origin 変数は step 6 で agent に、agent は step 7 で realm execution context に渡されます。このとき global object として新しく Window object を生成します。そして step 8 で **set up a window environmetn settings object** アルゴリズムに realm execution context を渡します。Step 9 では新しく Document を作り、その document's origin に先程決定した origin を設定します。Step 11 でこの document を browsingContext の active document として設定します。

> To **create a new browsing context**, given null or a Document object creator and browsing context group group:
>
> 1: Let browsingContext be a new browsing context.
>
> 4: Let **origin** be the result of **determining the origin** given browsingContext, about:blank, sandboxFlags, browsingContext's creator origin, and null.
>
> 6: Let **agent** be the result of obtaining a similar-origin window agent given **origin** and group.
>
> 7: Let **realm execution context** be the result of creating a new JavaScript realm with the following customizations:
> - For the agent, use agent.
> - **For the global object, create a new Window object.**
> - For the global this binding, use browsingContext's WindowProxy object.
>
> 8: **Set up a window environment settings object with realm execution context**, and let settingsObject be the result.
>
> 9: **Let document be a new Document**, marked as an HTML document in quirks mode, whose content type is "text/html", **origin is origin**, active sandboxing flag set is sandboxFlags, feature policy is feature policy, and which is both ready for post-load tasks and completely loaded immediately.
>
> 11: **Set the active document** of browsingContext to document.
>
> [https://html.spec.whatwg.org/multipage/browsers.html#creating-a-new-browsing-context](https://html.spec.whatwg.org/multipage/browsers.html#creating-a-new-browsing-context)

**determine the origin** アルゴリズムを見る前に **set up a window environment settings object** アルゴリズムを見ていきます。Step 2 で realm execution context の global object を window 変数に設定しています。この global object とは前アルゴリズムの step 7 で生成した Window object になります。そして step 4 で environment settings object の origin に the origin of window's associated Document を設定しています。この window's associated Document はこの時点では設定されておらず、前アルゴリズムの step 11 で **set the active document** アルゴリズムを呼んだときに設定されます。

> When the user agent is required to **set up a window environment settings object**, given a JavaScript execution context execution context and an optional environment reserved environment, it must run the following steps:
>
> 1: Let realm be the value of execution context's Realm component.
>
> 2: Let window be realm's global object.
>
> 4: Let settings object be a new environment settings object whose algorithms are defined as follows:
> - **The origin**: Return **the origin of window's associated Document**.
>
> [https://html.spec.whatwg.org/multipage/window-object.html#set-up-a-window-environment-settings-object](https://html.spec.whatwg.org/multipage/window-object.html#set-up-a-window-environment-settings-object)

**set the active document** アルゴリズムは次の通りで、この step 3 で window's associated Document の設定が行われています。

> To **set the active document** of a browsing context browsingContext to a Document object document, run these steps:
>
> 1: Let window be document's relevant global object.
>
> 2: Set browsingContext's WindowProxy object's [[Window]] internal slot value to window.
>
> 3: Set window's associated Document to document.
>
> 4: Set window's relevant settings object's execution ready flag.

さてさて、**determine the origin** アルゴリズムで決定された origin が window's associated Document に設定されて、それが environment settings object の origin として見えることが分かりました。一件落着。

・・・と思いたいところですが、実はまだ終わりではない！勘の鋭い方は気づかれたかもしれませんが、実はこの時点では iframe の src に指定した URL は読み込まれておらず、Document は一時的なものが作られています。

以下は **create a new browsing context** アルゴリズムの step 4 を再掲したものです。ここで **determine the origin** アルゴリズムを呼んでいますが、その引数に about:blank が指定されています。

> 4: Let **origin** be the result of **determining the origin** given browsingContext, **about:blank**, sandboxFlags, browsingContext's creator origin, and null.

**Determine the origin** アルゴリズムは次のようになっています。url 引数と invocationOrigin 引数が重要で、それぞれ about:blank と browsingContext's creator origin が設定されています。これにより、step 4 にマッチし、この時点では invocationOrigin (i.e., creator origin) が返ります。

> To determine the origin, given browsing context browsingContext, URL url, sandboxing flag set sandboxFlags, and two origins invocationOrigin and activeDocumentNavigationOrigin:
>
> 1: If sandboxFlags has its sandboxed origin browsing context flag set, then return a new opaque origin.
>
> 2: If url is null, then return a new opaque origin.
>
> 3: If activeDocumentOrigin is not null, and url's scheme is "javascript", then return activeDocumentNavigationOrigin.
>
> 4: If invocationOrigin is non-null and url is about:blank, then return invocationOrigin.
>
> 5: If url is about:srcdoc, then return the origin of browsingContext's container document.
>
> 6: Return url's origin.
>
> [https://html.spec.whatwg.org/multipage/browsers.html#determining-the-origin](https://html.spec.whatwg.org/multipage/browsers.html#determining-the-origin)

以上のように、iframe は初期生成時は about:blank が読み込まれ、その origin は creator origin が設定されます。それでは iframe の src に指定した URL はいつ読まれ、そしていつ origin は設定されるのでしょうか？

Window の note セクションには次のように書かれています。どうやら Window object に紐付いた Document はまず about:blank で読み込まれ、その後 navigate 時に新しいものに入れ替わるようです。

> The Document object associated with a Window object can change in exactly one case: when the navigate algorithm creates a new Document object for the first page loaded in a browsing context. In that specific case, the Window object of the original about:blank page is reused and gets a new Document object.
>
> [https://html.spec.whatwg.org/multipage/window-object.html#the-window-object](https://html.spec.whatwg.org/multipage/window-object.html#the-window-object)

# iframe の場合 (navigate)

前節では iframe の初期生成時の挙動を追いました。それによると iframe はまず about:blank を読み込み、environment settings object の origin として creator origin を指定することが分かりました。次に iframe は src に基づいて navigate を行うことが予想されます。その処理を追っていきます。

次のアルゴリズムは前節の最初に示した iframe がドキュメントに追加されるときに実行されるものです。Step 1 で about:blank を読み込んだ iframe が生成されるのは今まで見てきた通りです。次に step 2 を見ていきます。

> When an iframe element element is inserted into a document whose browsing context is non-null, the user agent must run these steps:
>
> 1: Create a new nested browsing context for element.
>
> 2: **Process the iframe attributes** for the "first time".
>
> [https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element)

**Process the iframe attributes** アルゴリズムは次の通りです。今回は iframe の srcdoc は指定されておらず、src が指定されていることを想定しているので、3 つ目の "Otherwise" 節が実行されます。

> When the user agent is to **process the iframe attributes**, it must run the first appropriate steps from the following list:
>
> If the srcdoc attribute is specified
> - ...
>
> Otherwise, if the element has no src attribute specified, and the user agent is processing the iframe's attributes for the "first time"
> - ...
>
> **Otherwise**
> - **Run the otherwise steps for iframe or frame elements.**
>
> [https://html.spec.whatwg.org/multipage/iframe-embed-object.html#process-the-iframe-attributes](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#process-the-iframe-attributes)

**the otherwise steps** は次の通りです。Step 1 では src attribute が指定されているのでそれの parse が行われ、その結果の URL が url 変数に設定されます。Step 3 では url に対する request が生成され、step 4 でそれに対して navigate が実行されます。

> The **otherwise steps** for iframe or frame elements are as follows:
>
> 1: If the element has no src attribute specified, or its value is the empty string, let url be the URL "about:blank".
> - **Otherwise, parse the value of the src attribute, relative to the element's node document.**
> - If that is not successful, then let url be the URL "about:blank". Otherwise, let url be the resulting URL record.
>
> 3: **Let resource be a new request whose url is url** and whose referrer policy is the current state of the element's referrerpolicy content attribute.
>
> 4: **Navigate the element's nested browsing context to resource.**
>
> [https://html.spec.whatwg.org/multipage/iframe-embed-object.html#otherwise-steps-for-iframe-or-frame-elements](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#otherwise-steps-for-iframe-or-frame-elements)

**Navigate** アルゴリズムは次の通りです。このアルゴリズムは長いのでだいぶ端折ってしまいますが、前アルゴリズムで準備した request をネットワークに送り、その response を step 16 で処理しています。

> 16: This is the step that attempts to obtain resource, if necessary. Jump to the first appropriate substep:
>
> If resource is a response
> - Run **process a navigate response** with null, resource, navigationType, the source browsing context, browsingContext, sandboxFlags, incumbentNavigationOrigin, and activeDocumentNavigationOrigin.
>
> [https://html.spec.whatwg.org/multipage/browsing-the-web.html#navigate](https://html.spec.whatwg.org/multipage/browsing-the-web.html#navigate)

**process a navigate response** アルゴリズムでは response の処理が行われます。今回は iframe の読み込みを行っているので、response が HTML MIME type を持つとして処理を追っていきます。

> 5: If the user agent has been configured to process resources of the given type using some mechanism other than rendering the content in a browsing context, then skip this step. Otherwise, if the type is one of the following types, jump to the appropriate entry in the following list, and process response as described there:
>
> - **an HTML MIME type**: Follow the steps given in the **HTML document** section providing browsingContext, request, response, sandboxFlags, incumbentNavigationOrigin, and activeDocumentNavigationOrigin. Once the steps have completed, return.
>
> [https://html.spec.whatwg.org/multipage/browsing-the-web.html#process-a-navigate-response](https://html.spec.whatwg.org/multipage/browsing-the-web.html#process-a-navigate-response)

HTML MIME type の場合は次の **an HTML document is to be loaded** アルゴリズムが実行されます。Step 1 で Document object の生成と初期化を行います。

> When an HTML document is to be loaded in a browsing context, provided browsingContext, request, response, sandboxFlags, incumbentNavigationOrigin, and activeDocumentNavigationOrigin, the user agent must queue a task on the networking task source to:
>
> 1: Let document be the result of **creating and initializing a Document object** providing "html", "text/html", request, response, browsingContext, sandboxFlags, incumbentNavigationOrigin, and activeDocumentNavigationOrigin.
>
> After creating the Document object, but before any script execution, certainly before the parser stops, the user agent must update the session history with the new page.
>
> [https://html.spec.whatwg.org/multipage/browsing-the-web.html#navigate-html](https://html.spec.whatwg.org/multipage/browsing-the-web.html#navigate-html)

**create and initialize a Document object** アルゴリズムを見ていきます。Step 2 でついに src に指定された URL で **determine the origin** アルゴリズムが呼ばれます。Step 4 では session history に about:blank Document のみがある場合、つまり iframe の初期化処理だけが行われている場合で、かつその window's associated Document の origin が same origin だった場合 (ここで associated Document の origin は about:blank の場合は creator origin になっていたことを思い出す) は、step 5 をスキップします。Step 5 では cross origin navigation の場合に window を作り直します。Step 6 で新しい document を設定し、その origin を **determine the origin** アルゴリズムで決定したものにします。この時点では window's associated Document の変更は行わずに、step 14 で document を return します。

> Some of the sections below, to which the above algorithm defers in certain cases, use the following steps to create and initialize a Document object, given a type type, content type contentType, a request request, a response response, a browsing context browsingContext, a sandboxing flag set sandboxFlags, two origins incumbentNavigationOrigin, activeDocumentNavigationOrigin, and an optional environment reservedEnvironment:
> 
> 2: **Let origin be the result of determining the origin given browsingContext, request's url, finalSandboxFlags, incumbentNavigationOrigin, and activeDocumentNavigationOrigin.**
> 
> 4: If browsingContext's only entry in its session history is the about:blank Document that was added when browsingContext was created, and navigation is occurring with replacement enabled, and that Document has the same origin as origin, then do nothing.
>
> 5: Otherwise:
> - 5.1: Let agent be the result of obtaining a similar-origin window agent given origin and browsingContext's group.
> - 5.2: Let realm execution context be the result of creating a new JavaScript realm with the following customizations:
>   - For the agent, use agent.
>   - For the global object, create a new Window object.
>   - For the global this binding, use browsingContext's WindowProxy object.
> - 5.3: **Set up a window environment settings object** with realm execution context and reservedEnvironment, if present.
>
> 6: **Let document be a new Document**, whose type is type, content type is contentType, **origin is origin**, feature policy is featurePolicy, and active sandboxing flag set is finalSandboxFlags.
>
> 14: Return document.
>
> [https://html.spec.whatwg.org/multipage/browsing-the-web.html#initialise-the-document-object](https://html.spec.whatwg.org/multipage/browsing-the-web.html#initialise-the-document-object)

一つ前の **an HTML document is to be loaded** アルゴリズムに戻ります。Document object を生成後に session history を更新しろとあるので、そのアルゴリズムを見ていきます。

> After creating the Document object, but before any script execution, certainly before the parser stops, the user agent must **update the session history with the new page**.
>
> [https://html.spec.whatwg.org/multipage/browsing-the-web.html#navigate-html](https://html.spec.whatwg.org/multipage/browsing-the-web.html#navigate-html)

**[Update the session history with the new page](https://html.spec.whatwg.org/multipage/browsing-the-web.html#update-the-session-history-with-the-new-page)** アルゴリズムについては条件分岐を追うのが煩雑なので省略しますが、いずれにせよこの中で **traverse the history** アルゴリズムが呼ばれます。

**traverse the history** アルゴリズムの step 4.3 で前述の **set the active document** アルゴリズムが呼ばれ、ここで window's associated Document が入れ替えられ、environment settings object の origin が期待される origin を返すようになります。長かった！

> 4.3: Set the active document of the browsing context to entry's Document object.
>
> [https://html.spec.whatwg.org/multipage/browsing-the-web.html#traverse-the-history](https://html.spec.whatwg.org/multipage/browsing-the-web.html#traverse-the-history)

最後に navigate 時の **determine the origin** アルゴリズムについて追っておきます。Step 1 では sandboxFlags の有無を確認し、フラグがある場合は opaque origin を返します。iframe には [sandbox attribute](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#attr-iframe-sandbox) を指定することができ、その iframe 内で実行できる処理に制限を加えることができます。[Opaque origin](https://html.spec.whatwg.org/multipage/origin.html#concept-origin-opaque) とはそれ自身を含む如何なる origin ともマッチしない origin です。 これはつまり Same Origin Policy を満たせなくなるので、それを要求するパワフルな API を使用不能にすることができます。これは sandbox を実現する上で好ましい制約です。Step 2 では URL が null のときに opaque origin を返します。null URL は origin を持たないので opaque origin になります。Step 3 では URL が javascript scheme を持つときにその URL に navigate させた origin を返しています。Step 4 は about:blank の場合の処理で、これは iframe の初回処理時に確認しました。Step 5 は URL が about:srcdoc だった場合で、この場合は container document の origin を返すようですが、今回は追ってないので詳細は分からないです。Step 6 は URL の origin を返します。今回は一般的な URL を iframe src に指定した場合を想定しているので、この step 6 で処理されます。

> To **determine the origin**, given browsing context browsingContext, URL url, sandboxing flag set sandboxFlags, and two origins invocationOrigin and activeDocumentNavigationOrigin:
>
> 1: If sandboxFlags has its sandboxed origin browsing context flag set, then return a new opaque origin.
>
> 2: If url is null, then return a new opaque origin.
>
> 3: If activeDocumentOrigin is not null, and url's scheme is "javascript", then return activeDocumentNavigationOrigin.
>
> 4: If invocationOrigin is non-null and url is about:blank, then return invocationOrigin.
>
> 5: If url is about:srcdoc, then return the origin of browsingContext's container document.
>
> 6: Return url's origin.
>
> [https://html.spec.whatwg.org/multipage/browsers.html#determining-the-origin](https://html.spec.whatwg.org/multipage/browsers.html#determining-the-origin)

以上が iframe における environment settings object の origin の決定方法でした。iframe の navigation が複雑なアルゴリズムを辿るためとても長くなってしまいましたが、origin の決定自体は基本的には **determine the origin** アルゴリズムに従うことが分かりました。

# Web Worker (Dedicated Worker / Shared Worker) の場合

Web Worker (Dedicated Worker もしくは Shared Worker) の origin の決め方は iframe に比べてアドホックに定義されています。Web Worker はそれを作る実行コンテキスト (親実行コンテキスト) と同じ origin を持つと決められているため、基本的には親の origin を引き継ぎます。

```new Worker()``` や ```new SharedWorker()``` を呼ぶと、**run a worker** アルゴリズムが実行されます。この step 8 で **set up a worker environment settings object** というアルゴリズムが呼ばれます。

> When a user agent is to **run a worker** for a script with Worker or SharedWorker object worker, URL url, environment settings object outside settings, MessagePort outside port, and a WorkerOptions dictionary options, it must run the following steps.
>
> 8: **Set up a worker environment settings object** with realm execution context and outside settings, and let inside settings be the result.
>
> [https://html.spec.whatwg.org/multipage/workers.html#run-a-worker](https://html.spec.whatwg.org/multipage/workers.html#run-a-worker)

**set up a worker environment settings object** は親実行コンテキストの environment settings object を outside settings 変数として受け取ります。Step 2 で outside settings の origin を inherited origin 変数に設定し、それを worker の environment settings object の origin として設定することで origin の継承を行っています。

> When the user agent is required to **set up a worker environment settings object**, given a JavaScript execution context execution context and **environment settings object outside settings**, it must run the following steps:
>
> 2: Let **inherited origin** be outside settings's origin.
>
> 6: Let settings object be a new environment settings object whose algorithms are defined as follows:
>
> - **The origin**: Return a unique opaque origin if worker global scope's url's scheme is "data", and inherited origin otherwise.
>
> [https://html.spec.whatwg.org/multipage/workers.html#set-up-a-worker-environment-settings-object](https://html.spec.whatwg.org/multipage/workers.html#set-up-a-worker-environment-settings-object)

興味深いのは worker global scope's url's schme が data scheme だった場合で、これはつまり worker script URL が Data URL だった場合ですが、そのときは origin の継承を行わずに opaque origin を設定します。

[Data URL](https://fetch.spec.whatwg.org/#data-urls) は ```data:[<mime type>][;base64],<base64 data>``` のような構造を持つ URL で、Base64 形式でエンコードしたデータを URL 内に埋め込むことができます。これに対するリクエストはネットワークに送られる代わりにブラウザ内で ```<base64 data>``` 部分をデコードして、あたかもネットワークから response が送られてきたかのように処理を行います。通常は画像データなどを Data URL に埋め込んで img タグに流し込むことでネットワークリクエストを行わずに画像を表示したりするために使われるのですが、HTML や JavaScript を埋め込んで iframe や worker のソースとして使用することもできます。このとき Data URL は origin 部を持たないため、opaque origin として扱われるようになっています。

ちなみに Worker コンストラクタや SharedWorker コンストラクタのアルゴリズムには下記の note が記載されており、ここでも Data URL は opaque origin になると明記されています。

> Any same-origin URL (including blob: URLs) can be used. **data: URLs can also be used, but they create a worker with an opaque origin.**
>
> [https://html.spec.whatwg.org/multipage/workers.html#dom-worker](https://html.spec.whatwg.org/multipage/workers.html#dom-worker)

ところで iframe の **determine the origin** アルゴリズムに Data URL を渡した場合はどうなるのでしょうか？**determine the origin** アルゴリズムには Data URL に関する記載はなく、step 6 の url's origin のケースで処理されます。

> To **determine the origin**, given browsing context browsingContext, URL url, sandboxing flag set sandboxFlags, and two origins invocationOrigin and activeDocumentNavigationOrigin:
>
> 6: Return **url's origin**.
>
> [https://html.spec.whatwg.org/multipage/browsers.html#determining-the-origin](https://html.spec.whatwg.org/multipage/browsers.html#determining-the-origin)

**url's origin** は次のように定義されています。ここでも Data URL に関する条件分岐がないため **Otherwise** 節に落ちて、opaque origin を返すことが分かります。要するに Data URL によって作られた iframe の environment settings object も opaque origin を持つようになります。

> A URL’s origin is the origin returned by running these steps, switching on URL’s scheme:
>
> **"blob"**:
> - If URL’s blob URL entry is non-null, then return URL’s blob URL entry’s environment’s origin.
> - Let url be the result of parsing URL’s path[0].
> - Return a new opaque origin, if url is failure, and url’s origin otherwise.
>
> **"ftp", "http", "https", "ws", "wss"**:
> - Return a tuple consisting of URL’s scheme, URL’s host, URL’s port, and null.
>
> **"file"**:
> - Unfortunate as it is, this is left as an exercise to the reader. When in doubt, return a new opaque origin.
>
> **Otherwise**:
> - Return a new opaque origin.
>
> [https://url.spec.whatwg.org/#concept-url-origin](https://url.spec.whatwg.org/#concept-url-origin)

# Service Worker の場合

Service Worker の origin は [Service Worker standard](https://w3c.github.io/ServiceWorker/) 内の **[Run Service Worker](https://w3c.github.io/ServiceWorker/#run-service-worker-algorithm)** アルゴリズムの中で定義されています。これによると Service Worker の environment settings object は registering service worker client's origin を origin として持つとされています。これは要するに Service Worker の登録を行う API である ```navigator.serviceWorker.register()``` を呼んだ実行コンテキストの origin を引き継ぐことを意味します。

> 7.4: Let settingsObject be a new environment settings object whose algorithms are defined as follows:
>
> **The origin**: Return its **registering service worker client's origin**.
>
> [https://w3c.github.io/ServiceWorker/#run-service-worker-algorithm](https://w3c.github.io/ServiceWorker/#run-service-worker-algorithm)

Service Worker はその性質上 Same Origin Policy を厳しく要求しており、Data URL のような opaque origin として解決される script URL を使って Service Worker の登録を行うことはできず、また opaque origin を持つ実行コンテキスト内 (e.g., Data URL iframe や Sandboxed iframe) で ```register()``` を呼ぶこともできないため、registering service worker client's origin が必ず non-opaque origin になることが保証されます。

# Worklet の場合

Worklet は Web Worker を軽量化して特定の用途で使いやすくした実行コンテキストです。詳しくは「[JavaScript のスレッド並列実行環境 - 6. Worklet](/2017/12/10/javascript-parallel-processing#6-worklet)」を読んでください。Worklet は Web Worker 以上にアドホックな仕様になっており、親実行コンテキストの origin や worklet script URL に関わらず常に opaque origin が設定されます。

この挙動は [Worklet standard](https://drafts.css-houdini.org/worklets/) の **[set up a worklet environment settings object](https://drafts.css-houdini.org/worklets/#set-up-a-worklet-environment-settings-object)** アルゴリズムで定義されています。

> 3: Let **origin** be a unique opaque origin.
>
> 9: Let settingsObject be a new environment settings object whose algorithms are defined as follows:
> - **The origin**: Return **origin**.
>
> [https://drafts.css-houdini.org/worklets/#script-settings-for-worklets](https://drafts.css-houdini.org/worklets/#script-settings-for-worklets)

Worklet はその上で使用できる API を意図的に厳しく制限しており、その一環で environment settings object に opaque origin が設定されています。詳しくは Worklet standard の [1.1. Motivations](https://drafts.css-houdini.org/worklets/#motivations) セクションを読んでください。

# まとめと雑感

今回は各種 environment settings object の origin の決め方を見てきました。まとめると次のようになります。

- iframe
  - **determine the origin** アルゴリズムによって origin が決定される。
  - iframe はまず about:blank の読み込みが行われ、その後に src attribute に対して navigate が行われる。about:blank 時は creator origin が environment settings object の origin になる。 
  - Data URL の場合は opaque origin になる。
- Web Worker (Dedicated Worker / Shared Worker)
  - Web Worker は基本的に親の environment settings object の origin を継承する。
  - Data URL で生成された場合は opaque origin になる。
- Service Worker
  - ```register()``` を呼んだ environment settings object の origin を引き継ぐ。
  - Data URL を service worker script として登録できず、sandboxed iframe 内での ```register()``` も禁止されているため、opaque origin にはならない。
- Worklet
  - 親の environment settings object や worklet script URL に関わらず常に opaque origin になる。

iframe がまず about:blank をロードすることは知っていましたが、仕様上の初期化処理を読んだのは今回が初めてでとても勉強になりました。Worker や Worklet の origin 決定アルゴリズムについては Chromium における実装者ということもあって既に知っていましたが、仕様を改めて読み通してその理解を明文化することができて良かったです。後世のメンテナの役に立てば幸いです :)

# 注釈