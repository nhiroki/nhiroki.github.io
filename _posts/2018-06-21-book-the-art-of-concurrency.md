---
layout: post
title: "『並行コンピューティング技法』読了"
date: 2018-06-21 00:00:00 +09:00
tags: book
image: /images/profile.png
---

『[並行コンピューティング技法 ― 実践マルチコア/マルチスレッドプログラミング](https://www.oreilly.co.jp/books/9784873114354/)』の読書メモです。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">『並行コンピューティング技法 ― 実践マルチコア/マルチスレッドプログラミング』を読み始めた <a href="https://t.co/dDpzcXtCnf">https://t.co/dDpzcXtCnf</a> <a href="https://twitter.com/hashtag/nhbk?src=hash&amp;ref_src=twsrc%5Etfw">#nhbk</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1002361216138805248?ref_src=twsrc%5Etfw">2018年6月1日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# 読書メモ

- 第 1 章「速くしたい人、手を挙げて！」
  - 並行処理に関する諸概念やモデルの説明。並行と並列、スレッド化へのステップ、並列アルゴリズムの理論モデル (PRAM 並列ランダムアクセスマシン、メモリの排他並行・参照更新モデル)、共有メモリと分散メモリ、生産者消費者モデル、RW ロックなど。
- 第 2 章「並行か非並行か？それが問題だ」
  - タスク分解とデータ分解を中心に並行アルゴリズム設計の話。タスク数とスレッド数、タスク粒度とオーバヘッド、データ依存、タスクの静的・動的スケジューリング、二次元配列を使ったデータ分割の例、状態依存を持つループ処理の依存の切り方など。
  - Boss-Worker と Producer-Consumer の違いが分かってなかったけど、P-C が共通のコンテナを介してタスクをやり取りするのに対し、B-W はメッセージングなどでボスにタスクを要求するモデルなのか。第一章に書いてあるのに読み流してた。

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">なるほど / &quot;著者は経験的に、データ交換を伴う大量のデータを分割する際には、ボリューム対サーフィスの比率を最大にするのが良いと考えています。&quot; <a href="https://twitter.com/hashtag/nhbk?src=hash&amp;ref_src=twsrc%5Etfw">#nhbk</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1003573120186212352?ref_src=twsrc%5Etfw">2018年6月4日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- 第 3 章「正当性の検証と性能測定」
  - 並行実行の一般化 (インタリーブや公平性など)、クリティカルセクション、排他制御の段階的な設計、Dekker のアルゴリズム、デッドロック発生の四条件 (相互排除条件、獲得後ウェイト、プリエンプトなし、循環待ち)、Amdahl の法則、Intel HT、SSE、など。
- 第 4 章「マルチスレッドアプリケーション設計の 8 つのルール」
  - どれも重要なルールだけど、その中でも特に印象に残ったのは「並行性はより上位で実装」と「処理能力があればデータは膨張する」と「スレッドの明示的な作成を避け、ライブラリの暗黙的なスレッドを使う」という話でした。
- 第 5 章「スレッドライブラリ」
  - スレッドを暗黙的に持つライブラリ (OpenMP, Intel TBB) 、スレッドを明示的に作るライブラリ (Pthread, Windows スレッド)、特定分野のスレッドライブラリ (Intel MKL, Intel IPP) などの簡単な紹介。
- 第 6 章「並列和とプリフィクススキャン」
  - 配列の並列和とプリフィクススキャンの PRAM モデルによるアルゴリズムと実際の実装、そして実行効率・簡潔性・可搬性・スケーラビリティによる評価。あと、それらを部品として使ったセレクションアルゴリズムの実装の解説、など。
  - セレクションアルゴリズムの実装の話は読んだけど全然頭に入らず。プリフィクススキャンを用いた配列のパッキングについてのコラムが面白かった。プレフィックススキャンしてマーク配列を作り直すことで、パック先インデックスの計算を前処理して並列実行できるようにする。
- 第 7 章「MapReduce」
  - MapReduce といっても分散処理フレームワークの話ではなく、Pthreads を使って MapReduce 処理をしようという話。Map 処理と Reduce 処理への分け方、Reduce 処理の同期のためのバリアオブジェクトの実装、性能評価、タスクのスケジューリングなど。
- 第 8 章「ソート」
  - バブルソート・奇遇転置ソート・シェルソート・クイックソート・基数ソートの並行版実装、良性のデータ競合 (benign data race)、スレッドプール (スレッドチーム) の話など。
- 第 9 章「サーチ」
  - リニアサーチとバイナリサーチの逐次版実装と OpenMP を使った並行化の話。
- 第 10 章「グラフアルゴリズム」
  - グラフ理論の用語解説、深さ優先探索と幅優先探索の並行版実装、モジュロロック、全頂点対最短経路（Floyd 法、Dijkstra 法）の並行版実装、最小スパニングツリー（Kruskal 法、Prim 法）の並行版実装、など。
- 第 11 章「スレッド対応ツール」
  - スレッド対応したデバッガ・プロファイラなどのとても簡潔な紹介。著者が Intel のエンジニアということもあって、Intel な製品率が高め。

# 感想

並行アルゴリズムの解説をするのは大変。著者と訳者の苦労がうかがい知れる一冊でした。読者としても、並行アルゴリズムのインタリーブ解析の説明を読むのはかなりしんどかった。普段コードレビューでやるのもしんどいしね。