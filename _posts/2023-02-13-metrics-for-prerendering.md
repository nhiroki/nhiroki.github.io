---
layout: post
title: "Speculation Rules API によるプリレンダリングのためのメトリクス設計"
date: 2023-02-13 00:00:00 +09:00
tags: web
image: /images/profile.png
math-expression: true
---

本記事では Speculation Rules API を使ったプリレンダリングの性能評価を行うためのメトリクスについて紹介します。

# はじめに

ウェブページの読み込みは本質的に時間のかかる処理です。ウェブブラウザは HTML ファイルを解析することでページ表示に必要なリソースを特定・収集・処理し、それらを組み合わせてページの描画（レンダリング）を行います。

ページ読み込みを高速化するアプローチの一つとして投機実行が知られています。投機実行はページ読み込みに必要な処理をあらかじめ実行しておくことで、実際にページを描画するときの処理を減らします。ウェブブラウザにはこのような投機実行を行うための API が多数実装されています。若干情報が古くなっていますが「[リソースの読み込みを助けるウェブブラウザ API の世界](/2021/05/06/resource-loading-apis)」という記事に投機実行のための API をまとめたので詳しくはそちらを見てください。

Chrome (Chromium) はこの投機実行を統一的に扱うための新しいインタフェースである [Speculation Rules API](https://github.com/WICG/nav-speculation/issues/) を仕様提案・実装しています。Speculation Rules API ではサーバからリソースを事前に取得する prefetch とページ全体を読み込んで JavaScript の事前実行まで行う prerender をサポートしています。

```
[Resolve DNS] -> [Connect TCP] -> [Fetch] -> [Load] -> [Render]
================================> prefetch
=====================================================> prerender
```

本記事では Speculation Rules API の使い方については説明しないので、Speculation Rules API についてまだあまり知らない方は「[Prerender pages in Chrome for instant page navigations](https://developer.chrome.com/blog/prerender-pages/)」を先に見てください。本記事では Speculation Rules API の基本知識を前提に、prerender の性能評価をどのように形式化して行うべきか紹介していきます。

ちなみに筆者は Speculation Rules API と prerender の開発を行っているチームの一員です。しかし、本記事は個人的な知見の整理を目的として書いており、その文責は私個人にあります。また本記事で紹介するメトリクスは一例であり、計測したいものに応じて適宜別のメトリクスを使い分けてください。

# 用語の整理

本題に入る前に用語について整理します。なおこれら用語は私の周辺で使われているものであり、広く一般的に使われているものとは違うかもしれません。特に統計用語についてはあまり自信がないので、間違いがあればこっそり教えてください。

## Navigation

Navigation とはあるページから別のページへ遷移することです。Single Page Application (SPA) のようなページ内遷移も navigation と呼びますが、ここではページをまたがる遷移についてのみ考えます。なお、このようなナビゲーションをページロードとも呼びます。

## Prediction

Prediction とはユーザが次に遷移するページ URL を予測することであり、何らかのヒューリスティクスで予測する機構のことを predictor と呼びます。predictor はブラウザ自身が持っている場合と、ウェブサイトの開発者が用意する場合があります。本記事ではそれぞれ browser predictor、website predictor と呼ぶことにします。ブラウザの URL バー (Omnibox) にはユーザ入力に応じて表示すべきページを予測して prefetch や prerender を行う機能がありますが、これは browser predictor の一例です。Chrome なら chrome://predictors から予測状況を見ることができます。

## Predictor Domain

Predictor domain とは各 predictor が予測する navigation の範囲です。例えば website predictor はウェブサイト内の navigation に対して予測を行い、URL バーからの navigation は関知しません。Predictor の予測性能について考えるときはその predictor が関知する domain 内の navigaton のみを考慮し、他の domain の navigation は考慮しません。また prerender がサポートしていない種類の navigation も domain から除外できます。例えば、本記事執筆時点では Speculation Rules は [cross-site navigation](https://github.com/WICG/nav-speculation/blob/main/prerendering-cross-site.md) や window.open() に対する prerender をまだサポートしていません。ブラウザ開発者は、Speculation Rules がサポートできる navigation の種類を増やすことで predictor domain を広げることを目指しています。

## Activation

Activation とは prerender 結果を使用した navigation のことです。理論上は prediction が当たって prerender されれば必ず activation が起きるように思えますが、実装上は様々な理由で prerender がキャンセルされることがあります。例えばメモリ使用量が逼迫していたり、ユーザが明示的に prerender を禁止している場合は予測が当たっていても prerender が実行されません。

```
[Prediction] --> [Prerendering] --> [Activation]
        (Prediction could fail)
                      (Prerendering could be canceled)
```

# メトリクス

本題であるメトリクスについて見ていきます。プリレンダリングのメトリクスは「ページの読み込み性能に関するメトリクス」と「ヒットレートに関するメトリクス」に大別することができます。

## ページの読み込み性能に関するメトリクス

プリレンダリングの目的はページ読み込みの高速化であり、その成果はどれだけページ読み込みを高速化したかで測られます。ページ読み込みのデファクトな指標は Core Web Vitals で、その中でも Largest Contentful Paint (LCP) が使われます。プリレンダリングを使うことで、この LCP の値が小さくなることが期待されます。

通常の navigation では LCP の値は navigation start と呼ばれるタイミング `T0` から LCP 要素の描画タイミング `T2` までの時間 `T(LCP) = t2 - t0` を表します。一方、activation では navigation start は事前実行が始まったタイミング `t-1 (< t0)` であり、そこからの時間で LCP を計測すると逆に LCP の値が大きく (`T(LCP) = (t2 - t-1) > (t2 - t0)`) なってしまいます。そこで LCP の計測基点を navigation start (prerender start) `t-1` ではなく activation start `t1` にずらして算出します。

```
Regular navigation:
        [Navigation Start (t0)]  ================================> [LCP (t2)]

Prerender:
[Navigation Start (t-1)]  ============ [Activation Start (t1)] ==> [LCP (t2)]
(Prerender Start)
```

Speculation Rules では Resource Timing API に新たに activationStart というタイミング情報を追加しており、activation の場合は LCP の計算を navigationStart ではなく activationStart に調整することで対応できます。詳しくは[こちらの記事](https://developer.chrome.com/blog/prerender-pages/#detecting-prerender-in-javascript)を参照してください。以下のコードは記事からの引用です。

```js
self.performance?.getEntriesByType?.('navigation')[0]?.activationStart;
```

なお Chrome User Experience Report (CrUX) や web-vitals ライブラリなどは[その調整を自動的に行なっています](https://developer.chrome.com/blog/prerender-pages/#measuring-performance)。

さて、上記のタイムラインから２つのことが分かります。

- Navigation start (prerender start) から activation start までの時間 (`t1 - t-1`) が長いほど、事前実行するための時間が増えるため LCP の値を小さくできます。この準備時間のことを lead time と呼ぶことにします。lead time を長く確保するには prediction と speculation rules の挿入をなるべく早く終わらせるべきです。
- Prerender があれば従来のページ読み込み最適化（例えば Priority Hints や Lazy Loading など）が不要に見えますがこれは間違いです。ページ読み込みが遅いページは長い lead time が必要になり、activation までにページ表示が間に合わない可能性があります。Prerender の導入と並行して通常時のページ読み込みの最適化も行うべきです。

## ヒットレートに関するメトリクス

前節では activation した場合の性能指標について見てきました。次に気になるのは「どのくらいの頻度で activation できているか」でしょう。この activation の実行頻度を hit rate と呼ぶことにします。Hit rate は次のように定式化できます。Predictor domain 内の navigation/activation のみを考慮する点に注意します。

$$
\text{Hit rate} =  \frac{\text{Number of activation in the domain}}{\text{Number of all the navigation in the domain}}
$$

さて、hit rate は定式化できましたが、これだと何をどうすれば hit rate が改善するのか分かりません。そこで、この式を recall と success rate に分解していきます。

Recall は domain 内の全 navigation のうち、正しく予測できた割合です。Recall はあくまでも予測できたかどうかを問題としており、予測の結果 prerender を実行したかは気にしません。

$$
\text{Recall} = \frac{\text{Number of correctly predicted navigation in the domain}}{\text{Number of all the navigation in the domain}}
$$

Success rate は正しく予測した navigation のうち、prerender が成功してキャンセルされなかった navigation の割合です。前述の通り、prerender は様々な理由でキャンセルされる可能性がありますが、それによってこの success rate は下がります。

$$
\text{Success rate} = \frac{\text{Number of successfully prerendered navigation in the domain}}{\text{Number of correctly predicted navigation in the domain}}
$$

Recall と success rate を使うと hit rate は次のように定式化できます。ここから hit rate を改善するには recall と success rate を上げれば良いことがわかります。

$$
\text{Hit rate} = \text{Recall} \times \text{Success rate}
$$

先に success rate の改善方法について検討していきます。Success rate は prerender のキャンセルによって下がります。ウェブサイトが speculation rules を消すことでもキャンセルが起きますが、それよりもブラウザの実装制約（リソース不足や非対応機能の使用など）やユーザセッティングによってキャンセルされることが多いです。そのためウェブサイト側からは改善するのが難しいメトリクスだと思います。かわりに私たちブラウザ開発者はブラウザによる制約をなるべく緩めようと取り組んでいます。また、DevTools 上にキャンセルされた理由を表示するようにし、ウェブサイト開発者がキャンセルされた理由を確認できるようにしています。もし success rate が理由で hit rate が伸び悩んでいる場合は、ぜひ開発チームまでお知らせください。

次に recall です。Recall はウェブサイト側から最も調整しやすいメトリクスですが、ここには様々なトレードオフの関係があるため最適値を決めるのは難しいです。Recall を上げる手っ取り早い方法は domain 内のあらゆる navigation に対して prerender を行うことです。閲覧者はそのうちのどれかに遷移するため recall を上げることができます。一方で、prerender は高コストなオペレーションのため、むやみやたらに prerender を行うのは推奨されません。不要なリクエストの増加によってサーバやネットワークに負荷がかかったり、ブラウザは負荷の上昇を検知すると自動的に prerender をキャンセルすることがあるため、その分 success rate が下がる可能性があります。これらを天秤にかけた上で、最適な predictor を設計・調整する必要があります。

このように一般的に website predictor を開発するのは難しいです。そこで、Speculation Rules によって prerender すべき URL を指定するのではなく、prerender しても良い URL パターンを指定することで、prerender するかどうかの判断をブラウザ (browser predictor) に任せる機能提案がされています。詳しくは[こちらの explainer](https://github.com/WICG/nav-speculation/blob/a01566b3722b23c027c70aa9e8e4146b1f5363ad/chrome-2023q1-experiment-overview.md#automatic-link-finding) を読んでください。

最後に各メトリクスの測り方について考えます。

Hit rate を測るには表示されたページが activate されたものか確認する必要がありますが、それは以下のコードで計測できます。`document.prerendering` はページがプリレンダリング中の場合は true を返します。`prerenderingchange` イベントは activation されるときに発火するイベントです。`prerenderingchange` イベントは activation 後にハンドラを登録しても発火しません。そこで activation がハンドラ登録よりも早く起きたときに備えて `activationStart` の値も確認しています。`activationStart` は `prerenderingchange` イベントが発火したタイミングで設定されます。

```js
if (document.prerendering) {
  document.addEventListener('prerenderingchange', e => {
    MarkAsActivated();
  });
} else if (self.performance?.getEntriesByType?.('navigation')[0]?.activationStart > 0)
  MarkAsActivated();
} else {
  MarkAsNotActivated();
}
```

Recall は navigation 先の URL を取得できれば良いので計測する方法は色々あると思います。例えば [Navigation API](https://developer.mozilla.org/en-US/docs/Web/API/Navigation_API) の navigate イベントで計測できます。

```
navigation.addEventListener('navigate', e => {
    if (predicted_urls.includes(e.destination.url)) {
        MarkAsCorrectlyPredicted();
    }
});
```

Prerender はブラウザによってウェブページへの通知なしでキャンセルされることがあるため、ウェブページが success rate を直接測定するのは難しいでしょう。かわりに hit rate と recall から逆算できます。

# まとめ

本記事では Speculation Rules API によるプリレンダリングのメトリクスについて紹介しました。

- ページの読み込み性能に関するメトリクスと改善方法
  - Activation 時のページ読み込みは activationStart を基点とした LCP で計測できる。
  - プリレンダリングに使える lead time を稼ぐために、prediction と speculation rules の挿入をできる限り早く行う。
  - 従来の最適化手法も実践し、ページ読み込み性能のベースラインも並行して上げる。
- ヒットレートに関するメトリクスと改善方法
  - Hit rate を上げるには recall と success rate を上げる必要がある。
  - ブラウザ開発者は success rate の向上に取り組んでいる。
  - ウェブサイト開発者は website predictor を調整することで recall を上げることができる。ただし無駄なプリレンダリングは避ける。
  - Website predictor を作る代わりに browser predictor に予測を任せる仕様提案がある。

紹介したメトリクスは一例であり、計測したいものに応じて様々なメトリクスを使い分けることが重要です。もし他のメトリクスを開発されたら是非教えて下さい。