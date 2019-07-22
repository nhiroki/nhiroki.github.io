---
layout: post
title: "論文「Reconsidering Custom Memory Allocation」(OOPSLA 2002)"
date: 2019-07-22 00:00:00 +09:00
tags: paper
image: /images/paper-reaps-region-mode-and-heap-mode.png
---

「[Reconsidering Custom Memory Allocation](https://dl.acm.org/citation.cfm?id=582421)」という論文を読んだのでその紹介です。本論文は OOPSLA (Object-Oriented Programming, Systems, Languages, and Applications) 2002 に採択されています。

2002 年の論文なので、本記事を書いている 2019 年時点では状況が変わっている可能性があることに注意してください。また私が読み間違えている可能性があります。正確な情報が欲しい方は必ず論文を読んでください。誤りの指摘や補足、議論などは [GitHub Issue](https://github.com/nhiroki/nhiroki.github.io/issues) や [Twitter](https://twitter.com/nhiroki_) へお願いします。

# 読んだ動機

別の論文「A Locality-Improving Dynamic Memory Allocator (MSP 2005)[^vam]」を読んでいたところ、今回紹介する論文が関連研究に挙がっていたから。

[^vam]: 「A Locality-Improving Dynamic Memory Allocator」を読み始めた理由は、Microsoft が 2019 年に公開したメモリアロケータ Mimalloc が、最も影響を受けた論文として挙げているため。

# 論文概要

特定のアプリケーションに最適化したメモリ割り当てを行うカスタムアロケータは広く使われている。また、プログラミングに関する書籍などでもその有用性が広く紹介されている。本論文ではカスタムアロケータを使った 8 個のアプリケーションを検証し、その有用性について改めて調査を行った。そのうち 6 個において (2002 年当時の最先端で) 汎用的なメモリアロケータである DLmalloc が、カスタムアロケータと同等以上のパフォーマンスを出すことが分かった。DLmalloc を凌駕した残りの 2 つは region-based なカスタムアロケータを使っていた。

Region-based メモリアロケータは、メモリ領域を前から順番に切り出してオブジェクト割り当てを行い、不要になったらそのメモリ領域全体を一気に解放する。切り出したオブジェクトは個別に解放することができない。短期間にたくさんのオブジェクトを割り当ててそれらを一気に解放するようなワークロードでは有効だが、一部のオブジェクトが長寿命になるワークロードでは領域全体が生かされるためメモリ使用量が高止まりするという弱点がある。このため、region-based メモリアロケータを汎用的に使うのは難しい。本論文では intensive memory reuse, unbounded buffers, dynamic arrays, producer-consumer pattern が適応が難しい例として挙げられている。

これに対し、本論文では region-based メモリアロケータと汎用的な heap-based メモリアロケータの両方の性質を兼ね備えた Reaps というメモリアロケータを提案している。Reaps は region による割り当てをサポートしつつ、個別オブジェクトの解放にも対応している。ベンチマーク評価により、Reaps は region-based カスタムアロケータに近い性能を出し、汎用的なメモリアロケータとして使った場合も多くのベンチマークで DLmalloc に近い性能を出せることが分かった。

システムアロケータの性能が不十分な場合はカスタムアロケータを実装するのではなく、汎用性を求めるなら DLmalloc、region を使いたいなら Reaps を使うべきと論文著者らは結論づけている[^conclusion]。

[^conclusion]: "We feel that programmers who find their system allocator to be inadequate should try using reaps or the Lea allocator rather than writing a custom allocator."

# Region-based メモリアロケータ

前提知識として、本論文内で挙げられている region-based メモリアロケータの特徴を以下に整理する。

- 領域をまとめて解放するため、長寿命オブジェクトがあると領域全体が生かされて不要なオブジェクトの解放が遅れる。一方、まとめて解放することでオブジェクトを個別に解放するよりもメモリリークが起きにくい。
- 領域単位でのメモリ解放となるため、通常の malloc / free というインタフェースではなく、malloc / freeAll のようなインタフェースになる。そのため汎用的なメモリアロケータと入れ替えて使うのが難しい。
- 領域分割されるため、マルチスレッディングでのメモリ空間の isolation として使いやすい。
- 実装には bump pointer[^bump-pointer] が使われることが多い。

[^bump-pointer]: bump pointer については[以前の記事](/2019/07/08/paper-snmalloc-a-message-passing-allocator)で解説した。

# Reaps の設計と実装

本論文では Reaps と呼ばれる新たなメモリアロケータを提案している。Reaps は region-based メモリアロケータと general-purpose (heap-based) メモリアロケータ両方の特性を活かせるように設計実装されている。Reaps という名称について明確には書かれていないが、恐らく REgions と hEAPs に由来していると思われる。

Reaps の特徴は以下の通り。

- Region-based でありながら、個別オブジェクトの解放にも対応。Region-based メモリアロケータの欠点であった region 内不要オブジェクトによるメモリ消費量の高止まりを抑えることができ、汎用的なメモリアロケータとしても使用できる。
- ネストを含んだあらゆる region セマンティクスをサポート。
- シングルスレッド専用。マルチスレッドメモリアロケータである Hoard への統合を計画している。

heap-based allocation では malloc と free, region-based allocation では malloc と freeAll が必要になる。両者の特性を持つ reaps では malloc, free, freeAll 全ての API をサポートする。具体的な関数シグネチャは次の通り。

```c
void reapCreate (void ** reap, void ** parent);
void reapDestroy (void ** reap);
void reapFreeAll(void ** reap);
void * reapMalloc (void ** reap, size_t size);
void reapFree (void ** reap, void * object);
```

reapCreate で region を生成し、reapDestroy でそれを破棄する。parent を指定することで nested region をサポートしている。reapMalloc は指定した region からオブジェクトの割り当てを行い、reapFree でそれを解放する。reapFreeAll は指定した region 全体を解放する。

Reaps のメモリアロケーションは次のとおりである。まず、大きなメモリチャンクを bump pointer で割り当てる。これを region mode と呼ぶ。一般的な region とは異なり、各オブジェクトはメタデータを含むヘッダー (boundary tag) を持つ。これは heap としてオブジェクトを管理するのに用いられる。

Reaps は reapFree によってオブジェクトが個別に解放されるまで region mode で動く。reapFree で解放されたオブジェクトは region に紐付いた heap (associated heap) に加えられ、以後は heap に返却されたオブジェクトを使い切るまで heap からメモリ割り当てが行われる。heap を使い切ったらまた region mode に戻る。

![Reaps region mode and heap mode](/images/paper-reaps-region-mode-and-heap-mode.png)

<p class='caption'>Reaps によるメモリ割り当てと解放の例。reapMalloc で割り当てられたオブジェクト x1-x4 はそれぞれ metadata を持つ。reapFree で解放されたオブジェクト x3 は associated heap に移され、以後のオブジェクト割り当てのモードを region から heap に切り替える。論文 Figure 4 より引用。</p>

詳細は調べていないが、Reaps は HeapLayers[^heap-layers] と呼ばれる C++ テンプレートを使ったカスタムメモリアロケータインフラの上で実装されている。設計や実装はこの HeapLayers に依存しているので、細かいことが知りたい人は論文を読んでください。

[^heap-layers]: 「Composing high-performance memory allocators (PLDI 2001)」参照。本論文と同じ人達が発表している。

# 評価

割愛。今回は Reaps の仕組みに興味があったので、評価については流し読みしただけ。あと 2002 年時点での評価結果なので現在ではあまり意味がないと思います。気になる人は論文を読んでください。

ランタイム性能の評価についてだけざっくりいうと、Reaps は region-based カスタムアロケータに近い速度を出しており、汎用的なメモリアロケータとして使った場合も多くのベンチマークで DLmalloc に近い性能を出していることが分かった。

# 雑感

- Region-based なメモリアロケータって現在どれくらい使われているのか気になった。論文内で region-based メモリアロケータの使用例として紹介されている Apache ではまだ使っている？Region-based メモリアロケーションに含めて良いのか分からないけど、自分は C/C++ の alloca() によるスタックアロケーションや Objective-C の autorelease pool を使ったことがあるくらい。
- Reaps のメモリ管理から region 単位での解放機能 (freeAll) や boundary tag を除外すると、モダンなアロケータで使われている bump pointer と free list を組み合わせたメモリ管理機構に近くなると感じた。この辺りの研究がその源流になっている？
- freeAll をマルチスレッド環境でやるのは困難 (リモートオブジェクトの解放どうするの？) なので、region をそのままマルチスレッド対応するのは厳しいはず。メモリリークしにくいという利点は失うが、メモリ空間の isolation は per-thread heap にした方が良いと思う。

# 注釈