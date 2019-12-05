---
layout: post
title: "Chrome 80 から Web Worker (Dedicated Worker) で ES Modules が使えます"
date: 2019-12-05 00:00:00 +09:00
tags: web
image: /images/profile.png
---

Chrome 80 から Web Worker (Dedicated Worker) で ES Modules が使えるようになります。本記事はその宣伝です。

# 前提知識

- ES Modules って何？
  - ざっくりいうとスクリプトファイルをモジューラブルに読み込む仕組みです。
  - 他の方が解説した記事がいっぱいあるのでそっちを見てください。
- Web Worker って何？
  - ざっくりいうと Web でスレッドを使うための API です。
  - MDN の解説（[これ](https://developer.mozilla.org/ja/docs/Web/API/Web_Workers_API)とか[これ](https://developer.mozilla.org/ja/docs/Web/API/Web_Workers_API/Using_web_workers)）を読むか、詳しく知りたい人は「[JavaScript のスレッド並列実行環境](/2017/12/10/javascript-parallel-processing)」を読んでください。

スクリプトファイルの読み込みについては以前「[JavaScript のスクリプトインポートを正しく使い分けようという話](/2018/09/07/javascript-import)」に詳しくまとめたのでそちらも併せて読んでください。

# 使い方

Dedicated Worker で ES Modules (Module Script) をインポートする方法は 3 つあります。

## Worker コンストラクタ

1 つ目は Worker コンストラクタの第二引数で type オプションを指定する方法です。type に 'module' を指定すると worker のトップレベルスクリプト（要するに最初に実行されるスクリプト）が Module Script として実行されます。

```html
<script>
const worker = new Worker('module-worker.js', { type: 'module' });
</script>
```

このとき worker を作るスクリプトが Module Script である必要はありません。上の例だと ```<script>``` タグに ```type='module'``` を指定しなくても大丈夫です。

## Static import

2 つ目は worker 内で static import する方法です。

```js
// In module-worker.js
import * as module from './hello-module.js';
module.SayHello();
```

Static import は Module Script 内でしか使えないため、1 つ目の方法で worker のトップレベルスクリプトを Module Script として起動しておく必要があります。以下の例だとトップレベルスクリプトが Classic Script になるため static import が失敗し、worker の起動自体がエラーになります。

```html
<script>
// Worker will be loaded as classic script.
const worker = new Worker('module-worker.js');
worker.onerror = e => {
  // This error event handler will be fired.
};
</script>
```

## Dynamic import

3 つ目は worker 内で dynamic import する方法です。

```js
// In module-worker.js
import('./hello-module.js')
  .then(module => module.SayHello());
```

Dynamic import は Classic Script として起動した worker 内でも使えます。

```html
<script>
const worker = new Worker('module-worker.js');
worker.onerror = e => {
  // This error event handler won't be fired.
};
</script>
```

# よくある質問

## Shared Worker や Service Worker でも使えますか？

本記事執筆時点ではまだ使えません。仕様は定義されていますがまだ実装されていません。実装状況は [crbug/824646](https://crbug.com/824646) と [crbug/824647](https://crbug.com/824647) をそれぞれチェックしてください。この機能が早く欲しい方はチケットにスターを付けてもらえると作業の優先度を決める参考になります。

## 他のブラウザでも使えますか？

本記事執筆時点では Chromium ベースのブラウザ (Edge, Opera etc) でのみ使うことができます。Firefox や Safari はまだ実装していません。そのうち実装されると思うので、それぞれのチケット（[Firefox チケット](https://bugzilla.mozilla.org/show_bug.cgi?id=1247687) / [Safari チケット](https://bugs.webkit.org/show_bug.cgi?id=164860)）をチェックしてください。

## Module Script 内で importScripts() は使えますか？

使えません。```importScripts()``` は Classic Script でのみ使えます。詳しくは『[JavaScript のスクリプトインポートを正しく使い分けようという話](/2018/09/07/javascript-import)』を見てください。

## modulepreload は使えますか？

使えません。そもそも Web Worker は preload にもまだ対応していません ([crbug/946510](https://crbug.com/946510))。代わりに prefetch してください。

## (他に質問があれば気軽に聞いてください)

# リンク

- [Dedicated Worker 向けに ES Modules を有効化した変更](https://chromium-review.googlesource.com/c/chromium/src/+/1844524)
- [Intent to Implement and Ship: ES Modules for dedicated workers (‘module’ type option) - blink-dev](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/hnIOyxASKFU)
- [ES Modules for dedicated workers ('module' type option) - Chrome Platform Status](https://www.chromestatus.com/feature/5761300827209728)