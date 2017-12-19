---
layout: post
title:  "JavaScript のスレッド並列実行環境"
date: 2017-12-10 00:00:00 +09:00
tags: web
image: /images/javascript-parallel-processing-opg-image.png
---

これは [Chromium Browser アドベントカレンダー](https://qiita.com/advent-calendar/2017/chromium)の十日目の記事です。本記事では Chromium における JavaScript のスレッド並列実行環境について仕様・実装・API の面から包括的に紹介します。ブラウザの内部実装に興味がある人を対象に、各機能の使い方ではなく実行モデルに焦点を当てて説明しているため、難易度は高いです。使い方を知りたい人は [MDN](https://developer.mozilla.org/ja/) などの記事を読んでください。この記事をきっかけに実装解読に挑戦してみる人が一人でも増えると幸いです。

本記事を書くにあたり、[yuki3](https://qiita.com/yuki3) さんに多くのコメントをいただき、議論に付き合っていただきました。ありがとうございました。なお、文責はすべて私 (nhiroki) にあります。誤りや補足、質問などは気軽に [GitHub Issue](https://github.com/nhiroki/nhiroki.github.io/issues) もしくは [Twitter](https://twitter.com/nhiroki_) へお寄せください。

**更新履歴**

- 2017/12/20 Async DOM 関係の試み (WorkerNode や DOM Worklets) について注釈に追記
- 2017/12/10 公開

# 目次

1. [レンダリングエンジンと JavaScript の実行モデル](#1-レンダリングエンジンと-javascript-の実行モデル)
  - レンダリングエンジンのメッセージループ
  - JavaScript のイベントループ
  - タスクの実行
  - タスクキュー
2. [Web Worker](#2-web-worker)
  - Worker のモデル
  - 同期処理と DOM アクセス
  - Worker の種類とプロセスモデル
3. [スレッド間 (実行コンテキスト間) 通信](#3-スレッド間-実行コンテキスト間-通信)
  - postMessage()
  - Structured Clone アルゴリズムと Transferable オブジェクト
  - SharedArrayBuffer
4. [共有メモリモデル](#4-共有メモリモデル)
  - Agent
  - Agent Cluster
5. [Worker のコスト](#5-worker-のコスト)
6. [Worklet](#6-worklet)
  - Worklet とは
  - Worklet のサンプルコード (Paint Worklet)
  - Worklet の実行モデル
7. [新しい仕様の提案とさらなる最適化](#7-新しい仕様の提案とさらなる最適化)
  - Tasklets API
  - Web Locks API
  - Thread API
  - Off-main-thread
8. [まとめ](#8-まとめ)


# 1. レンダリングエンジンと JavaScript の実行モデル

ウェブブラウザで一番重要なコンポーネントはレンダリングエンジンです。Chromium では Blink というレンダリングエンジンを使用しています。レンダリングエンジンという名前ですが、実際にはレンダリング以外にも、JavaScript を実行したり、ウェブプラットフォーム API を実装したりしています。

本章ではまずレンダリングエンジンの実行モデルについて説明し、その後 JavaScript の実行モデルについて説明していきます。

## レンダリングエンジンのメッセージループ

レンダリングエンジンは基本的にシングルスレッドで動作します[^thread-model]。このスレッドをメインスレッドとか UI スレッドと呼びます。シングルスレッドですが、メッセージループによるタスク駆動型の実行方式を取るため、様々なジョブを並行して処理することが可能です。ここでいうジョブとは「多数のタスクをまとめた一つの意味ある仕事」を意味しています。

例えばリソースフェッチというジョブは、大雑把に「リクエストを投げるタスク」と「レスポンスを処理するタスク」に分けることができます。ジョブがタスクに分割されていれば、リソースフェッチジョブを実行しながら、UI に関するジョブを並行して走らせる、といったことができます。

[^thread-model]: コンポジションや I/O などは専用のスレッドで行われます。

![メッセージループ](/images/javascript-parallel-processing-message-loop.png)

## JavaScript のイベントループ

HTML の仕様により、JavaScript も[イベントループ](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)によるタスク駆動型の実行方式を取ると定義されています。イベントループでは、JavaScript のイベントやコールバック関数の実行、UI の処理などが行われます。

このイベントループはレンダリングエンジンのメッセージループをそのまま使う形で実装が行われています。つまり、JavaScript のタスクもレンダリングエンジン内部のタスクもすべて同じように実行されます。これは実装が容易である反面、色々と不都合も生じさせるのですが、それについては後述します。以後、JavaScript のイベントループもひっくるめてメッセージループと表記します。なお、メッセージループの詳細な実装については本アドベントカレンダー 23 日目に tzik さんが記事を書いてくれるはずです。

JavaScript は初回実行したあとはイベントやコールバックによって非同期に処理が進みます。このイベントやコールバックがタスクにあたります。例えば、以下のようなソースコードを実行する場合、初回実行時には ```setTimeout()``` と ```fetch()``` が同期的に実行されますが、それらのコールバックは条件が揃った時にメッセージループのタスクとして非同期に実行されます。そのため、ユーザからはタイマージョブとリソースフェッチジョブが並行して動いているように見えます。

```js
setTimeout(() => {
  renderSomething();
}, 1000);

fetch('http://example.com/get_message')
  .then(response => response.getText())
  .then(text => renderText(text));
```

![メッセージループの処理](/images/javascript-parallel-processing-event-loop.png)

## タスクの実行

メッセージループはタスクの処理中に別のタスクに実行を切り替えることはできません。つまりプリエンプティブなマルチタスクではありません。タスクに割り込んで別のタスクが動かせてしまうと、あらゆるタスクは割り込みを想定したコードを書く必要が生じ、容易に競合状態が起きてしまうからです。しかし割り込みができないとなると、実行権を握り続けるタスクによって他のタスクの実行が阻害されてしまう可能性があります。例えば、あるタスクが無限ループすると、他のタスクは永遠に実行されません。レンダリングエンジン内では無限ループすることはありません (あったらバグです) が、JavaScript では任意のコードが動かせるため無限ループすることがありえます。これが起きるとレンダリングエンジンの処理もブロックされてしまいます。

例えば次の HTML は一秒ごとに "a" を出力するページですが、"Start Infinite Loop" ボタンを押すと無限ループが始まり、以降は "a" の出力が全く行われないことが確認できます。

```html
<button onclick="while(true){}">Start Infinite Loop</button>
<div id="output"></div>
<script>
setInterval(() => {
    document.querySelector("#output").innerHTML += "a";
}, 1000);
</script>
```

このような挙動は特に 3rd-party iframe などで問題になりやすく、無限ループされないまでも、iframe 内で ```document.write()``` などが呼ばれるとトップレベルのフレームを含めて処理がブロックされてしまいます。前述のようにタスクに割り込みをすることはできないので、この処理が終わるまで延々とブロックされることになります。

Chromium ではこれを防ぐいくつかの手法が提案されています。詳しくは以下のドキュメントを参照してください。

- [Isolating performance of third-party iframes](https://docs.google.com/document/d/1CEggurHQGXenhu_GQT7KnRvtSuowuenXpxVzYSeRxSY/edit)
- [Restricted nested message loop](https://docs.google.com/document/d/1Z0FDQnVEcYGED1TZJQP4LVi57xFEzhHoHdDWQpMkKAs/edit)

またレンダリングエンジン内でもタスクをチャンクに分けるといった試みが行われています。これによりジョブの終了時間は伸びますが、タスクのスループットを上げることができます。

## タスクキュー

さきほどの図ではタスクキューがただ一つ存在するように描かれていましたが、実際にはタスクの種類によって複数のタスクキューが存在し、タスクスケジューラーによって管理されています。

![メッセージループ](/images/javascript-parallel-processing-message-loop-with-scheduler.png)

メッセージループはレンダリングエンジンと JavaScript のイベントループで共用されているため、タスクの種類は両方のものが用意されています。イベントループ用のタスクタイプ (これを [Task Source](https://html.spec.whatwg.org/multipage/webappapis.html#task-source) と呼びます) の[一部は HTML の仕様で定義](https://html.spec.whatwg.org/multipage/webappapis.html#generic-task-sources)されていて、その他の細かいものについては各 API の仕様で定義されています。一方、レンダリングエンジン内部で使われるタスクタイプは明確な仕様がないため、イベントループのタスクタイプに準拠しながら必要に応じて独自に定義しています。

Chromium の実装ではスケジューラーとタスクキューはページのメインフレームや iframe 毎に存在しています。これは厳密に言うと HTML の仕様と合致していないため、[修正する提案](https://groups.google.com/a/chromium.org/d/msg/scheduler-dev/6xQ33iNoE7o/1h4aELPpAwAJ)が行われています。スケジューラーはフレームの可視性などによってタスク実行を抑制したりします。例えば、フレームがウィンドウ背面に隠れたら処理を抑制します。ただし、オーディオのような背面でも実行されるべきタスクは免除されます。

このフレーム毎のスケジューラーの実装は [WebFrameSchedulerImpl](https://chromium.googlesource.com/chromium/src/+/8535de60c5047263328ed2c9d6e82aef69661a69/third_party/WebKit/Source/platform/scheduler/renderer/web_frame_scheduler_impl.h#47) にあり、タスクタイプとタスクキューのマッピングは [WebFrameSchedulerImpl::GetTaskRunner()](https://chromium.googlesource.com/chromium/src/+/8535de60c5047263328ed2c9d6e82aef69661a69/third_party/WebKit/Source/platform/scheduler/renderer/web_frame_scheduler_impl.cc#215) で行われています。

# 2. Web Worker

複数のタスクを高速に切り替えて実行させれば、それらが見かけ上は並列に実行されているように見えます。しかし、実際にはメインスレッド 1 つしか動いていないため、処理すべきタスクが多くなったり、一つ一つの処理に時間がかかるようになってくると UI の応答性が悪くなってきます。そこで、タスクを別スレッドで動かす仕組みができました。それが [Web Worker](https://html.spec.whatwg.org/multipage/workers.html#workers) です。

Web Worker は次のように使います。index.html と worker.js はそれぞれ別のスレッドで動くため、並列に実行できます。

```js
// index.html
const worker = new Worker('worker.js');  // start a worker asynchronously
worker.postMessage("Hello, world!");
```

```js
// worker.js
console.log("I'm a worker!");
onmessage = event => {
  console.log(event.data);  // "Hello, world!"
};
```

Web Worker の詳しい使い方については下記の記事を参照してください。

- [Using Web Workers - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers)

これ以降、Web Worker を Worker と表記することにします。

## Worker のモデル

用語の整理をしていきます。Worker が動くスレッドをワーカースレッドと呼びます。Worker は 1 ワーカースレッドを専有して使い、Worker が作られるたびにスレッドが起動します。Worker はそれぞれ JavaScript 実行環境 (V8 のインスタンス) と HTML 仕様で定義された実行コンテキストである [WorkerGlobalScope](https://html.spec.whatwg.org/multipage/workers.html#workerglobalscope) を持ちます。

同様にメインスレッド上で動くいわゆるページの実行コンテキストもあり、これを [Window](https://html.spec.whatwg.org/multipage/window-object.html#window) と呼びます。ここでとても紛らわしいことに、Chromium ではページの実行コンテキストとしての機能を Window というクラスではなく [Document](https://chromium.googlesource.com/chromium/src/+/1d45070d3678ed2cebe505f71801a7761caf9110/third_party/WebKit/Source/core/dom/Document.h#264) というクラスで実装しています。HTML 仕様では Window の下にあるオブジェクトとして [Document](https://dom.spec.whatwg.org/#interface-document) オブジェクトを定義しており、なおかつ Document は複数作られる可能性があるため実行コンテキストとして扱うのは無理があるのですが、歴史的な経緯によりこのようないびつな実装になっています。本記事では Chromium の実装に従い、ページの実行コンテキストは Document と表記していくことにします。

WorkerGlobalScope もそれぞれタスクスケジューラーとタスクキューのセットを持ち、並列に動作します。

![ワーカーメッセージループ](/images/javascript-parallel-processing-worker-message-loop.png)

## 同期処理と DOM アクセス

Worker は計算処理やストレージアクセス、リソースフェッチを行うことを意図して設計されています。UI を阻害してしまうため、メインスレッド上ではスレッドをブロックする同期的な API (e.g., [Synchronous XHR](https://developer.mozilla.org/ja/docs/Web/API/XMLHttpRequest/Synchronous_and_Asynchronous_Requests)) を使うことは避けるべきですが、Worker 上では UI を阻害することがないため気軽に使うことができます。ただし、現在では Worker 上であっても同期的な API の使用は推奨されておらず、非同期 API を使うことを強くお勧めします。比較的新しい API である [CacheStorage API](https://developer.mozilla.org/ja/docs/Web/API/Cache) や [Fetch API](https://developer.mozilla.org/ja/docs/Web/API/Fetch_API/Using_Fetch) ではそもそも同期的な API を提供していません。また後述する [Service Worker](https://developer.mozilla.org/ja/docs/Web/API/ServiceWorker_API) では同期 API が一切定義されていません。

メインスレッドでは何らかの処理結果を DOM を変化させることで視覚的に見せます。Worker でも同様に計算結果を DOM に反映させられると良いのですが、そうすると仕様や実装が複雑になってしまうため[^off-main-thread-dom]、Worker 上では DOM を操作することはできません。DOM 操作を行えるようにする提案がたびたび行われていますが、今のところ実現に至ったものはありません。一方、DOM ではありませんが、ペイントやアニメーションを別スレッドから行えるようにする API (Worklet) の実装が進んでいます。これについては後述します。

以上をまとめたのが次の表です。WorkletGlobalScope については後の章で解説します。

| 実行コンテキスト | DOM アクセス | ストレージアクセス | ネットワークアクセス |
| :--- | :--- | :--- | :--- |
| Document | 同期のみ | 非同期のみ | 非同期のみ |
| WorkerGlobalScope | 不可 | 同期もしくは非同期 | 同期もしくは非同期 |
| WorkletGlobalScope | 不可 | 不可 | 不可 |

[^off-main-thread-dom]: 例えば、メインスレッド上の DOM 処理との実行順序の決め方や同期をどうするか考える必要があります。解決策の一つに Worker 側に DOM オブジェクトへの Proxy を提供し、実際の DOM への反映はメインスレッドで行うという方法が考えられますが、あらゆる DOM オブジェクトに対して Proxy を定義する必要があり、一筋縄ではいきません。**(2017/12/20 追記) Async DOM 関係の試みとして [WorkerNode](https://github.com/drufball/worker-node/blob/master/EXPLAINER.md) や DOM Worklets といった話題が挙がっている (["New async DOM effort / list" - blink-dev](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/kMlBLNv0uwM))**

## Worker の種類とプロセスモデル

Worker にはページと一対一対応を持つ Dedicated Worker という仕組みと、同一オリジン内の複数のページと多対一対応を持つ Shared Worker という仕組みがあります。Shared Worker は複数のページで一つの Web Socket コネクションを共有する場合などに使われます。また、イベント駆動型サービス実行基盤として使われる Service Worker というものがあり、例えばオフライン対応やプッシュ通知を実装する際に使われます。Service Worker の実行モデルについては拙著の「[イベント駆動型サービス実行基盤としての Service Worker](https://qiita.com/nhiroki/items/65efc9e41ec1d928afcd)」を見てください。

Worker のプロセスモデルについては本アドベントカレンダー三日目の amiq11 さんの記事「[Chromiumのプロセス構成と Worker/SharedWorker/ServiceWorkerのうごき](https://qiita.com/amiq11/items/61cf100e5f9fac8533b6)」で詳細に解説されているので、そちらを参考にしてみてください。ざっくりまとめると、Dedicated Worker はそれを作った Document コンテキストと同じレンダラープロセスに作られますが、Shared Worker と Service Worker は別のレンダラープロセス上に作られる可能性があります。また、Dedicated Worker は Shared Worker 上からも作ることができ、この場合はその Shared Worker と同じプロセスに割り当てられます。**Dedicated Worker はそれを作った実行コンテキストと必ず同じプロセスに割り当てられるため in-process worker、Shared Worker と Service Worker はそれを作った実行コンテキストと別プロセスに割り当てられる可能性があるため out-of-process worker と呼ばれます。**

次の図はこれらをまとめたものです。実際にはこんなに複雑な依存関係を作ることはないと思います。

![ワーカープロセスモデル](/images/javascript-parallel-processing-worker-process-models.png)

# 3. スレッド間 (実行コンテキスト間) 通信

Document と Worker、及び Worker 間では JavaScript のオブジェクト空間が独立しています。例えば Document 側で作成したオブジェクトは Worker 側から触ることができません (次のソースコード参照)。実行コンテキスト間でオブジェクトをやりとりするにはメッセージを送るか、共有メモリを使う必要があります。

```js
// index.html
this.answer = 42;
console.log(this.answer + ' is the answer.');  // '42 is the answer.'
const worker = new Worker('worker.js');
```

```js
// worker.js
console.log(this.answer + ' is the answer.');  // 'undefined is the answer.'
                                               // |this.answer| is not available.
```

## postMessage()

Document と Worker 間の通信や Worker どうしの通信にはメッセージパッシング型の通信機構である ```postMessage()``` が使われます。```postMessage()``` のインタフェースは複数あり、Worker オブジェクト、MessageChannel、BroadcastChannel などがあります。

![postMessage()](/images/javascript-parallel-processing-postmessage.png)

Worker オブジェクトの ```postMessage()``` は次のように使います。Worker 側では、```self.postMessage()``` によって生成元の Document と通信することができます。

```js
// index.html
const worker = new Worker('worker.js');
worker.onmessage = event => {
  console.log(event.data);  // 'Pong'
};
worker.postMessage('Ping');
```

```js
// worker.js
onmessage = event => {
  console.log(event.data);  // 'Ping'
  self.postMessage('Pong');
};
```

[MessageChannel](https://developer.mozilla.org/ja/docs/Web/API/MessageChannel) は二つの端点 (MessagePort) を持つ双方向の通信路です。次のソースコードのように使います。片側の実行コンテキストで MessageChannel オブジェクトを作り、前述の ```postMessage()``` で port の一つを転送することで、実行コンテキスト間の通信路を確立することができます。新しく通信路を作るために Worker の ```postMessage()``` を使うのは奇妙に思えるかもしれません。Worker の ```postMessage()``` は実行コンテキストに紐付いた通信方法だったのに対し、MessageChannel の port は任意の実行コンテキストへ何度でも転送することができるため、特定の実行コンテキストに紐付かないような通信を行いたい場合に活用することができます。例えば、複数の実行コンテキストが順番に通信をしたい場合などに最適です。

```js
// index.html
const worker = new Worker('worker.js');
const channel = new MessageChannel;
channel.port1.onmessage = event => {
  console.log(event.data);  // 'Pong'
};
worker.postMessage('Ping', [channel.port2]);
```

```js
// worker.js
onmessage = event => {
  console.log(event.data);  // 'Ping'
  event.ports[0].postMessage('Pong');
};
```

[BroadcastChannel](https://developer.mozilla.org/ja/docs/Web/API/Broadcast_Channel_API) は同一オリジン内にいる複数の相手に同時にメッセージを送信する仕組みです。BroadcastChannel のコンストラクタはチャネル名を引数に取ります。名前に対応するチャネルが存在しない場合は新しく作り、存在する場合はそれを購読します。BroadcastChannel も双方向に通信が可能です。

```js
// index.html
const worker1 = new Worker('worker.js');
const worker2 = new Worker('worker.js');
Promise.all([
  new Promise(resolve => worker1.onmessage = resolve),
  new Promise(resolve => worker2.onmessage = resolve)
]).then(results => {
  console.log(results[0].data);  // 'Ready'
  console.log(results[1].data);  // 'Ready'
  const channel = new BroadcastChannel('channel_name');
  channel.postMessage('Hello!');
  channel.close();
});
```

```js
// worker.js
const channel = new BroadcastChannel('channel_name');
channel.onmessage = event => {                                        
  console.log(event.data);  // 'Hello!'
  channel.close();
};
self.postMessage('Ready');
```

## Structured Clone アルゴリズムと Transferable オブジェクト

オブジェクトは [Structured Clone](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) というアルゴリズムによってシリアライズ (ディープコピー) されて送られます。これによって配列やマップといったコンテナ型のオブジェクトも送ることができます。

![Structured Clone](/images/javascript-parallel-processing-structured-clone.png)

シリアライザは [V8ScriptValueSerializer.cpp](https://chromium.googlesource.com/chromium/src/+/9478e129bf1ab74b7629d2837b88189d234587b7/third_party/WebKit/Source/bindings/core/v8/serialization/V8ScriptValueSerializer.cpp) がメイン実装です。また、[SerializationTag.h](https://chromium.googlesource.com/chromium/src/+/dc78612b571a8ca5729f1125f39ffa0d50632061/third_party/WebKit/Source/bindings/core/v8/serialization/SerializationTag.h) にはシリアライズフォーマットが記載されているので、興味がある人は見てみると良いと思います。

Structured Clone アルゴリズムはオブジェクトのディープコピーを伴うため、大きなオブジェクトを頻繁にやり取りするような場合性能が著しく悪くなります。そこで、オブジェクトをコピーするのではなくムーブで送る [Transferable](https://developer.mozilla.org/ja/docs/Web/API/Transferable) オブジェクトという仕組みが導入されました。Transferable として送れるオブジェクトは現在のところ [ArrayBuffer](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) や [OffscreenCanvas](https://developer.mozilla.org/en-US/docs/Web/API/OffscreenCanvas) などに限られています。また、前述の MessagePort も Transferable オブジェクトとして送信しています。

![Transferables](/images/javascript-parallel-processing-transferables.png)

Transferable オブジェクトは所有権を渡してしまうため、ある時点で同時にアクセスできるスレッドは一つに限られます。このため、パイプライン処理のようにオブジェクトのアクセサが規則的に順番に代わっていくような場合にはうまくフィットしますが、複数のアクセサがランダムに読み書きしようとすると所有権の管理がややこしいことになります。

```js
// index.html
const array = new Uint8Array(1);
array[0] = 42;
console.log(array[0]);  // '42'
const worker = new Worker('worker.js');
// Specify a transferable object as the second argument.
worker.postMessage(array.buffer, [array.buffer]);
console.log(array[0]);  // 'undefined'. |array| was transferred.
```

```js
// worker.js
onmessage = event => {
  const array = new Uint8Array(event.data);
  console.log(array[0]);  // '42'
};
```

## SharedArrayBuffer

Transferable はオブジェクトの所有権を渡すことでデータコピーのコストを削減する仕組みでした。しかし所有権を渡してしまうため、複数の実行コンテキストがランダムにアクセスするようなユースケースにおいて使いにくさがありました。これを解決するのが [SharedArrayBuffer](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer) です。SharedArrayBuffer は ArrayBuffer を共有メモリ上に割り当てたものです。同一オリジンにある複数の実行コンテキストから同時にアクセスすることができます。

![SharedArrayBuffer](/images/javascript-parallel-processing-shared-array-buffer.png)

スレッド間 (実行コンテキスト間) でメモリを共有しているため、素朴にアクセスすると競合状態が発生します。スレッド間で同期を取るための機構として [Atomics API](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Atomics) が提供されています。Atomics API はその名の通り、store や exchange といったアトミック命令を提供します。

```js
// index.html
const array = new Uint8Array(new SharedArrayBuffer(1));
const index = 0;
Atomics.store(array, index, 42);
console.log(Atomics.load(array, index));  // '42'
const worker = new Worker('worker.js');
worker.onmessage = e => {
  // '24'. |array| was updated by the worker.
  console.log(Atomics.load(array, index));
};
worker.postMessage(array.buffer);
console.log(array);  // |array| is still valid.
```

```js
// worker.js
onmessage = event => {
  const array = new Uint8Array(event.data);                                      
  const index = 0;
  console.log(Atomics.load(array, index));  // '42'
  Atomics.store(array, index, 24);
  console.log(Atomics.load(array, index));  // '24'
  self.postMessage('');
};
```

また、Linux の [Futex](https://en.wikipedia.org/wiki/Futex) を参考に策定された wait / wake 関数も定義されていて、実行コンテキスト間で待ち合わせすることもできます。

```js
// index.html
// wait() and wake() are available only for Int32Array.
const array = new Int32Array(new SharedArrayBuffer(4));
const index = 0;
Atomics.store(array, index, 42);
console.log(Atomics.load(array, index));  // '42'
const worker = new Worker('worker.js');
worker.onmessage = e => {
  Atomics.store(array, index, 24);

  // Wake up a waiting client on array[index]. Return the number of woken
  // clients.
  const result = Atomics.wake(array, index, 1);
  console.log(result);  // '1'
};
worker.postMessage(array.buffer);
```

```js
// worker.js
onmessage = event => {
  const array = new Int32Array(event.data);
  const index = 0;
  console.log(Atomics.load(array, index));  // '42'
  self.postMessage('waiting an update...');

  // Wait while array[index] is changed from 42.
  const result = Atomics.wait(array, index, 42);
  console.log(result);  // 'ok'
  console.log(Atomics.load(array, index));  // '24'
};
```

Atomics API はスレッドをブロックする可能性があるため、スレッドブロックが許可されている Dedicated Worker と Shared Worker でのみ使うことができます。Document はブロックすると UI が止まってしまうため許可されていません。同様に Service Worker もブロックするとナビゲーションが止まってしまうため許可されていません。これらの制限については次章で解説します。

# 4. 共有メモリモデル

SharedArrayBuffer はどの実行コンテキスト間でも共有できるわけではありません。ECMAScript の仕様では、スクリプトの実行スレッドを Agent という概念によって抽象化しています。また、Agent どうしが共有メモリによって通信できる範囲を Agent Cluster と定義しています。一方、HTML の仕様では Agent と AgentCluster を使って、実行コンテキストが SharedArrayBuffer によって通信可能な範囲を定義しています。

本章ではこれらの仕様について説明し、Chromium でどのように実現されているか見ていきます。

## Agent

[Agent](https://tc39.github.io/ecma262/#sec-agents) はスクリプトの実行スレッドと実行に必要な環境を抽象化するものです。各 Agent は Agent Record と呼ばれるフィールドを持っていて、それにより様々な特徴づけが行われています。次の表は仕様からフィールドの一部を抜粋したものです。

| フィールド名 | 値 | 意味 |
| :--- | :--- | :--- |
| [[LittleEndian]] | Boolean | 処理系のエンディアン |
| [[CanBlock]] | Boolean | Agent がブロックできるかどうか |
| [[IsLockFree1]] | Boolean | 1 byte のアトミック処理がロックフリーで行えるかどうか |
| [[IsLockFree2]] | Boolean | 2 bytes のアトミック処理がロックフリーで行えるかどうか |

Agent における「実行スレッド」とは抽象的なものであり、実際には複数の Agent によって下位のネイティブスレッドが共有される場合があります。この場合スレッドをブロックしてしまうと他の Agent もブロックされてしまうため、[[CanBlock]] は false となります。

[HTML 仕様](https://html.spec.whatwg.org/multipage/webappapis.html#integration-with-the-javascript-agent-formalism) ではこの Agent の概念を実行コンテキストに適用しています。具体的には以下のものが定義されています。

- **Similar-origin window agent** : Window (Document) のための Agent。UI を止めないために [[CanBlock]] が false です。
- **Dedicated worker agent** : Dedicated Worker のための Agent。[[CanBlock]] は true です。
- **Shared worker agent** : Shared Worker のための Agent。[[CanBlock]] は true です。
- **Service worker agent** : Service Worker のための Agent。ナビゲーションを止めないために [[CanBlock]] が false です。
- **Worklet agent** : Worklet のための Agent。Worklet については後述。Worklet 間でネイティブスレッドを共有するため [[CanBlock]] が false です。

## Agent Cluster

[Agent Cluster](https://tc39.github.io/ecma262/#sec-agent-clusters) は Agent が共有メモリによって通信できる範囲を定義したもので、Agent のセットによって構成されます。各 Agent は必ず一つの Agent Cluster に所属しています。

Agent Cluster 内のすべての Agent は [[LittleEndian]] と [[IsLockFree1/2]] が同じ値を持っている必要があります。例えばエンディアンが異なると、Agent 間でマルチバイトのデータをメモリ共有するのが困難になってしまいます。

Agent 同様、[HTML 仕様](https://html.spec.whatwg.org/multipage/webappapis.html#integration-with-the-javascript-agent-cluster-formalism) によってどのように Cluster が構成されるかが定義されています。Cluster を構成する上でまず考えなくてはならないのは、セキュリティと共有可能なリソースです。これは Cluster を構成できる範囲を [same origin-domain](https://html.spec.whatwg.org/multipage/origin.html#same-origin-domain) に制限することで実現しています。次にレンダリングエンジンによる実装可能性を考慮する必要があります。一般的にプロセスをまたいでメモリを共有するのは実装上難しいため、異なるプロセスに割り当てられる可能性のある実行コンテキストは別の Cluster に分類しています。

以上を踏まえ、HTML の仕様ではいくつかの具体例が挙げられています。以下のような場合、それぞれの実行コンテキストは同じプロセスでかつ same origin-domain に割り当てられるため、同一の Agent Cluster になります。

- Window とそれが作った Dedicated Worker。
- 任意の Worker とそれが作った Dedicated Worker。
- Window とそれが作った same origin-domain iframe の Window。
- Window とそれが開いた same origin-domain Window。

一方、次のような場合はそれぞれ別の Agent Cluster に分類されます。まず Shared Worker や Service Worker は out-of-process worker であるため、同一プロセスであることが保証できません。また、same origin-domain ではない実行コンテキスト間でリソースを共有するのはセキュリティの問題があるため、そのような実行コンテキストは別 Cluster になります。

- Window とそれが作った Shared Worker。
- 任意の Worker とそれが作った Shared Worker。
- Window とそれが作った Service Worker。
- Window とそれが開いた same origin-domain ではない iframe の Window。

Worker に着目して以上をまとめると、次の図のようになります。これから分かる通り、プロセスをまたいで Agent Cluster が構築されることがないようになっています。異なるプロセスに割り当てられる可能性のある Shared Worker や Service Worker の間で SharedArrayBuffer を共有することはできません。

![Agent Clusters](/images/javascript-parallel-processing-agent-clusters.png)

なお同一の Agent Cluster に所属していない場合も、Structured Clone アルゴリズムによる ```postMessage()``` によって通信することができます。

Chromium では Agent Cluster に直接対応する実装はありませんが、前述の Worker のプロセスモデルの制約により、自然とそのようなモデルになっています。

# 5. Worker のコスト

タスクを別スレッドで実行できる便利な Worker ですが、実は Worker の起動にはかなりのコストがかかります。

まずオペレーティングシステムのネイティブスレッドをメインスレッドから起動します。メインスレッドが忙しい場合は起動に少し時間がかかる可能性があります。スレッドの起動数はオペレーティングシステムによって制限があるため、気を付ける必要があります。またスレッドはそれぞれ独自のメモリスタックを持つためメモリ使用量が増え、スレッド数が多いとコンテキストスイッチによるオーバヘッドも増加します。

スレッドを起動したら今度はそのスレッド上で JavaScript の実行エンジンを初期化します。JavaScript の実行エンジンの起動もセットアップや実行環境の保持にリソースを使います。さらにその上に JavaScript の実行コンテキスト (WorkerGlobalScope) を構築し、JavaScript ソースコードを評価する必要があります。

![ワーカースタート](/images/javascript-parallel-processing-worker-start.png)

このことは Web Worker の仕様でも明言されています。

> Workers (as these background scripts are called herein) are relatively heavy-weight, and are not intended to be used in large numbers. For example, it would be inappropriate to launch one worker for each pixel of a four megapixel image. The examples below show some appropriate uses of workers.
> 
> Generally, workers are expected to be long-lived, have a high start-up performance cost, and a high per-instance memory cost.
>
> ([10.1.1 Scope](https://html.spec.whatwg.org/multipage/workers.html#scope-2) / HTML
Living Standard — Last Updated 9 September 2017)

以上により、Worker の使用は控えめにすべきです。もしたくさんのタスクを実行したい場合は、タスクごとに Worker を作るのではなく、あらかじめ Worker をプールしておいて負荷に応じて適当な Worker にタスクを振り分けるといった方式 (ワーカープール) を検討すべきです。また、Worker は使い終わったら速やかに ```Worker#terminate()``` もしくは ```WorkerGlobalScope#close()``` によって破棄することをおすすめします。

V8 の実行コンテキストのスタートアップコストを削るために、JavaScript の実行環境のスナップショットを使って起動を早くする試みがあります。これについては 12 日目の peria さんの記事で詳しく紹介されると思います。

# 6. Worklet

本章では JavaScript の新しい実行コンテキストである Worklet について紹介します。

## Worklet とは

最近の傾向として、ブラウザの様々な処理をフックしてカスタムスクリプトを仕込めるような仕組みが次々と作られています。例えば、Service Worker はネットワークリクエストをフックして任意のレスポンスを返すことができます。Service Worker ではカスタムスクリプトの実行環境として、独自の Worker を定義しました。このようなカスタムスクリプトの実行環境を各 API でそれぞれ独自に定義するのは大変なので、ベースとなる実行環境を定義し、それを各 API が自由に拡張して使える仕組みが提案されました。それが [Worklet](https://drafts.css-houdini.org/worklets/) です[^houdini-worklet]。

[^houdini-worklet]: [Worklet](https://drafts.css-houdini.org/worklets/) の仕様を見て気づかれた方もいるかもしれませんが、Worklet は CSS を拡張可能にするプロジェクト [CSS Houdini](https://github.com/w3c/css-houdini-drafts/wiki) の一部として仕様が策定されています。しかし、CSS 専用の機能ではありません。これは元々 Worklet が CSS Paint API のカスタムスクリプトの実行環境として作られたという歴史的な理由によるもので、他の用途でも使われるようになった現在でもそのまま Houdini の仕様として残っています。

Worklet はあくまでもベースとなる実行環境であり、それ単体では使用することはできません。Worklet を拡張してカスタムスクリプトを提供している API には、[Paint Worklet](https://drafts.css-houdini.org/css-paint-api-1/)、[Animation Worklet](https://wicg.github.io/animation-worklet/)、そして [Audio Worklet](https://webaudio.github.io/web-audio-api/#AudioWorklet) があります。どの API も Chrome Canary (本記事公開時点ではバージョン 64) 上でフラグを有効にすると試すことができます。Paint Worklet は[まもなくデフォルトで有効化](https://groups.google.com/a/chromium.org/forum/#!msg/blink-dev/Jex3idOld48/rKUrV8JfAAAJ)され、Audio Worklet は[試験的に有効化](https://groups.google.com/a/chromium.org/d/msg/blink-dev/oeBf3websgM/Smi7gQxjAQAJ)する段階です。実はもう一つ [Layout Worklet](https://drafts.css-houdini.org/css-layout-api/#layout-worklet) というものがありますが、まだ実装が始まっていません。

まだ少数ですが、Worklet に関して紹介している資料がいくつかあるので紹介しておきます。

- [Houdini – CSS の秘密を解き明かすもの](https://developers.google.com/web/updates/2016/05/houdini?hl=ja)
- [Houdini Paint API](https://blog.jxck.io/entries/2017-10-31/houdini-paint-api.html)
- [Compositor Worklet evolves into Animation Worklet!](https://dassur.ma/things/animworklet/)
- AudioWorklet :: What, Why and How ([slide](https://docs.google.com/presentation/d/11OZyHyWRTWOCETW4x7m5gCPNSdSl9HevCQ1aiR_0xFE/edit?usp=sharing), [video](https://www.youtube.com/watch?v=g1L4O1smMC0))
  - Chromium での AudioWorklet の実装者 (Hongchan さん) による発表。

## Worklet のサンプルコード (Paint Worklet)

Worklet の雰囲気を掴んでもらうために、Paint Worklet のサンプルコードを提示します。次のコードは ```<div>``` タグ内に同心円を描くスクリプトを登録しています。

```html
<!-- index.html -->
<html>
<head>
<title>PaintWorklet: Circles</title>
</head>
<body>
<div id='box'></div>
<style>
#box {
  --outer-circle-color: #0000FF;
  --inner-circle-color: #00FF00;
  background-image: paint(circle);
  width: 300px;
  height: 300px;
}
</style>
<script>
CSS.paintWorklet.addModule('paintworklet.js')
  .then(() => console.log('Registered'))
  .catch(e => console.log('Failed: ' + e.message));
</script>
</body>
</html>
```

```js
// paintworklet.js
registerPaint('circle', class {                                                 
  static get inputProperties() {
    return ['--outer-circle-color', '--inner-circle-color'];
  }
  paint(ctx, geom, properties) {
    const x = geom.width / 2;
    const y = geom.height / 2;
    const radius = Math.min(x, y);

    // Draw an outer circle.
    ctx.fillStyle = properties.get('--outer-circle-color').toString();
    ctx.beginPath();
    ctx.arc(x, y, radius, 0, 2 * Math.PI);
    ctx.fill();

    // Draw an inner circle.
    ctx.fillStyle = properties.get('--inner-circle-color').toString();
    ctx.beginPath();
    ctx.arc(x, y, radius / 2, 0, 2 * Math.PI);
    ctx.fill();
  }
});
```

実行結果は次のとおりです。

![PaintWorklet Circles](/images/javascript-parallel-processing-paint-worklet.png)

各 Worklet API については別途記事を書く予定です。本章では Worklet の実行モデルに焦点を当てて見ていきます。

## Worklet の実行モデル

Worklet は基本的には Worker のようなものですが、実行モデルや想定されている用途が異なります。以下でいくつか重要な点を紹介していきます。Worklet の仕様の [1.1. Motivations](https://drafts.css-houdini.org/worklets/#motivations) も合わせて参照してください。

- Worker は一つの WorkerGlobalScope がスレッドや V8 実行環境 (v8::Isolate) を専有するのに対し、Worklet は複数の WorkletGlobalScope がスレッドや V8 実行環境を共有することができます[^worklet-isolate]。スレッド上の実行コンテキストの数などは各 API の仕様や実装毎に自由に決めることができます (具体例は下の表を参照してください)。また、WorkerGlobalScope は一つのトップレベルスクリプトしかホストできませんが、WorkletGlobalScope は複数のトップレベルスクリプトをホストすることができます。これにより、実行コンテキスト毎にスレッドや V8 をスタートするコストを削減することができ、細かいスクリプトを大量に実行する場合に有利になります。一方、重い計算タスクを実行するような場合はスレッドを専有できる Worker を使うほうが効率が良いです。**より抽象的な視点から見ると、Worklet はスレッドや実行コンテキストの数とタスクディスパッチのポリシーを自由に設計できるワーカプールみたいなものです。**

[^worklet-isolate]: V8 に詳しい人向けの説明：Chromium では各スレッド毎に v8::Isolate のインスタンスを持ち、グローバルスコープ毎に v8::Context を作ります。Worker では一つの v8::Isolate インスタンスに一つの v8::Context でしたが、Worklet では一つの v8:Isolate インスタンスに複数の v8::Context が作られます。

![Worker and Worklet thread models](/images/javascript-parallel-processing-worker-worklet-thread-models.png)

| 名前 | 実行スレッド | 実行コンテキスト数 |
| :--- | :--- | :--- |
| Paint Worklet | メインスレッド | ブラウザが最低でも二個以上作る[^paint-worklet-global-scope] |
| Animation Worklet | ワーカスレッド[^animation-worklet-thread] | (記述なし) |
| Audio Worklet | ワーカスレッド | AudioContext 毎に最低一個以上作る[^audio-worklet-global-scope] |

[^paint-worklet-global-scope]: "The user agent must have, and select from at least two PaintWorkletGlobalScopes in the worklet’s WorkletGlobalScopes list, unless the user agent is under memory constraints. Note: This is to ensure that authors do not rely on being able to store state on the global object or non-regeneratable state on the class." / [CSS Painting API Level 1 - Editor’s Draft, 9 November 2017](https://drafts.css-houdini.org/css-paint-api-1/#draw-a-paint-image)

[^animation-worklet-thread]: 現在の実装ではコンポジタースレッドで動いていますが、ワーカスレッドへの移行作業が進んでいます。 / [AnimationWorklet - Use a dedicated backing thread](https://bugs.chromium.org/p/chromium/issues/detail?id=731727)

[^audio-worklet-global-scope]: "At least one AudioWorkletGlobalScope exists for each AudioContext that contains one or more AudioWorkletNodes. The running of imported scripts is performed by the UA as defined in [worklets-1], in such a way that all scripts are applied consistently to every global scope, and all scopes thus exhibit identical behavior. Beyond these guarantees, the creation of global scopes is transparent to the author and cannot be observed from the main window scope." / [Web Audio API - W3C Editor's Draft 30 November 2017](https://webaudio.github.io/web-audio-api/#AudioWorkletGlobalScope)

- Worker はコンストラクタオプションで従来のクラシックスクリプトとモジュールスクリプト (ES6 Modules) のどちらで実行するか選ぶことができますが[^module-worker]、Worklet はデフォルトでモジュールスクリプトとして動きます。

[^module-worker]: モジュールスクリプトとして動くワーカーはまだどのブラウザでも実装されていません。Chromium での実装状況については[こちらの issue](https://bugs.chromium.org/p/chromium/issues/detail?id=680046) を参照してください。

- Worklet は実行してほしいモジュールスクリプトを WorkletGlobalScope に追加していく、というセマンティクスを持ちます。これは Worklet オブジェクトに実装されている ```addModule()``` 関数によって行います。追加されたスクリプトはプールされているすべての WorkletGlobalScope にインポートされます。このスクリプトは ```registerXXX()``` (XXX は仕様名が入る) によってクラスを登録することができ、レンダリングエンジンの処理がフックポイントに来たタイミングで適当な WorkletGlobalScope を選んで、登録されたクラスの特定の関数を呼び出します。前述の Paint Worklet のサンプルコードだと、```registerPaint()``` でクラスを登録し、フックポイントでは ```inputProperties()``` と ```paint()``` が呼ばれることになります。

- スクリプトの実行環境や実行順序は完全に実装依存なので、それらを仮定したスクリプトを書いてはいけません。例えば、グローバルオブジェクトにデータを保存することは推奨されていません。また、同様の理由により、不確定性を生み出すネットワーキング API (e.g., Fetch API や Dynamic import) は無効化されており、その他の不確定性のある機能も使用が非推奨とされています。詳しくは仕様の [1.1. Motivations](https://drafts.css-houdini.org/worklets/#motivations) や [1.2. Code Idempotency](https://drafts.css-houdini.org/worklets/#code-idempotency) を見てください。

# 7. 新しい仕様の提案とさらなる最適化

前章までで JavaScript のスレッド並列環境について包括的に見てきました。本章では現在実装中だったり、これから実装されるであろう仕様や最適化について紹介します。

## Tasklets API

[Tasklets API](https://github.com/GoogleChromeLabs/tasklets) は Worklet を基盤とした汎用的なバックグラウンドタスクの実行環境です。まだ仕様の提案が始まった段階です。前章で紹介したとおり、Worklet は各 API が拡張して使うための機能であり、Worker のように任意のタスクをバックグラウンドで実行させる仕組みではありませんでしたが、Tasklet によってこれが可能になります。

```js
// fetcher.js
export async function fetchDataObject() {
  const resp = await fetch(/*...*/);
  const json = await resp.json();
  return doSomeExpensiveProcessing(json);
}

// app.js
const fetcher = await tasklets.addModule('fetcher.js');
const json = await fetcher.fetchDataObject();
// ...
```

Tasklet の提案には次のような背景があります。

1. 汎用的なタスク実行環境には Dedicated Worker がありますが、Dedicated Worker はスレッドを専有して実行するため、その起動にコストがかかります。タスクごとに Dedicated Worker を起動するのは現実的ではありません。これを解決する方法としては Dedicated Worker を使ったワーカプールを構築するという手法がありますが、プールの管理やワーカの負荷状況に応じたタスクディスパッチの仕組みを自前で用意しなければいけません。
2. Dedicated Worker の ```postMessage()``` で高度なメッセージングを行う場合、命令タイプを String オブジェクトにエンコードしてやりとりする必要があり、煩雑になりがちです。
3. Android や iOS では UI 固有の処理以外は基本的にバックグラウンドスレッドで実行するように強制されます。一方、ウェブでは任意のタスクをメインスレッド上で実行することが可能で、それが UI タスクを阻害しがちです。

Tasklet はこれらを解決できるように設計議論が行われています。まず (1) に関しては、Tasklet は Worklet をベースにしているためスレッドを共有して複数の実行環境を持つことができ、Worker に比べれば低コストです。またタスクディスパッチもブラウザが負荷状況に応じて適切に実行環境を選ぶことができます。(2) に関しては、Tasklet では ECMAScript を拡張して、スレッド間 RPC (Remote Procedure Call) のようなことを行えるようにする提案がなされています。前述のサンプルコードの ```fetchDataObject()``` がそれです。引数は Structured Clone アルゴリズムによってシリアライズして送られます。これによって命令タイプを自身でエンコードする必要がなくなります。(3) に関しては、将来的に Non-UI タスクを Tasklet 上でしか実行できなくすることで、意図しない UI タスクのブロックを減らせることが期待できます。

## Web Locks API

Atomics API は同一 Agent Cluster 内 (プロセス内) の実行コンテキストで同期をとる仕組みでした。これに対し、[Web Locks API](https://github.com/inexorabletash/web-locks) は Agent Cluster 間 (プロセス間) の実行コンテキストで同期をとる仕組みとして提案されています。平たく言うと、ブラウザのタブ間で同期をとる仕組みです。スレッドの機能とはちょっと違いますが、ワーカー上でも使えるようなのでここで紹介します。

ストレージ系の API はオリジン毎にデータを持つため、同一オリジンのページを複数のタブで開いている場合はそれらを協調して動作させる必要がありました。Indexed DB ではトランザクションによってこれを実現していますが、あくまでも Indexed DB 固有の機能のため、他のストレージ API やネットワーク API で直接利用することはできません。これに対し、Web Locks API は汎用的に使える同期機構として設計されていて、[Chromium で試験実装が始まっています](https://chromium-review.googlesource.com/c/chromium/src/+/657902)。

次のコードは Web Locks API を使って、Readers-Writer Lock パターンを実装している例です。```navigator.locks.acquire()``` でロックを取得します。第一引数はロックのスコープを表し、文字列もしくは文字列の配列を指定します。詳しくは後述します。第二引数はオプションで、ロックモード (共有ロック "shared" と排他ロック "exclusive") を指定することができ、これによって Readers-Writer Lock パターンを実現することができます。デフォルトだと exclusive のようです。第三引数はロック取得時に呼ばれるコールバックで、Lock オブジェクトを引数に取ります。この Lock オブジェクトが生きている間はロックがかけられている状態になり、スコープアウトするとロックが解放されます。

```js
async function get_lock_then_write() {
  await navigator.locks.acquire('resource', {mode: 'exclusive'}, async lock => {
    await async_write_func();
  });
}

async function get_lock_then_read() {
  await navigator.locks.acquire('resource', {mode: 'shared'}, async lock => {
    await async_read_func();
  });
}
```

さて、第一引数のスコープですが、これはロックの名前と言い換えても良いかもしれません。配列指定できることから分かる通り、指定されたロックモードで複数のロックをまとめて取得することができます。以下の例は GitHub 上の説明から引用しました。

- Held
  - #1: scope: ['a'], mode: "exclusive"
  - #2: scope: ['b'], mode: "shared"
- Requested:
  - #3: scope: ['b'], mode: "shared"
  - #4: scope: ['a', 'b'], mode: "exclusive"
  - #5: scope: ['b'], mode: "shared"
  - #6: scope: ['c'], mode: "exclusive"

ロックリクエスト #1, #2 は成功していて、それぞれロック a を排他ロック、ロック b を共有ロックしています。この状態でリクエスト #3 が来ると、ロック b は共有ロックなのでロックに成功します。次にリクエスト #4 はロック a, b の排他ロックを要求しますが、それらは既にロック済みなのでリクエスト #1, #2, #3 がロックを解放するまで待ちます。リクエスト #5 はロック b の共有ロックを要求します。これは一見するとリクエスト #4 を追い越してただちに成功しそうですが、Web Locks API ではロックリクエストの処理順序が保証されるため、リクエスト #4 がロック b を獲得・開放するまで待ちます。このあたりはロック順序に保証がない一般的な Readers-Writer ロックとは異なる部分だと思います。

このように第一引数によってロックの範囲を指定することができ、これがスコープと呼ばれる所以になっています。ロックスコープが重なるリクエストが行われた場合、重なる部分が排他的ロックな場合はリクエストが待たされることになります。

## Thread API

Worker や Worklet はスレッドプリミティブに JavaScript から安全に使えるようにラッパーを被せたものでしたが、JavaScript からスレッドプリミティブに直接触れるようにするのはどうか、という検討も行われています。詳しくは「[Concurrent JavaScript: 
It can work! - WebKit Blog](https://webkit.org/blog/7846/concurrent-javascript-it-can-work/)」を参照してください。次のコードスニペットはブログから引用しています。

```js
let result = new Thread(() => 42).join(); // returns 42
```

```js
let lock = new Lock();
lock.hold(function() { /* ...perform work with lock held... */ });
```

```js
let threadLocal = new ThreadLocal();
function foo()
{
    return threadLocal.value;
}
new Thread(function() {
    threadLocal.value = 43;
    print("Thread sees " + foo()); // Will always print 43.
});
threadLocal.value = 42;
print("Main thread sees " + foo()); // Will always print 42.
```

## Off-main-thread

本記事の最初で紹介したとおり、メインスレッドはレンダリングエンジンにおいてとても重要なリソースであり、なるべくメインスレッドを使える状態にしておくことが滑らかな UI を実現する上で重要になります。しかし、実際には多くのタスクが実装上の理由によりメインスレッドでしか動かせないため、メインスレッドは忙しい傾向があります。特にページ読み込み時は JavaScript の実行やリソース読み込みなどがメインスレッドで実行されるため、とても忙しくなります。

これを解消するために、様々なタスクをメインスレッド以外でも実行できるようにする試みが行われています。例えば Worker 周りだと、実装都合によりワーカー上のネットワークリクエストは必ず一旦メインスレッドを通るのですが、これをワーカスレッド上で処理できるようにする変更が加えられています。また、Worker のスタートアップも必ずメインスレッドから行われるのですが、これを別のスレッドからも行えるように実装を変更することが検討されています。

# 8. まとめ

本記事では Chromium における JavaScript のスレッド並列実行環境について仕様・実装・API の面から包括的に紹介しました。JavaScript における並列プログラミングはまだまだ発展途上の段階で、これから様々な提案がなされていくと思います。本記事を読んで仕様や実装に興味を持ち、Chromium プロジェクトに参加してくれる方が一人でも増えたら幸いです。

# 注釈
