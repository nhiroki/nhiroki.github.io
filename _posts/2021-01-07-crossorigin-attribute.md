---
layout: post
title: "crossorigin 属性の仕様を読み解く"
date: 2021-01-07 00:00:00 +09:00
tags: web
image: /images/profile.png
---

本記事では HTML タグに指定可能な crossorigin 属性について仕様を参照しながら詳しく解説します。crossorigin 属性は複数の意味を表しており、またそれを指定するタグの他の属性値によって振る舞いが変わってしまうことから、その挙動を正確に理解するのがなかなか難しい属性です。

HTML 仕様は日々進化しています。本記事で説明している内容は記事執筆時点のものであり、閲覧時点では古くなっている可能性があります。正確な情報を知りたい方は必ず最新の仕様を確認するようお願いします。

要点だけを知りたい方は最後の「まとめ」を読んでください。

# 目次

- [crossorigin 属性はどこで使われている？](#crossorigin-属性はどこで使われている)
- [crossorigin 属性は何を意味するのか？](#crossorigin-属性は何を意味するのか)
  - request mode
  - credentials mode
  - crossorigin 属性の意味のまとめ
- [crossorigin 属性の振る舞い (img タグ)](#crossorigin-属性の振る舞い-img-タグ)
  - no-cors mode と opaque filtered response
  - cross-origin リクエストによる画像読み込みの仕様
- [crossorigin 属性の振る舞い (script タグ)](#crossorigin-属性の振る舞い-script-タグ)
  - classic script
  - module script
- [crossorigin 属性の振る舞い (link preconnect タグ)](#crossorigin-属性の振る舞い-link-preconnect-タグ)
  - preconnect とは？
  - コネクションプール
  - preconnect における crossorigin 属性
- [非宣言的な API における request mode と credentials mode](#非宣言的な-api-における-request-mode-と-credentials-mode)
- [まとめ](#まとめ)

# crossorigin 属性はどこで使われている？

[現在の HTML 仕様](https://html.spec.whatwg.org/multipage/indices.html#attributes-3:attr-link-crossorigin)では crossorigin 属性は audio, img, link, script, video タグで指定可能だと定義されています。

![crossorigin attribute availiability](/images/crossorigin-attribute-availability.webp)

例えば、次のように指定できます。

```html
<img src="https://cross-origin.example.com/image.jpg" crossorigin>
<script src="https://cross-origin.example.com/script.js" crossorigin></script>
```

# crossorigin 属性は何を意味するのか？

crossorigin 属性は [CORS settings attributes](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#cors-settings-attributes) として定義されています。その名の通り、CORS (Cross Origin Resource Sharing) の振る舞いに関する設定です。CORS は複雑な機能でまじめに解説すると記事がとんでもなく長くなってしまうため、その役目は他の記事に譲ります。これ以降は CORS が何なのか理解している前提で話を進めます。

## request mode

さて、次のコードスニペットを見てください。

```html
<!-- (1) crossorigin 属性が未指定 => no-cors mode -->
<img src="https://cross-origin.example.com/image.jpg">
<!-- (2) crossorigin の属性値が未指定 => cors mode -->
<img src="https://cross-origin.example.com/image.jpg" crossorigin>
```

**原則として**、

- (1) のように crossorigin 属性を指定しないと no-cors モード
- (2) のように crossorigin 属性を指定すると cors モード

・・・でそのタグのリクエストが処理されます。この「モード」は [request mode として Fetch 仕様で定義](https://fetch.spec.whatwg.org/#concept-request-mode)されています。Request mode には same-origin や navigate といった cors & no-cors 以外の値もありますが、以降の話では出てこないので無視してもらって大丈夫です。

ところで、**上の方で「原則として」と前置きしたのには理由があります。それについては後ほど説明するので、ここだけを読んで「cross-origin リクエストを投げたいときに crossorigin 属性を指定するのね」と早合点しないでください！**

## credentials mode

crossorigin 属性は値として `""` (空文字列), `anonymous`, `use-credentials` を取ります。(3) のように属性値が空文字列の場合は `anonymous` として扱われます。また先程のコードスニペットの (2) のように属性値を指定しなかった場合も `anonymous` として扱われます。

```html
<!-- (2) crossorigin の属性値が未指定 => anonymous -->
<img src="https://cross-origin.example.com/image.jpg" crossorigin>
<!-- (3) crossorigin の属性値が空文字列 => anonymous -->
<img src="https://cross-origin.example.com/image.jpg" crossorigin="">
<!-- (4) crossorigin が anonymous -->
<img src="https://cross-origin.example.com/image.jpg" crossorigin="anonymous">
<!-- (5) crossorigin が use-credentials -->
<img src="https://cross-origin.example.com/image.jpg" crossorigin="use-credentials">
```

これら属性値は仕様では State と呼ばれ、CORS モードでリクエストを投げる時の credentials mode を決めます。State は No CORS, Anonymous, Use Credentials の 3 つで、crossorigin 属性を指定しない (1) のときは No CORS になります。

|属性|State|
|(1) 未指定 (デフォルト)|No CORS|
|(2) crossorigin|Anonymous|
|(3) crossorigin=""|Anonymous|
|(4) crossorigin=anonymous|Anonoymous|
|(5) crossorigin=use-credentials|Use Credentials|

credentials は cookies, TLS client certificate, HTTP authenticaion といった認証に関わる情報のことです。[Fetch 仕様](https://fetch.spec.whatwg.org/#credentials)で定義されています。[credentials mode](https://fetch.spec.whatwg.org/#concept-request-credentials-mode) はリクエストとともに credentials を送るかどうかを決めるもので、omit, same-origin, include の三種類があります。omit は常に credentials を送らず、same-origin は same-origin リクエストの時にだけ送信し、include は常に送信します。

State が No CORS である (1) や Anonymous である (2,3,4) では credentials mode が same-origin に、(5) のように Use Credentials だと credentials mode が include になります。この決定ルーチンは [CORS settings attribute credentials mode](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#cors-settings-attribute-credentials-mode) として定義されています。

![crossorigin attribute credentials mode](/images/crossorigin-attribute-credentials-mode-routine.webp)

この CORS settings attribute credentials mode は決定ルーチンの一つに過ぎず、例えばリクエストを生成する [creating a potential-CORS request](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#create-a-potential-cors-request) アルゴリズムでは独自のルーチンで処理しており、No CORS ステートの credentials mode を include としています。ここは混乱ポイントですね。実は新しめの機能 (例えば module script) は cors モードによってのみリクエストができるようになっており、その場合は crossorigin 属性は credentials mode にのみ影響を及ぼします。新しい機能ではセキュリティやプライバシーの観点から credentials mode のデフォルトが include ではなく same-origin になっており (仕様議論は[この辺](https://github.com/whatwg/html/issues/2557))、その決定ロジックを共通化したルーチンが CORS settings attribute credentials mode なのです。昔からある機能の場合は互換性維持のため creating a potential-CORS request のような個別のルーチンで決めています。この辺は後ほど個別に仕様で確認していきます。

## crossorigin 属性の意味のまとめ

以上をまとめると次の表のようになります。

|属性|State|Request mode|Credentials mode|
|(1) 未指定 (デフォルト)|No CORS|no-cors|same-origin (or include)|
|(2) crossorigin|Anonymous|cors|same-origin|
|(3) crossorigin=""|Anonymous|cors|same-origin|
|(4) crossorigin=anonymous|Anonymous|cors|same-origin|
|(5) crossorigin=use-credentials|Use Credentials|cors|include|

このように **crossorigin 属性は request mode と credentials mode の二つを規定していて、credentials mode の決定方法はいくつかある**ことを覚えておいてください。

ここまでで各タグにおける crossorigin 属性の振る舞いについて説明する準備が整いました。crossorigin 属性をサポートした全てのタグについて説明するのは大変なので、以降では img, script, link タグについて紹介します。

# crossorigin 属性の振る舞い (img タグ)

img タグにおける crossorigin 属性の挙動は素直で分かりやすいので最初に説明します。img タグはデフォルトで no-cors モードで cross-origin の画像を取得します。単に cross-origin の画像を表示するだけなら問題ないですが、JavaScript から画像を処理しようとすると困ったことになります。

## no-cors mode と opaque filtered response

[request mode](https://fetch.spec.whatwg.org/#concept-request-mode) の仕様によると no-cors モードで取得した画像は opaque filtered response として扱われます。

![crossorigin attribute no-cors request](/images/crossorigin-attribute-no-cors-request.webp)

[opaque filtered response](https://fetch.spec.whatwg.org/#concept-filtered-response-opaque) ではそのヘッダーやボディを JavaScript から参照することができず、リクエストの成否すら知ることはできません。

![crossorigin attribute opaque filtered response](/images/crossorigin-attribute-opaque-filtered-response.webp)

セキュリティやプライバシー保護のためにこのような挙動になっているのですが、詳しくは CORS の解説を読んでください。この挙動は cross-origin の画像を Canvas API で処理したい場合などに問題となります。これを解決するには crossorigin 属性を指定して cors モードで画像を読み込む必要があります。

## cross-origin リクエストによる画像読み込みの仕様

それでは仕様を確認してみましょう。img タグの読み込みは [update the image data](https://html.spec.whatwg.org/multipage/images.html#update-the-image-data) アルゴリズムで処理されます。Step 17 で img タグの crossorigin 属性の値を creating a potential-CORS request アルゴリズムに渡しています。

> When the user agent is to update the image data of an img element, optionally with the restart animations flag set, it must run the following steps:
>
> - 17\. Let request be the result of creating a potential-CORS request given urlString, "image", and the current state of the element's crossorigin content attribute.

[creating a potential-CORS request](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#create-a-potential-cors-request) アルゴリズムは request オブジェクトを作り、request mode と credentials mode の設定を行っています。

まず step 1 で corsAttributeState (crossorigin 属性の State) を確認し、No CORS ステートなら no-cors モードを、Anonymous もしくは Use Credentials ステートなら cors モードを設定します。same-origin fallback flag は今回指定されていないため step 2 は飛ばします。次に step 3 で credentialsMode を include で初期化し、step 4 で corsAttributeState が Anonymous なら same-origin で上書きします。

> To create a potential-CORS request, given a url, destination, corsAttributeState, and an optional same-origin fallback flag, run these steps:
>
> 1. Let mode be "no-cors" if corsAttributeState is No CORS, and "cors" otherwise.
> 2. If same-origin fallback flag is set and mode is "no-cors", set mode to "same-origin".
> 3. Let credentialsMode be "include".
> 4. If corsAttributeState is Anonymous, set credentialsMode to "same-origin".
> 5. Let request be a new request whose url is url, destination is destination, mode is mode, credentials mode is credentialsMode, and whose use-URL-credentials flag is set.

以上により「anonymous なら same-origin、use-credentials なら include」という状態が成立します。また、credentialsMode が include で初期化されていることから No CORS ステートの場合は credentials mode が include になることも確認できました。

img タグによる cross-origin リクエストは昔からある機能でその互換性維持 (デフォルトの credentials mode が include) のために [CORS settings attribute credentials mode](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#cors-settings-attribute-credentials-mode) ではなく creating a potential-CORS request の独自ルーチンで credentials mode を決定しているのでした。

# crossorigin 属性の振る舞い (script タグ)

script には classic script と module script の二種類があります。この使い分けについては以前記事を書いたのでそちらを見てください。

- [JavaScript のスクリプトインポートを正しく使い分けようという話](/2018/09/07/javascript-import)

仕様を追いやすくするため、classic script と module script の挙動を先にまとめて示しておきます。ポイントは classic script で No CORS ステートのときに credentials mode が include になること、module script では State によらず request mode が常に cors になることですね。

|Script type|属性|State|Request mode|Credentials mode|
|classic|未指定|No CORS|no-cors|include|
||crossorigin|Anonymous|cors|same-origin|
||crossorigin=""|Anonymous|cors|same-origin|
||crossorigin=anonymous|Anonymous|cors|same-origin|
||crossorigin=use-credentials|Use Credentials|cors|include|
|module|未指定|No CORS|cors|same-origin|
||crossorigin|Anonymous|cors|same-origin|
||crossorigin=""|Anonymous|cors|same-origin|
||crossorigin=anonymous|Anonymous|cors|same-origin|
||crossorigin=use-credentials|Use Credentials|cors|include|

## classic script

まずは classic script の場合です。classic script はデフォルトで no-cors モードでリクエストを投げますが、crossorigin 属性を指定することで cors モードでリクエストを投げることができます。img タグと同じく、crossorigin 属性の State が Anonymous (2,3,4) の場合は credentials mode が same-origin になり、Use Credentials (5) の場合は include になります。また No CORS (1) の場合は include になり、これはさきほど紹介した [CORS settings attribute credentials mode](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#cors-settings-attribute-credentials-mode) による決定ルーチンとは異なる部分です。これも互換性維持のためですね。これらを仕様で確認してみましょう。

script タグは HTML 仕様の [prepare a script](https://html.spec.whatwg.org/multipage/scripting.html#prepare-a-script) アルゴリズムで処理されます。Step 18 で crossorigin 属性の値を classic script CORS setting に代入し、それを Step 26.6 で fetch a classic script アルゴリズムに渡しています。

> To prepare a script, the user agent must act as follows:
>
> - 18\. Let classic script CORS setting be the current state of the element's crossorigin content attribute.
> - 26\. If the element has a src content attribute, then:
>   - 6\. Switch on the script's type:
>     - "classic": Fetch a classic script given url, settings object, options, classic script CORS setting, and encoding.

[fetch a classic script](https://html.spec.whatwg.org/multipage/webappapis.html#fetch-a-classic-script) アルゴリズムは creating a potential-CORS request アルゴリズムを呼び出して request オブジェクトを作成し、それを使ってフェッチを行います。

> To fetch a classic script given a url, a settings object, some options, a CORS setting, and a character encoding, run these steps. The algorithm will asynchronously complete with either null (on failure) or a new classic script (on success).
>
> 1. Let request be the result of creating a potential-CORS request given url, "script", and CORS setting.

[creating a potential-CORS request](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#create-a-potential-cors-request) アルゴリズムの挙動は img タグの場合と同じなので割愛します。以上により classic script の credentials mode の決定方法を確認できました。

さて classic script 特有の挙動として、[muted errors](https://html.spec.whatwg.org/multipage/webappapis.html#muted-errors) があります。これは no-cors モードでフェッチされたスクリプト (opaque filtered response) のエラーを JavaScript に通知しないようにするためのフラグです。

![crossorigin attribute muted errors](/images/crossorigin-attribute-muted-errors.webp)

opaque filtered response によってヘッダーやボディの中身を不可視にしても、エラー内容を見れてしまうとそこからレスポンスの中身が推察できてしまうため、このような振る舞いが定義されています。このフラグは[レスポンスが opaque (CORS-cross-origin) の時にセット](https://html.spec.whatwg.org/multipage/webappapis.html#fetch-a-classic-script)され、スクリプト実行時にチェックされます。

> To fetch a classic script given a url, a settings object, some options, a CORS setting, and a character encoding, run these steps. The algorithm will asynchronously complete with either null (on failure) or a new classic script (on success).
>
> - 9\. Let muted errors be true if response was CORS-cross-origin, and false otherwise.

ちなみに module script は常に cors モードで処理されるため、このフラグは使われません。

## module script

次に module script の場合です。**module script は crossorigin 属性の有無に関わらず常に cors モードでリクエストを処理します。よって、crossorigin 属性は credentials mode にのみ影響を与えます。**credentials mode は No CORS ステート (1) や Anonymous ステート (2,3,4) の場合は same-origin、Use Credentials ステート (5) の場合は include になります。No CORS ステートの場合に same-origin になるのが classic script と違いますね。module script は新しめの機能なので、credentials mode のデフォルトが include ではなく same-origin となっています。これらも仕様で確認してみましょう。

classic script と同じく、module script は HTML 仕様の [prepare a script](https://html.spec.whatwg.org/multipage/scripting.html#prepare-a-script) アルゴリズムで処理されます。Step 19 で crossorigin 属性の値から credentials mode を決定し、それを module script credentials mode に代入します。この決定には CORS settings attribute credentials mode ルーチンが使われます。No CORS ステートや Anonymous ステートの場合は same-origin を、Use Credentials ステートの場合は include を返します。Step 24 では script fetch options の credentials mode フィールドに先程決定した module script credentials mode をセットし、step 26.6 でその options を fetch an external module script graph アルゴリズムに渡します。

> To prepare a script, the user agent must act as follows:
>
> - 19\. Let module script credentials mode be the CORS settings attribute credentials mode for the element's crossorigin content attribute.
> - 24\. Let options be a script fetch options whose cryptographic nonce is cryptographic nonce, integrity metadata is integrity metadata, parser metadata is parser metadata, credentials mode is module script credentials mode, and referrer policy is referrer policy.
> - 26\. If the element has a src content attribute, then:
>   - 6\. Switch on the script's type:
>     - "module": Fetch an external module script graph given url, settings object, and options.

[fetch an external module script graph](https://html.spec.whatwg.org/multipage/webappapis.html#fetch-a-module-script-tree) アルゴリズムは渡された options を fetch a single module script アルゴリズムに渡します。

> To fetch an external module script graph given a url, a settings object, and some options, run these steps. The algorithm will asynchronously complete with either null (on failure) or a module script (on success).
>
> 1. Fetch a single module script given url, settings object, "script", options, settings object, "client", and with the top-level module fetch flag set. If the caller of this algorithm specified custom perform the fetch steps, pass those along as well. Wait until the algorithm asynchronously completes with result.

[fetch a single module script](https://html.spec.whatwg.org/multipage/webappapis.html#fetch-a-single-module-script) アルゴリズムは step 6 で cors のリクエストを作り、step 8 でそれを options でセットアップしてリクエストを投げます。

> To fetch a single module script, given a url, a fetch client settings object, a destination, some options, a module map settings object, a referrer, and a top-level module fetch flag, run these steps. The algorithm will asynchronously complete with either null (on failure) or a module script (on success).
>
> - 6\. Let request be a new request whose url is url, destination is destination, mode is "cors", referrer is referrer, and client is fetch client settings object.
> - 8\. Set up the module script request given request and options.

以上により、module script のフェッチは request mode が cors で、credentials mode は CORS settings attribute credentials mode で決められた値を使って行われることが分かりました。

# crossorigin 属性の振る舞い (link preconnect タグ)

link タグは rel 属性で様々なリソースを読み込むことができます。基本的にはさきほど紹介した [creating a potential-CORS request](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#create-a-potential-cors-request) アルゴリズムで処理されるため、img タグや classic script の場合と同じ挙動をします。しかし、この原則とは違う挙動をするケースもあります。今回はその一例として preconnect について説明します。

## preconnect とは？

[preconnect](https://w3c.github.io/resource-hints/#preconnect) は TCP に関するヒントを与えるウェブブラウザ API です。[Resource Hints 仕様](https://w3c.github.io/resource-hints/)で定義されています。後ほどアクセスするドメインを link タグや HTTP レスポンスの Link ヘッダで指定することで、そのドメインへの TCP コネクションをあらかじめ確立するようブラウザに知らせることができます。

```html
<!-- (1) crossorigin 属性が未指定 -->
<link rel="preconnect" href="https://cross-origin.example.com/">
<!-- (2) crossorigin の属性値が未指定 -->
<link rel="preconnect" href="https://cross-origin.example.com/" crossorigin>
<!-- (3) crossorigin の属性値が空文字列 -->
<link rel="preconnect" href="https://cross-origin.example.com/" crossorigin="">
<!-- (4) crossorigin が anonymous -->
<link rel="preconnect" href="https://cross-origin.example.com/" crossorigin="anonymous">
<!-- (5) crossorigin が use-credentials -->
<link rel="preconnect" href="https://cross-origin.example.com/" crossorigin="use-credentials">
```

さて、preconnect は TCP コネクションに関する API であり、一見すると request mode や credentials mode とは無関係に思えます。いったい crossorigin 属性によって何が変わるのでしょうか？これを理解するにはブラウザがどのようにコネクションを管理しているか知る必要があります。

## コネクションプール

[コネクションの管理方法は「コネクションプール」として Fetch 仕様で定義](https://fetch.spec.whatwg.org/#concept-connection-pool)されています。

> A user agent has an associated connection pool. A connection pool consists of zero or more connections. Each connection is identified by a key (a network partition key), an origin (an origin), and credentials (a boolean).

コネクションプールは network partition key, origin, credentials (boolean) をキーとしてコネクションを管理します。

- network partition key はコネクションや HTTP キャッシュをセキュリティやプライバシー管理のために分離する仕組みで、接続元の top frame site と current frame site のペアで構成されています。これはとても重要な機能なんですが本記事の趣旨からは外れてしまうため、詳しいことは「[Gaining security and privacy by partitioning the cache](https://developers.google.com/web/updates/2020/10/http-cache-partitioning)」を読んでください。
- origin は接続先のオリジンです。
- リクエストが credentials を含む場合 (credentials requests) は true になり、リクエストが credentials を含まない場合 (uncredentialed requests) は false になります。つまり credentials の有無でコネクションが分離されます。

crossorigin 属性の話をする上でこの credentials の有無によるコネクション分離が重要になってきます。

## preconnect における crossorigin 属性

話を preconnect に戻します。既にお気づきかもしれませんが、preconnect では**これから事前確立するコネクションが credentialed リクエストのためのものなのか、それとも uncredentialed リクエストのためのものなのかを crossorigin 属性によって指定します。**crossorigin 属性は request mode と credentials mode を規定するという話をしましたが、このうちの credentials mode だけが preconnect で意味をなすわけです。

それでは仕様からその挙動を読み解いていきましょう。crossorigin 属性の値からコネクションの credentials キーへの変換は、Resource Hints 仕様の [initiate a preconnect](https://w3c.github.io/resource-hints/#dfn-initiate-a-preconnect) アルゴリズムで定義されています。

まず step 2 で preconnect 先の origin を、step 3 で crossorigin 属性の state を取得します。step 4 では credentials (boolean) を true に初期化します。step 5 では state が Anonymous で、かつ現在のドキュメントの origin が preconnect 先の origin と異なる (つまり cross-origin preconnect の) 場合 、これは uncredentialed (anonymous) cross-origin requests になるので、credentials を false にします。

> To initiate a preconnect, the user agent MUST run these steps:
>
> 1. Resolve the URL given by the href attribute.
> 2. Let origin be preconnect URL's origin.
> 3. Let corsAttributeState be the current state of the element's crossorigin content attribute.
> 4. Let credentials be a boolean value set to true.
> 5. If corsAttributeState is Anonymous and origin is not equal to current Document's origin, set credentials to false.
> 6. Attempt to obtain connection with origin and credentials.

initiate a preconnect アルゴリズムを全てのケースで計算すると次の表のようになります。

|Target origin|属性|State|Credentialed?|
|same|未指定|No CORS|true|
||crossorigin|Anonymous|true|
||crossorigin=""|Anonymous|true|
||crossorigin=anonymous|Anonymous|true|
||crossorigin=use-credentials|Use Credentials|true|
|cross|未指定|No CORS|true|
||crossorigin|Anonymous|false|
||crossorigin=""|Anonymous|false|
||crossorigin=anonymous|Anonymous|false|
||crossorigin=use-credentials|Use Credentials|true|

実用上は credentialed リクエストのための preconnect では crossorigin を指定せず、uncredentialed cross-origin リクエストのための preconnect では crossorigin だけを指定する感じになるでしょう。もし同じオリジンに対して credentialed リクエストと uncredentialed リクエストを送る場合は preconnect を二回指定します。

```html
<!-- preconnect for credentialed cross-origin requests -->
<link rel="preconnect" href="https://cross-origin.example.com/">
<!-- preconnect for uncredentialed cross-origin requests -->
<link rel="preconnect" href="https://cross-origin.example.com/" crossorigin>
```

リクエストが credentialed かどうかはそれを発行する API によって変わります。例えば [Fetch API ではオプションで credentials を指定](https://developer.mozilla.org/ja/docs/Web/API/Fetch_API/Using_Fetch#Sending_a_request_with_credentials_included)できます。また [CSS Fonts 仕様ではフォントのフェッチは Anonymous モード (uncredentialed) で行うよう定義](https://drafts.csswg.org/css-fonts/#font-fetching-requirements)されています。つまり CSS Fonts に対して preconnect を使いたい場合は crossorigin を指定する必要があります。

> When fetching, user agents must use "Anonymous" mode, set the referrer source to the stylesheet’s URL and set the origin to the URL of the containing document.

このように crossorigin 属性によって所望のリクエストが事前確立したコネクションを使えるかどうかが決まるので、闇雲に指定する前にしっかり確認しましょう。

まとめると、crossorigin 属性はコネクションに流すリクエストの credentials に関するものであること、リクエストが credentialed かどうかによってコネクションが使い分けられること、credentialed かどうかは API によって変わることを覚えておいてください。

# 非宣言的な API における request mode と credentials mode

crossorigin 属性はタグのような宣言的 (declarative) な要素に対して request mode と credentials mode を指定するものでした。ではタグではない命令的 (imperative) な API ではどのように指定するのでしょうか？

preconnect の項でも少し紹介しましたが、[Fetch API ではオプションで request mode や credentials mode を指定](https://developer.mozilla.org/ja/docs/Web/API/Fetch_API/Using_Fetch#supplying_request_options)することができます。

```js
const response = await fetch(url, {
  method: 'GET',
  mode: 'cors',
  credentials: 'same-origin',
  ...
});
```

Web Workers ではコンストラクタの引数で credentials mode を指定できます。Web Workers は Same Origin Policy に従うため request mode は常に same-origin になり、また credentials mode の指定は module workers でのみ意味を持ちます (classic workers では常に same-origin credentials mode になります)。

```js
// Module worker
const module_worker = new Worker('module-worker.js', {
  type: 'module',
  credentials: 'omit'
});

// Classic worker (credentials パラメータは無視される)
const classic_worker = new Worker('classic-worker.js', {
  credentials: 'omit'
});
```

# まとめ

本記事では crossorigin 属性の挙動について仕様を参照しながら詳しく解説しました。

- crossorigin 属性は request mode と credentials mode の振る舞いを規定している。
- module script のような新しい機能では request mode が常に cors になるため、crossorigin 属性によって credentials mode だけが変化する。またデフォルトの credentials mode が include から same-origin になっている。
- 同様に preconnect のように request mode が意味を持たない API では credentials mode が重要となる。
- Fetch API のような命令的な API では request mode と credentials mode をオプションで指定する。