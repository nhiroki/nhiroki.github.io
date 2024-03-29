---
layout: post
title: "JavaScript のスクリプトインポートを正しく使い分けようという話"
date: 2018-09-07 00:00:00 +09:00
tags: web
image: /images/profile.png
---

JavaScript の文脈で「スクリプトをインポートする」といった場合、色々な可能性が考えられます。現場での混乱を避けるためにも用語を正しく使い分ける必要があります。そこで本記事では JavaScript のスクリプトインポートについて整理します。

**更新履歴**

- 2019/12/05
  - Dedicated Worker の ES Modules サポートについて追記しました。
- 2018/09/08
  - Worklet の type とその上での dynamic import について追記しました。
  - Service Worker 上での importScripts について追記しました。

# Classic Script と Module Script

スクリプトインポートを理解するには、スクリプトについて正確に理解する必要があります。HTML の仕様では、[スクリプト](https://html.spec.whatwg.org/multipage/webappapis.html#concept-script)には [Classic Script](https://html.spec.whatwg.org/multipage/webappapis.html#classic-script) と [Module Script](https://html.spec.whatwg.org/multipage/webappapis.html#module-script) の二種類があると定義されています。Classic Script は従来型のスクリプトで、Module Script は ES Modules のことです。ES Modules については詳しく解説した記事がたくさんあるので、本記事では特に説明しません。

スクリプトの種別は \<script\> タグの type 属性で指定します。[Chrome ではバージョン 61 から Module Script をサポート](https://www.chromestatus.com/feature/5365692190687232)しています。

```html
<script>
// Classic Script
</script>

<script type='module'>
// Module Script
</script>
```

Web Worker の場合はコンストラクタオプションの type で指定します。type を省略した場合は Classic Script になります。Classic Script を使った Worker を Classic Worker、Module Script を使った Worker を Module Worker と呼びます。

```js
const classic_worker = new Worker('worker.js', { type: 'classic' });
const module_worker = new Worker('worker.js', { type: 'module' });
```

<s>Module Worker は Chrome 69 の時点で実装済みですが、まだデフォルトで有効化されていません](https://www.chromestatus.com/feature/5761300827209728)。試すには chrome://flags から --enable-experimental-web-platform-features フラグを有効にする必要があります。</s> 

**UPDATED(2019/12/05):** Chrome 80 から Dedicated Worker で Module Script が使えるようになりました。Shared Worker ではまだ使えません。詳しくは「[Chrome 80 から Web Worker (Dedicated Worker) で ES Modules が使えます](/2019/12/05/es-modules-for-dedicated-workers)」を見てください。

Service Worker の場合は register オプションの type で指定します。type を省略した場合は Classic Service Worker になります。[Module Service Worker は Chrome 70 の時点では未実装の機能](https://bugs.chromium.org/p/chromium/issues/detail?id=824647)です。

```js
const reg1 = await navigator.serviceWorker.register('sw.js', { type: 'classic' });
const reg2 = await navigator.serviceWorker.register('sw.js', { type: 'module' });
```

Worklet は Module Script オンリーの機能なため、type を指定する方法はありません。[Chrome ではバージョン 65 から (Paint) Worklet をサポート](https://www.chromestatus.com/feature/5275637463908352)しています。

```js
const result = CSS.paintWorklet.addModule('worklet.js');
```

\<script\> タグや new Worker() などによって読み込まれたスクリプトはトップレベルスクリプトと呼ばれます。

# インポート方法の違い

スクリプトをインポートする方法は三つあります。

## Static import

Static import は Module Script のインポートを静的に行います。トップレベルモジュールスクリプトのロード時にそれに含まれるすべての static import 文が列挙され、モジュールグラフを構築します。このグラフ内のスクリプトがすべてロードされた時点でトップレベルスクリプトを実行します。実行前にすべてのモジュールの解決を行うため static import と呼ばれています。

```html
<!-- index.html -->
<script src='top-level.js' type='module'></script>
```

```js
// top-level.js
import * as module1 from './descendant1.js';  // static import
import * as module2 from './descendant2.js';  // static import
module1.Foo();
module2.Bar();

// descendant1.js
export function Foo() { ... };

// descendant2.js
import * as module from './descendant3.js';  // static import
export function Bar() { module.Baz() };

// descendant3.js
export function Baz() { ... };
```

これを図解したのが次の図です。

![static import](/images/javascript-import-static-import.png)

## Dynamic import

Dynamic import は Module Script のインポートを行う関数ライクな仕組みです。[Chrome ではバージョン 63 からサポート](https://www.chromestatus.com/feature/5684934484164608)しています。スクリプトの実行中に Module Script を動的にインポートしたいときに使います。Dynamic import は Module オブジェクトで解決される Promise を返します。

```html
<!-- index.html -->
<script src='top-level1.js' type='module'></script>
```

```js
// top-level1.js
import * as module1 from './descendant1.js';
import * as module2 from './descendant2.js';

// descendant2.js
import * as module from './descendant3.js';
import('./top-level2.js').then(module => module.Hoge());  // dynamic import

// top-level2.js
import * as module1 from './descendant4.js';
import * as module2 from './descendant5.js';
export function Hoge() { ... };
```

Dynamic import はスクリプトを新しいトップレベルモジュールスクリプトとして読み込みます。インポートされたスクリプト内の全ての static import 文を列挙して新しくモジュールグラフを構築し、全スクリプトがロードされた時点で実行します。

![dynamic import](/images/javascript-import-dynamic-import.png)

Dynamic import は Classic Script でも使えます。

```html
<!-- index.html -->
<script>
import('./top-level.js').then(module => module.DoSomething());
</script>
```

Worklet は Module Script として動作しますが、dynamic import を使うことはできません。Worklet では不確定性のある機能が無効化されていて、dynamic import を含む一切のネットワーク API が使えません。static import は実行前に解決されるので使えます。その理由は以前「[JavaScript のスレッド並列実行環境 ― Worklet の実行モデル](https://nhiroki.jp/2017/12/10/javascript-parallel-processing#6-worklet)」という記事で紹介したのでそちらを見てください。

## importScripts

importScripts は Classic Worker 上で Classic Script をインポートする機能です。Worker 上では \<script\> タグが使えないため、そのかわりに importScripts によるスクリプト読み込みがサポートされています。

importScripts はスクリプト実行時に同期的にスクリプト読み込みを行います。importScripts は Module Scripts のように名前空間を分けてスクリプトを読み込むのではなく、\<script\> タグのように現在の名前空間にベタッとスクリプトを貼り付けたように動作します。

```html
<script>
const classic_worker = new Worker('worker.js', { type: 'classic' });
</script>
```

```js
// worker.js
importScripts('descendant1.js', 'descendant2.js');
Say('Hello, world!');  // 'Hello, world!'

// descendant1.js
function Say(str) { console.log(str); }
```

Module Worker 上で importScripts を呼ぶと、TypeError 例外を投げます。

```html
<script>
const module_worker = new Worker('worker.js', { type: 'module' });
</script>
```

```js
// worker.js
try {
  importScripts('descendant1.js', 'descendant2.js');
} catch (e) {
  console.log(e.name);  // TypeError
}
```

Classic Service Worker では importScripts を呼べるタイミングに制限があります。詳しくは「[Service Worker 上での未インストールスクリプトに対する importScripts()](/2018/08/15/service-worker-import-scripts-after-installation)」という記事を見てください。

# スクリプト種別毎の利用可能なインポート

Classic Script と Module Script でスクリプトのインポート方法が違うことを説明しました。最後にその違いを表にまとめてみました。

| | static import | dynamic import | importScripts |
| :--- | :--- | :---| :--- |
| classic | x | o | x |
| classic worker | x | o | o |
| module | o | o | x |
| module worker | o | o | x |
| module worklet | o | x | x |

- Classic Script では dynamic import が使えます。また Classic Worker Script であれば importScripts も使えます。
- Module Script では static import と dynamic import が使えます。Module Worker Script では importScripts は使えません。
- Worklet は Module Script として動作しますが、その実行モデルの制限により dynamic import は使えません。

# まとめ

本記事では JavaScript のスクリプトインポートについてまとめました。スクリプトインポートには static import, dynamic import, importScripts がありますが、それぞれ挙動 (static or dynamic) や使える場面 (Document or Worker) が異なります。状況に応じて適切なものを使いましょう。