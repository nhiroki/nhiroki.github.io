---
layout: post
title:  "Service Worker スクリプトのインストールと更新処理"
date: 2018-02-15 00:00:00 +09:00
tags: web serviceworker
image: /images/service-worker-install-and-update-scripts-updateViaCache.png
---

[Service Worker](https://github.com/w3c/ServiceWorker) の実装が主要ブラウザで揃い始めて盛り上がってきましたね。その流れに便乗して久しぶりに Service Worker の仕様や実装に関する記事を書いてみました。今回は Service Worker スクリプトのインストールと更新処理についてです。

この記事は Service Worker スクリプトを少しでも手書きして動かしたことがある人を想定読者にしています。Service Worker について全く知らない人はまず別の入門記事を参照してください。また、細かいことを気にせずに Service Worker を使いたい人は [Workbox](https://developers.google.com/web/tools/workbox/) といったライブラリやフレームワークの利用をおすすめします。

**更新履歴**

- 2018/06/07: [Chrome 68 から updateViaCache が使用可能](https://www.chromestatus.com/feature/6059838387781632)になりました。これに伴い Service Worker スクリプトの更新確認のデフォルト挙動が変更されています。それについて加筆しました。

# はじめに

Service Worker スクリプトはオフラインでも動作できるようにインストールされ、定期的に更新確認されます。[Workbox](https://developers.google.com/web/tools/workbox/) などのライブラリやフレームワーク経由で Service Worker を使っている場合はそれらが更新処理を隠蔽するので、その仕組みを意識する必要はほとんどありません。一方、Service Worker を手書きする場合は Service Worker スクリプトの更新に伴うリソースファイルの更新などを自前で行う必要があるので、更新処理がどのように行われるのかしっかり理解しておく必要があるでしょう。

ただ残念ながら Service Worker スクリプトの更新確認がどのように行われているのか詳細に解説した記事はほとんどないようです。また、このインストールや更新処理を Cache Storage API を使ったリソースキャッシュとごっちゃにされている方もいるようです。

そこで、本記事では Service Worker スクリプトのインストールと更新処理について詳しく解説します。読み終わった頃には次のことが理解できているはずです。

- Service Worker スクリプトの更新確認のタイミングと更新有無の判定方法
- Service Worker スクリプトの入れ替えタイミング
- リソースキャッシュの扱い

# Service Worker スクリプトとは？

Service Worker スクリプトとは ```register()``` で指定したメインスクリプトと、そこから ```importScripts()``` で読み込まれたスクリプトを指します。

```js
// [index.html] sw.js は Service Worker スクリプト
navigator.serviceWorker.register('sw.js');
```

```js
// [sw.js] imported-1.js と imported-2.js は Service Worker スクリプト
importScripts('imported-1.js', 'imported-2.js');
```

これ以外のウェブページ自体を構成するファイル (html, css, js, jpg など) をリソースファイルと呼ぶことにします。

Service Worker スクリプトはオフラインでも動作できるようにローカルに保存されます。これをインストールと呼びます。インストール先はブラウザキャッシュ (HTTP キャッシュ) や Cache Storage API とは異なる領域で、ウェブページからアクセスすることはできません。この領域について特定の名称はありませんが、本記事では便宜上「スクリプトストレージ」と表記することにします。スクリプトストレージへのインストールはブラウザが自動的に行います[^store-service-worker-scripts]。

[^store-service-worker-scripts]: Service Worker スクリプトがスクリプトストレージに保存されるのは Install イベントの終了後です。Install イベントの ```waitUntil()``` に rejected promise が渡された場合は保存されません。

サーバ側で Service Worker スクリプトが更新された場合はインストール済みのスクリプトも更新しなければいけません。さもないと、バグの混入したスクリプトや悪意のあるスクリプトがインストールされてしまった場合にサーバ側から正常状態へ復帰させることができません。Service Worker スクリプトの更新を検知するために、ブラウザは定期的にサーバへ更新確認します。

# Service Worker スクリプトの更新確認のタイミング

仕様では Service Worker がインストールされているページをユーザが訪れた時、前回のスクリプトの更新確認から 24 時間以上経過している場合は必ず更新確認をするように定義されています。それ以外にもブラウザが任意のタイミングで更新確認をすることが仕様上許可されています。例えば Chrome では Fetch イベントや Push イベントの発火などによって Service Worker が起動するときに更新確認しています[^delayed-update-check]。

[^delayed-update-check]: 更新確認は遅延実行されるので、更新確認によって Service Worker の起動が遅れることはありません。

また、[ServiceWorkerRegistration オブジェクト](/2015/07/05/service-worker-registration)の ```update()``` によって JavaScript から明示的に更新確認することができます。

```js
// ページ上で更新確認をする場合
// getRegistration() などで registration オブジェクトを取得する
const registration = await navigator.serviceWorker.getRegistration();
registration.update();
```

```js
// Service Worker スクリプト上で更新確認をする場合
self.registration.update();
```

# 更新確認のロジック

Service Worker スクリプトの更新チェックはスクリプトをバイト単位で比較して判定しています。1 バイトでも変更があれば「更新あり」と判定し、新たにインストールします。

> **CAUTION:** Chrome 64 時点では更新確認の実装に仕様に沿っていない部分があります。本来ならばバイト単位のチェックは Service Worker のメインスクリプトとそれからインポートされたスクリプトすべてについてそれぞれ行うべきですが、Chrome 64 の時点ではメインスクリプトしかバイト単位のチェックを行っていません ([Issue 648295](https://bugs.chromium.org/p/chromium/issues/detail?id=648295))。
>
> これにより、**importScripts() で読み込むスクリプトだけを更新する場合**には注意が必要です。例えば、次のような Service Worker スクリプト sw.js があるとします。このとき imported_script.js の中身だけ更新した場合、メインの sw.js はバイト単位で一致するため更新処理が走りません。
>
> ```js
> // [sw.js]
> importScripts('imported_script.js');
> ```
>
> これを回避するために、imported_script.js の更新時に sw.js のインポート URL に適当なパラメータを付けるというテクニックがあります。
> ```js
> // [sw.js]
> importScripts('imported_script.js?v=2');
> ```
>
> ただしこの方法は本来不要なメインスクリプトへの更新確認を引き起こすため、問題が解決された後は使わない方が良いと思います。

更新確認はブラウザキャッシュを経由する場合としない場合があります。

![Service Worker スクリプトの更新](/images/service-worker-install-and-update-scripts-updateViaCache.png)

まず前回のスクリプトの更新確認から 24 時間以上経過した場合はブラウザキャッシュを経由せず必ずサーバに聞きに行きます。これはバグや悪意のある Cache-Control ヘッダの設定によって Service Worker スクリプトが永続化されないようにするためです。

その他の場合でブラウザキャッシュを経由するかどうかは ServiceWorkerRegistration の updateViaCache フィールドの値によって変わります。これは ```register()``` のオプション引数で指定することができます。

```js
navigator.serviceWorker.register('sw.js', { updateViaCache: 'all' });
```

指定できる値は次の通りです。```all``` は Service Worker のメインスクリプトとインポートされたスクリプトの更新確認をブラウザキャッシュ経由で行います。```imports``` はインポートされたスクリプトの更新確認のみブラウザキャッシュ経由で行い、メインスクリプトの更新確認は直接サーバへ問い合わせます。```none``` はどちらのスクリプトであっても直接サーバへ問い合わせます。updateViaCache オプションが指定されていない場合は ```imports``` をデフォルト値として使用します。

```js
enum ServiceWorkerUpdateViaCache {
  "imports",
  "all",
  "none"
};
```

> **UPDATED(2018/06/07):** [Chrome 68 から updateViaCache が使用可能](https://www.chromestatus.com/feature/6059838387781632)になりました。これに伴いデフォルトの更新確認の挙動が変更されています。Chrome 68 以前の更新確認は必ずブラウザキャッシュを経由する ```all``` でした。Chrome 68 以降はインポートされたスクリプトの更新確認はブラウザキャッシュを経由しますが、メインスクリプトの更新確認はブラウザキャッシュを経由せずに直接サーバに問い合わせる ```imports``` になります。詳しくは Google が公開している「[
Fresher service workers, by default](https://developers.google.com/web/updates/2018/06/fresher-sw)」という記事を参照してください。
>
> **CAUTION(2018/02/15):** updateViaCache は比較的最近仕様に追加された機能であり、Chrome 64 時点ではまだ開発中の状態です ([Issue 675540](https://bugs.chromium.org/p/chromium/issues/detail?id=675540))。以前の仕様では Service Worker スクリプトに対する更新確認は必ずブラウザキャッシュを経由するようになっていたため、[Chrome 64 時点での実装はそのようになっています](https://www.chromestatus.com/feature/5897293530136576) (もちろん24 時間以上経過している場合はキャッシュをバイパスします)。この古い仕様において常になるべく新しい Service Worker スクリプトを使わせたい場合は、Service Worker スクリプトの Cache-Control ヘッダに no-store や max-age=0 を指定するというテクニックが使われていました。これをより柔軟に操作できるように導入されたのが updateViaCache オプションです ([Spec Issue](https://github.com/w3c/ServiceWorker/issues/893))。

ブラウザキャッシュを確認する場合はキャッシュされたレスポンスの Cache-Control ヘッダの値に従います。前述の通り、ブラウザは適当なタイミングで Service Worker スクリプトの更新確認を行いますが、これがサーバ負荷の原因となることがあります。その場合は updateViaCache オプションと Cache-Control ヘッダを適切に指定することで、サーバへの確認頻度を下げることができます。

更新確認に関して気をつけるべき点はリソースファイルのキャッシュの扱いです。更新確認が行われるのは Service Worker スクリプトだけであり、Cache Storage などに保存したリソースファイルは更新確認されません。それらは新しくインストールされた Service Worker スクリプトの Install イベントや Activate イベントなどで明示的に更新確認をする必要があります。

```js
// [sw.js]
self.addEventListener('install', e => {
  // Cache Storage にリソースをキャッシュし直す
  e.waitUntil(PopulateResourcesInCacheStorage());
});

self.addEventListern('activate', e => {
  // 不要なリソースを Cache Storage から削除する
  e.waitUntil(DeleteOutdatedResourcesInCacheStorage());
});
```

> **NOTE:** Cache Storage は歴史的な理由[^cache-storage-history]により Service Worker の仕様内で定義されていますが、実際には Service Worker から独立した機能で、他のストレージ API (localStorage や IndexedDB) などと扱いは一緒です。Service Worker の処理に伴って Cache Storage のデータがブラウザによって勝手に書き換えられたりすることはありません。

[^cache-storage-history]: Cache Storage が Service Worker の仕様と一緒に策定された理由は、オフライン機能の実現のために、リクエストとレスポンスを Key-Value Store のように手軽に保存できて、かつ Opaque レスポンスを格納できるストレージが必要だったからです。Opaque レスポンスはボディの中身が読めないため、IndexdedDB などに保存することができません。

本番環境ではブラウザキャッシュを有効にしたくても、開発環境では修正を即座に反映させるためにキャッシュを無効にしたくなることも多いと思います。開発環境では updateViaCache を none にすることでも実現できますが、デプロイ時に再度有効にする必要があり、バグの原因になりそうです。その場合、DevTools の Network タブにある "Disable cache" 機能を有効にすると、ネットワークリクエスト時に必ずブラウザキャッシュをバイパスするようになります。

![Disable cache](/images/service-worker-install-and-update-scripts-disable-cache.png)

# 新しい Service Worker スクリプトが実際に使われるタイミング

更新処理が終わった後に、すぐに新しい Service Worker スクリプトが既存の Service Worker スクリプトと入れ替わって動作するわけではありません。Service Worker の入れ替えは稼働中の Service Worker がコントロールしているすべてのページが閉じられたときです。コントロールされているページの有無に限らず、強制的に新しい Service Worker に入れ替えさせる ```skipWaiting()``` という機能もあります。ページコントロールについては「[Service Worker のスコープとページコントロール](/2015/02/28/service-worker-scope-and-page-control)」、```skipWaiting()``` については「[Service Worker の Registration](/2015/07/05/service-worker-registration)」という記事で紹介しているのでそちらを参照してください。

開発時のハマりポイントの一つが、ブラウザのリロードボタンを押したのに Service Worker スクリプトが入れ替わらないというものです。ナビゲーションの仕組み上、通常のリロードでは Service Worker は入れ替わりません。ハードリロードをした後にリロードをすると入れ替わります。この手間を避けるには、DevTools の Applications タブの "Service Workers" メニューにある "Update on reload" という機能を使うのがおすすめです。これを有効にすると、ページリロード時に Service Worker スクリプトを更新し、新しい Service Worker スクリプトにページをコントロールさせます。

![Update on reload](/images/service-worker-install-and-update-scripts-update-on-reload.png)

# まとめ

本記事をざっくりまとめると次のようになります。

- Service Worker スクリプトの更新確認のタイミング
  - Service Worker の起動時
  - ```update()``` を呼んだ時
- Service Worker スクリプトの更新有無の判定方法
  - バイト単位で比較を行う (ただし Chrome 64 時点ではバグがある)
  - ブラウザキャッシュをパイパスする場合としない場合がある
    - 前回更新から 24 時間以上経過した場合はバイパスする
    - それ以外は updateViaCache オプションに従う
- Service Worker スクリプトの入れ替えタイミング
  - すべてのコントロールされているページが閉じられたとき
  - ```skipWaiting()``` を呼んだ時
  - リロード時は入れ替わらない点に注意
- リソースキャッシュの扱い
  - Service Worker スクリプトの更新確認とは無関係
  - Install イベントや Activate イベントなどで明示的に行う

# 謝辞

本記事の草稿を [@kinu](https://twitter.com/kinu) さんに確認していただきました。ありがとうございます。なお、文責はすべて私 (nhiroki) にあります。

# 注釈