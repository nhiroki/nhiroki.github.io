---
layout: post
title: "Service Worker 上での未インストールスクリプトに対する importScripts()"
date: 2018-08-15 00:00:00 +09:00
tags: web
image: /images/profile.png
---

Chrome 69 時点での Service Worker は、InstallEvent 後にインストールされていないスクリプトに対して importScripts() を呼ぶことができますが、これは仕様に沿っていません。そこで、これを廃止しようという提案が Blink の開発者メーリングリストに出されています。

- [Intent to Deprecate and Remove: importScripts() of new scripts after service worker installation](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/a6P-niHWgF4)

Firefox と Edge は既に仕様通りに実装されており、未インストールスクリプトに対する importScripts() はエラーを返すようになっています。一方、Safari は Chrome と同じ挙動をしています ([Bug Issue](https://bugs.webkit.org/show_bug.cgi?id=188495))。

本記事では、この変更がどのような背景で行われ、どのような影響を及ぼすのかざっくり解説します。

**更新履歴**

- 2019/02/06: Chrome 71 で廃止されました ([Remove importScripts() of new scripts after service worker installation](https://www.chromestatus.com/feature/5748516353736704))。

# 前提知識

**Service Worker スクリプト**

register() で指定したメインスクリプトと、そこから importScripts() で読み込まれたスクリプトを Service Worker スクリプトと呼びます。

```js
// [index.html] service-worker.js は Service Worker スクリプト
navigator.serviceWorker.register('service-worker.js');
```

```js
// [sw.js] imported-1.js と imported-2.js は Service Worker スクリプト
importScripts('imported-1.js', 'imported-2.js');
```

Service Worker スクリプトはオフラインでも動作できるようにローカルに保存されます。これを「インストール」と呼びます。このあたりは以前「[Service Worker スクリプトのインストールと更新処理](/2018/02/15/service-worker-install-and-update-scripts)」という記事に詳しく書いたので、そちらも見てください。

**ライフサイクル**

Service Worker は固有のライフサイクルを持ちます。Service Worker の登録が行われるとまずスクリプトの実行が行われ、各種イベントハンドラーの登録を行います。その後 InstallEvent, ActivateEvent の順に発火し、FetchEvent や PushEvent などは ActivateEvent 後に発火するようになります。

```js
// [service-worker.js]
// Phase 1: Evaluate code snippets on the top-level scope,
DoSomething();
// and register event handlers.

// Phase 2: Fire an InstallEvent to cache resources.
oninstall = e => { ... };

// Phase 3: Fire an ActivateEvent to clean up old cached resources.
onactivate = e => { ... };

// Phase 4: Fire various event handlers.
onfetch = e => { ... };
onpush = e => { ... };
```

ライフサイクルについては Google が公開している「[Service Worker の紹介 - Service Worker のライフサイクル](https://developers.google.com/web/fundamentals/primers/service-workers/?hl=ja#service_worker_1)」という記事も参考になります。

# どう変わるのか？

廃止前は importScripts() を任意のタイミングで呼び出すことができました。ただし、Service Worker スクリプトの一部としてインストールされるのは InstallEvent までにインポートされたスクリプトのみです。

具体例を見てみましょう。次のコードの場合、installed-1.js と installed-2.js は Service Worker スクリプトとしてインストールされるのでオフライン時にもインポートできます。一方、not-installed-1.js と not-installed-2.js は InstallEvent 後に呼ばれているためインポートはされますがインストールはされません。そのため、オフライン時は not-installed-1.js と not-installed-2.js のインポートに失敗します。

```js
// [service-worker.js]
// importScripts() on the top-level scope.
// This will be installed as a service worker script.
importScripts('installed-1.js');

// importScripts() on the InstallEvent.
// This will be installed as a service worker script.
oninstall = e => importScripts('installed-2.js');

// importScripts() after installation.
// These won't be installed as service worker scripts.
onactivate = e => importScripts('not-installed-1.js');
onfetch = e => importScripts('not-installed-2.js');
```

廃止後は InstallEvent 後の importScripts() は NetworkError を返すようになります。

```js
// [service-worker.js]
// importScripts() on the top-level scope.
// This will be installed as a service worker script.
importScripts('installed-1.js');

// importScripts() on the InstallEvent.
// This will be installed as a service worker script.
oninstall = e => importScripts('installed-2.js');

// importScripts() after installation.
// These throw NetworkErrors.
onactivate = e => importScripts('not-installed-1.js');  // NetworkError
onfetch = e => importScripts('not-installed-2.js');     // NetworkError
```

ただし、インストール済みのスクリプトに対しては InstallEvent 後であっても importScripts() を呼ぶことができます。

```js
// [service-worker.js]
// importScripts() after installation.
// These successfully import the installed scripts.
onactivate = e => importScripts('installed-1.js');
onfetch = e => importScripts('installed-2.js');
```

# 対応方法

使用するスクリプトはすべて InstallEvent までにインポートし、インストールしましょう。

常に最新のスクリプトをインポートしたい場合は、Service Worker の更新処理機構に頼るようにしましょう。Service Worker の登録時に { updateViaCache: 'none' } オプションを指定することで、インポートされたスクリプトの更新確認頻度を上げることができます。ただし、Chrome 69 時点ではインポートされたスクリプトの更新確認処理に仕様に沿っていない部分があるためうまく動作しません。これのワークアラウンドについては「[Service Worker スクリプトのインストールと更新処理](/2018/02/15/service-worker-install-and-update-scripts)」を参照してください。

# なぜ未インストールスクリプトのインポートが禁止されているのか？

性能上の理由です。

- Service Worker はリソースリクエストのクリティカルパスになりえるため、importScripts() を除いたすべての同期的 API が禁止されています。importScripts() の利用もアクティブな Service Worker 上では極力避けるべきです。特に未インストールスクリプトのインポートはネットワークリクエストを伴うためコストが大きくなります。
- スクリプトをインストールすると、必要なスクリプトの先読みや V8 によるコンパイル済みコードのキャッシュといった最適化を行えます。このあたりの最適化については詳しい記事があるのでそちらを参照してください。
  - 「[ServiceWorkerがちょっとずつスピードアップしてる話 - Script Streaming](https://qiita.com/amiq11/items/016d00364a2cfe05f040#script-streaming)」
  - 「[ChromiumにおけるJavaScriptのコードキャッシュについて](https://qiita.com/horo/items/28ebd011b31f580a410a)」

あと、禁止することによってレンダリングエンジンの実装をシンプルにすることができます。

# まとめ

InstallEvent 後に未インストールスクリプトに対する importScript() 呼び出しが禁止されます。使用するスクリプトはすべて InstallEvent までにインポートしましょう。