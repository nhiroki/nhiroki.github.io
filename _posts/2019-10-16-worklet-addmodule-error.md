---
layout: post
title: "Chrome 79 から Worklet.addModule() が詳細なエラーを返すようになった話"
date: 2019-10-16 00:00:00 +09:00
tags: web
image: /images/profile.png
---

Chrome 79 から Worklet が addModule() を失敗したときにより詳細なエラーを返すようになります。本記事はその紹介です。

# 今まで (Chrome 78 以前)

Worklet[^worklet] に module script を読み込むときは [addModule()](https://developer.mozilla.org/en-US/docs/Web/API/Worklet/addModule) という関数を使います。例えば [Paint Worklet](https://developers.google.com/web/updates/2018/01/paintapi) の場合は次のようになります。

```js
window.CSS.paintWorklet.addModule('worklet.js');
```

addModule() は promise を返します。成功した場合は undefined で resolve された promise が、失敗した場合は Error オブジェクトで reject されます。

```js
window.CSS.paintWorklet.addModule('worklet.js')
    .then(result => assert_equals(result, undefined))
    .catch(error => assert_equals(error.name, 'AbortError'));
```

addModule() が reject される原因はいくつかあります。例えば、module script の fetch に失敗した場合や、文法ミスで parse に失敗した場合に reject されます[^addmodule-rejection]。

Chrome 78 以前では失敗原因に関わらず常に AbortError で reject されていました。これは実装レベルの問題ではなく仕様レベルで規定されていた挙動で、実際の失敗原因が全く通知されないためアプリケーションのデバッグを著しく困難にしていました。

# これから (Chrome 79 以降)

Chrome 79 からは失敗原因に沿った Error オブジェクトで reject するようになりました。

Worklet script に文法ミスがある場合は SyntaxError で reject されます。

```js
// syntax-error-worklet.js
1+;

// index.html
window.CSS.paintWorklet.addModule('syntax-error-worklet.js')
    .then(result => assert_not_reached())
    .catch(error => assert_equals(error.name, 'SyntaxError'));
```

Static import に指定した識別子が不適合な場合は TypeError で reject されます。

```js
// invalid-specifier-worklet.js
import 'invalid';

// index.html
window.CSS.paintWorklet.addModule('invalid-specifier-worklet.js')
    .then(result => assert_not_reached())
    .catch(error => assert_equals(error.name, 'TypeError'));
```

Module script の fetch に失敗した場合は引き続き AbortError で reject されます。


```js
window.CSS.paintWorklet.addModule('non-existent-worklet.js')
    .then(result => assert_not_reached())
    .catch(error => assert_equals(error.name, 'AbortError'));
```

Worklet の仕様変更は[こちら](https://github.com/w3c/css-houdini-drafts/pull/958)と[こちら](https://github.com/w3c/css-houdini-drafts/pull/967)で、Chrome の実装変更は [crbug.com/782066](https://crbug.com/782066) で確認できます。Chrome Platform Status の[エントリ](https://www.chromestatus.com/feature/5116796497559552)もあります。

# 技術的な話

この問題は Worklet 実装初期から知られていましたが、ECMAScript の定義する [Native Error](https://tc39.es/ecma262/#sec-native-error-types-used-in-this-standard) オブジェクト (要するに SyntaxError や TypeError) が clonable ではなかったために実現できていませんでした。

Worklet はメインの実行コンテキスト[^execution-context]である Window から独立した実行コンテキスト WorkletGlobalScope で実行されます。Worklet の script fetch や parse は WorkletGlobalScope で行われ、Native Error オブジェクトはそれに紐付いた形で生成されます。一方、addModule() を呼んだのは Window なので、promise を reject するために Native Error オブジェクトを WorkletGlobalScope から Window へ渡す必要があります。これは JavaScript の実行コンテキストをまたぐため clonable であるべきですが、仕様的 に Native Error オブジェクトを clone することはできませんでした。

最近 yhirano さんが clonable にする変更を [HTML 仕様](https://github.com/whatwg/html/pull/4665)と [Chrome 実装](https://crbug.com/970079)の両方に加えたためこれが実現できるようになりました。素晴らしい！

# まとめ

Chrome 79 から Worklet.addModule() がより詳細なエラーを返すようになりました。Worklet を使ったアプリケーションを開発するときに役立つはずです。

[^worklet]: Worklet については「[JavaScript のスレッド並列実行環境 - 6. Worklet](/2017/12/10/javascript-parallel-processing#6-worklet)」を参照。
[^addmodule-rejection]: addModule() が reject されるのは script fetch や parse の間にエラーが起きたときだけで、スクリプト評価時のエラー (たとえば dynamic import の失敗) などは addModule() を reject しません。
[^execution-context]: 実行コンテキストについては「[JavaScript のスレッド並列実行環境 - 2. Web Worker](/2017/12/10/javascript-parallel-processing#2-web-worker)」を参照。