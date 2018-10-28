---
layout: post
title: "Chrome 69 で Web Worker から Web Worker を作れるようになった話"
date: 2018-10-29 00:00:00 +09:00
tags: web
image: /images/profile.png
---

少し前のバージョンですが、Chrome 69 より Web Worker から Web Worker を作れるようになりました。この機能は Nested Workers とも呼ばれています。

- [Nested Dedicated Workers - Chrome Platform Status](https://www.chromestatus.com/feature/6080438103703552)
- [Intent to Implement and Ship: Nested dedicated workers - blink-dev](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/KZx0i3e5nZM)

# 使用方法とユースケース

使い方は Window 上で Worker を作る場合と同じです。次のコードでは、Window から Parent Worker、Parent Worker から Child Worker を作り、```postMessage()``` で Child Worker から Window までメッセージを送信します。

```js
// index.html
const worker = new Worker('parent-worker.js');
const msgEvent = await new Promise(resolve => parent_worker.onmessage = resolve);
console.log(msgEvent.data);  // 'Hello from a nested worker!'

// parent-worker.js
const child_worker = new Worker('child-worker.js');
const msgEvent = await new Promise(resolve => child_worker.onmessage = resolve);
postMessage(msgEvent.data);

// child-worker.js
postMessage('Hello from a nested worker!');
```

Parent Worker を terminate すると、Child Worker 達も一緒に terminate されます。

```js
// index.html
const worker = new Worker('parent-worker.js');
worker.terminate();  // The parent and its all desendant workers will be terminated.

// parent-worker.js
const child_worker1 = new Worker('child-worker.js');
const child_worker2 = new Worker('child-worker.js');

// child-worker.js
const grandchild_worker = new Worker('grandchild-worker.js');
```

現在 Nested Workers が可能なのは Dedicated Worker から Dedicated Worker を作る場合のみで、Shared Worker や Service Worker から Dedicated Worker を作ったり、逆に Dedicated Worker から Shared Worker や Service Worker を作ることはできません[^worker-creation]。各 Worker の違いについては「[JavaScript のスレッド並列実行環境 - 2. Web Worker - Worker の種類とプロセスモデル](/2017/12/10/javascript-parallel-processing#2-web-worker)」を参照してください。

[^worker-creation]: Dedicated Worker から Shared Worker の生成や、Shared Worker から Dedicated Worker の生成は HTML 仕様では定義されていますが Chrome では実装されていません。同様に Dedicated Worker から Service Worker の登録も Service Worker 仕様では定義されていますが Chrome では実装されていません ([crbug.com/371690](https://bugs.chromium.org/p/chromium/issues/detail?id=371690))。一方、Service Worker から Dedicated Worker や Shared Worker の生成は仕様の議論が行われている段階です (see: "[Allow workers & shared workers to be created within a service worker - whatwg/html](https://github.com/whatwg/html/issues/411)")。

![nested workers creation](/images/nested-workers-creation.png)

Nested Workers が実装される前は、Dedicated Worker から新しく Dedicated Worker を起動したい場合は Window を経由して作る必要がありました。これは Window との ```postMessage()``` の分余計なオーバヘッドがかかり、かつ Worker 間で直接メッセージングをするには別途 ```MessageChannel``` による通信路を作る必要があるなど、使い勝手が良いものではありませんでした (see: "[JavaScript のスレッド並列実行環境 - 3. スレッド間 (実行コンテキスト間) 通信](/2017/12/10/javascript-parallel-processing#3-%E3%82%B9%E3%83%AC%E3%83%83%E3%83%89%E9%96%93-%E5%AE%9F%E8%A1%8C%E3%82%B3%E3%83%B3%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88%E9%96%93-%E9%80%9A%E4%BF%A1)")。

Nested Workers によりこれらの問題が解決され、Dedicated Worker 内で Dedicated Worker を使うライブラリを埋め込みやすくなるなど、ユースケースも広がります。

![nested workers creation via window](/images/nested-workers-via-window.png)

Web Worker を使う際の一般的な注意点として、Worker を作りすぎないように気をつける必要があります。Worker を作るとネイティブスレッドが一つ作られ、さらにスレッドごとに V8 のインスタンスが生成されるため、むやみに ```new Worker()``` すべきではありません。最悪の場合、メモリ使用量の上限に引っかかって Out-Of-Memory (OOM) が発生し、sad タブ状態になります。スレッドプールを作ったり、不要な Worker は停止するなど、控えめに利用することをおすすめします (see: "[JavaScript のスレッド並列実行環境 - 5. Worker のコスト](/2017/12/10/javascript-parallel-processing#5-worker-%E3%81%AE%E3%82%B3%E3%82%B9%E3%83%88)")。

# 実装の裏話

実は Nested Workers は新しい仕様ではありません。Chromium のバグトラッカーには 8 年以上前からバグ ([crbug.com/31666](https://bugs.chromium.org/p/chromium/issues/detail?id=31666)) が登録されており、Firefox では以前から利用可能でした。バグ番号が五桁であるところに歴史を感じます[^bug-number]。

[^bug-number]: 現在はもうすぐ六桁目を使い切りそうなところです。

以前から Nested Workers を実装したいという気持ちはあったのですが、Blink には歴史的な経緯により「ワーカースレッドは常にメインスレッドから起動される」という実装制約があり、そのアーキテクチャの複雑さから長年実現できていませんでした。単純にその制約を外せばいいかというとそんなことはなく、ワーカスレッドの起動や停止に合わせて協調するその他のコンポーネントがちゃんと動くようにする必要があります。

私は 2017 年に Worklet の実装を行いつつ、Nested Workers を目指した Worker インフラの再設計を進めていました (see: "[Worker infrastructure roadmap 2017 - Google Docs](https://docs.google.com/document/d/1RIMCo_xejzvm0BlJdhAg2nfpdOE7xy_ijHVXk6kPHws/edit?usp=sharing)")。しかし、その新設計の実装が一段落ついた段階でも Nested Workers が本当に実現可能か不透明な部分が多く、二の足を踏んでいる状態でした。

一方その頃、幸運にも一人のスーパーエンジニアが Worker チームに加わり、彼の抜群の実装力によってあっさり実現できてしまいました。その結果 Chrome 69 で無事ローンチすることができました。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">同僚のパッチ書く速度が異次元過ぎるし、既存実装がどんなに難解であってもそれを突き抜けてくる突破力のようなものを感じる</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/989700852666417157?ref_src=twsrc%5Etfw">2018年4月27日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# まとめ

Chrome 69 より Dedicated Worker から Dedicated Worker が作れるようになり、Web Worker のユースケースが広がりました。もし Nested Workers を使っていておかしな挙動を見つけた場合は、是非 [Chromium のバグトラッカー](https://crbug.com/)へ報告してください。

# 注釈