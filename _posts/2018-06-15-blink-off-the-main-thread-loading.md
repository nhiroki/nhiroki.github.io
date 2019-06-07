---
layout: post
title: "ネットワーク API のメインスレッド依存をなくす話"
date: 2018-06-15 00:00:00 +09:00
tags: web
image: /images/profile.png
---

本記事では Blink レンダリングエンジンにおけるリソースローディングのメインスレッド依存をなくす試みについて紹介します。

# はじめに

[前回の記事](/2018/05/07/off-the-main-thread-api)では off-the-main-thread API の流れと、その代表的なものとして Web Worker を紹介しました。Web Worker はワーカスレッドで JavaScript を実行する API ですが、Blink レンダリングエンジンの内部処理がすべてワーカスレッドで完結するわけではありません。

# メインスレッド依存を取り除く試み

Web Worker 上でありながらメインスレッドに依存する処理の代表的なものがネットワーク API です。リソースの取得がメインスレッドに依存しているため、Web Worker 上でネットワークリクエストを行うと必ずメインスレッドを経由します。このため、メインスレッドで大量のタスクや実行時間の長いタスクが実行されていると、Web Worker によるネットワーク処理のレイテンシが大きくなります。

![on-the-main-thread loading](/images/blink-off-the-main-thread-loading-on.png)

現在、この制約を取り除いてネットワーク API が off-the-main-thread で動作するように実装が進められています。これが実現すると次の図のようになります。

![off-the-main-thread loading](/images/blink-off-the-main-thread-loading-off.png)

この一年で多くのネットワーク API からメインスレッド依存が取り除かれました。残るは Web Worker の top-level classic script (非 ES Modules) のローディングで、これも現在活発に開発が行われています。

| ネットワーク API | Off-the-main-thread 化？ |
| :--- | :--- |
| Fetch API | 済 ([Issue 443374](https://bugs.chromium.org/p/chromium/issues/detail?id=443374)) |
| XMLHttpRequest | 済 ([Issue 706331](https://bugs.chromium.org/p/chromium/issues/detail?id=706331)) |
| WebSocket | 済 ([Issue 825740](https://bugs.chromium.org/p/chromium/issues/detail?id=825740)) |
| Worker classic script loading | 実装中 ([Issue 835717](https://crbug.com/835717), [924041](https://crbug.com/924041), [924043](https://crbug.com/924043)) |
| importScripts() | 済 ([Issue 706331](https://crbug.com/706331)) |
| Worker module script loading | 済 ([Issue 680046](https://bugs.chromium.org/p/chromium/issues/detail?id=680046#c37)) |
| Worklet module script loading | 済 ([Issue 846938](https://bugs.chromium.org/p/chromium/issues/detail?id=846938)) |
| Dynamic import() | 済 ([Issue 680046](https://bugs.chromium.org/p/chromium/issues/detail?id=680046#c37)) |

私は主に WebSocket のメインスレッド依存を取り除く作業を担当しました。依存を取り除いた結果、メインスレッドがそこそこ忙しい状況を再現した簡易的なベンチマークのスループットが二倍になったという[報告](https://bugs.chromium.org/p/chromium/issues/detail?id=825740#c44)があります。現実のアプリケーションではそこまでの性能は出ないと思いますが、いずれにせよメインスレッド依存を取り除くことで Web Worker 上でのネットワーク API の性能向上が見込めそうです。

# 未来

ネットワーク処理の中でも特にスクリプトローディングの off-the-main-thread 化が実現することで、Web Worker の起動をメインスレッド非依存で行えるようになり、スケーラビリティの向上や Worker 起動にかかる遅延を抑えることができます。特に Service Worker の起動が早くなると、ナビゲーションからレスポンスを返すまでの時間を削減することができます (Service Worker の起動時間による影響は "[Speed up Service Worker with Navigation Preloads](https://developers.google.com/web/updates/2017/02/navigation-preload)" という記事が詳しいです)。さらに Web Worker から Web Worker を作る [nested workers](https://bugs.chromium.org/p/chromium/issues/detail?id=31666) など、従来のアーキテクチャだと実装が困難だった機能が実現できるようになります。

**追記**: Chrome 69 から nested workers が使えるようになりました。詳しくは「[Chrome 69 で Web Worker から Web Worker を作れるようになった話](/2018/10/29/nested-workers)」を見てください。

# (おまけ) 実装のざっくりした話

最後にネットワーク処理がメインスレッド依存していた理由について、実装の観点からざっくりと話をします。なお、本節に出てくる実行コンテキストや Web Worker のプロセスモデルの話は、以前書いた「[JavaScript のスレッド並列実行環境](/2017/12/10/javascript-parallel-processing#2-web-worker)」で詳しく説明しているのでそちらを見てください。

ページの実行コンテキストは Document というクラスで表現されており、Document はページを表示する Frame を持ちます。ローダーのクラス群はこの Frame に紐付いています。この Frame はメインスレッドでしか触ることができません。

一方、Web Worker の実行コンテキストは WorkerGlobalScope というクラスで表現されています。Web Worker にはページ描画という概念が存在しないため、WorkerGlobalScope は Frame を持ちません。つまりローダーのクラス群が使えません。そこで、どこかの Frame にどうにかしてアクセスして、ローダーの機能を間借りする必要があります。

In-process worker である Dedicated Worker は親の Document が必ずプロセス内に存在するため、その Frame のローダーを利用します。一方、Service Worker のような out-of-process worker ではプロセス内に Document が存在しない可能性があるため、ダミーの Document を用意してそのローダーを利用します。この仕組みを "[Shadow Page](https://chromium.googlesource.com/chromium/src/+/f64293300707fd73569d833af8df2a6cd5b89e60/third_party/blink/renderer/core/exported/worker_shadow_page.h#26)" と呼んでいます。Document は本来ページを表示するためのもので、描画に関する多くの処理を伴います。しかしこれは Web Worker にとっては不要な処理であり、特に Web Worker 起動時のオーバーヘッドになる可能性が指摘されています。

![ワーカープロセスモデル](/images/javascript-parallel-processing-worker-process-models.png)

<p class='caption'>Web Worker のプロセスモデル。以前の記事より引用</p>

ネットワーク API の off-the-main-thread 化とは、Loader のコードを Frame から引き剥がし、WorkerGlobalScope から直接ネットワークリクエストを投げられるようにするものです。これにより、将来的に Shadow Page の仕組みをなくし、オーバヘッドを削減することが期待できます。 