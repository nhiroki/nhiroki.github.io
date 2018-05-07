---
layout: post
title: "ウェブブラウザの off-the-main-thread API の話"
date: 2018-05-07 00:00:00 +09:00
tags: web
image: /images/off-the-main-thread-api-blinkon9.jpg
---

ウェブブラウザにおいてメインスレッドはとても重要なリソースです。なるべくメインスレッドを使える状態にしておくことが滑らかな UI/UX を実現する上で重要になります。しかし、実際には多くの処理が実装上の理由やブラウザ仕様の不足によりメインスレッドでしか動かせないため、メインスレッドは忙しくなりがちです。特にページロード時は JavaScript の実行やリソース読み込みなどでとても忙しくなります。

![とあるページの perf プロファイル](/images/off-the-main-thread-api-perf-profile.png)

<p class='caption'>とあるページの perf プロファイル。メインスレッドでせわしなく処理が行われている様子が分かる。</p>

これを解消するために、ブラウザの処理をメインスレッド以外 (off-the-main-thread) でも実行できるようにする試みが行われています。

# 1. Off-the-main-thread とは

メインスレッド以外のスレッドに処理を委譲することを off-the-main-thread と呼んでいます。Off-the-main-thread は multi-thread とも言えますが、ここでは複数スレッドにすることよりも、メインスレッド以外で処理をすることが重要なので off-the-main-thread と呼んでいます。

Off-the-main-thread の実現には大雑把に二つの方法があります。ひとつは JavaScript API レベルで off-the-main-thread 化された機能を提供するという方法です。例えば、Web Worker は JavaScript の実行環境をまるごとメインスレッド以外で動かす機能です。もうひとつはブラウザ内部の処理を一部別スレッドに委譲するという方法です[^unship-off-the-main-thread]。例えば、[V8 はスクリプトのパースやコンパイルをバックグラウンドスレッドで行います](https://v8project.blogspot.jp/2018/03/background-compilation.html)。

前者はウェブアプリケーション開発者がアプリケーションの処理を高速化するためのものであるのに対し、後者はウェブブラウザ開発者がブラウザ内部の処理を高速化するためのものです。本記事では前者の「JavaScript API レベルでの off-the-main-thread 化」について見ていきます。

[^unship-off-the-main-thread]: 内部処理の off-the-main-thread 化は必ずしもうまくいくものではありません。例えば、HTML のパーサーの一部処理は長らく off-the-main-thread で実行されていましたが、スレッド間通信のコストや実装の複雑さに対して得られる性能向上が少ないことから、メインスレッドで実行されるように変更されました。詳しくは「[HTMLParser Redesign](https://docs.google.com/document/d/1PC-Q2zHmC6C3QZw6dhULPS4xhy_pV5ec6i429opLJSo/edit?usp=sharing)」を読んでください。

# 2. Off-the-main-thread API

Off-the-main-thread API の筆頭は Web Worker です。前述の通り、Web Worker は JavaScript の実行環境をまるごとメインスレッド以外 (ワーカースレッド) で動かす機能です。Web Worker は DOM に触れなかったり[^dom-on-worker]、一部のブラウザ API が使えなかったり[^browser-api-on-worker]しますが、基本的には汎用的な処理を行うことができます。

[^dom-on-worker]: DOM に何らかの形で Web Worker 上からアクセスできるようにしようという提案もあります。また、通常は DOM に紐付いた処理を Web Worker 上では DOM に紐付かない形で使えるようにする API (e.g., OffscreenCanvas) もあります。

[^browser-api-on-worker]: ブラウザ API は Web Worker 向けに仕様定義されている必要があります。例えば、最近だと WebUSB の処理を off-the-main-thread 化するために Web Worker で使えるようにしようという提案が出されています ([Intent to Implement: WebUSB on Web Workers - blink-dev](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/MReOVYgRpKk))。

Web Worker 以外には、特定用途に特化した off-the-main-thread API があります。[Audio Worklet](https://developers.google.com/web/updates/2017/12/audio-worklet) はオーディオ処理をワーカースレッド上でスクリプティングする機能です。[Animation Worklet](https://github.com/WICG/animation-worklet) はアニメーションにまつわる処理、例えばページスクロールに応じたアニメーションをスクリプティングする機能です。オーディオやアニメーションはリアルタイム性が求められることから、メインスレッド上で実行するよりも off-the-main-thread で実行した方が好ましいのでこういった API が提供されています。

なぜ Web Worker ではなく Worklet というものを作ったのか。これは求められる実行モデルの違いによるのですが、その辺りの経緯は「[JavaScript のスレッド並列実行環境](/2017/12/10/javascript-parallel-processing#6-worklet)」という記事で詳しく紹介したのでそちらを参照してください。

# 3. Off-the-main-thread API の未来

Off-the-main-thread 処理は Blink 開発者の中でホットなトピックとなりつつあります。先日行われた Blink 開発者が集まるカンファレンス [BlinkOn 9](https://bit.ly/blinkon9-info) では、off-the-main-thread に関するセッションが行われ、様々なアイディアが議論されました。

- [Improving the developer experience for doing work off the main thread](https://bit.ly/blinkon9-omt)

![BlinkOn 9](/images/off-the-main-thread-api-blinkon9.jpg)

## 3.1. 論点の整理

Off-the-main-thread 周りの議論は、様々なユースケースや問題意識によって混沌とした状況になっています。BlinkOn 9 でのディスカッションを踏まえ、私なりに論点を整理してみます。

### 3.1.1. 並列処理の目的と現状

本記事では off-the-main-thread の視点からウェブの並列処理を見てきましたが、高性能な計算環境のための multi-thread という視点からの議論もあります。両者のモチベーションと現時点での手段をざっくりまとめると以下のようになります。

- 処理の委譲による UI ジャンクの削減 (off-the-main-thread)
  - **バックグラウンドタスク**: とりあえずどこでもいいからメインスレッド以外でタスクを実行しておいて欲しい。現状は Web Worker 上にタスク実行環境を自分で作る。
  - **DOM 操作**: バックグラウンドタスクの実行結果を元に、DOM 操作も off-the-main-thread で実行できると便利。現在はメインスレッド上でしか実行できない。
  - **ネットワークやストレージアクセス**: ネットワーク処理やストレージ処理がメインスレッドをブロックすべきではない。現在の実装では、ネットワークやストレージへの物理的なアクセスは専用スレッドで行われており、かつ Web Worker 上でこれらの API を使うことで付帯的な処理も off-the-main-thread で実行可能。
  - **メディア処理やセンサー・アクチュエータ処理**: ウェブ技術が様々なデバイスで使われるようになったことで多様なメディアやセンサー・アクチュエータを処理するニーズがある。こうした処理を off-the-main-thread で実行する API が不足している。
- ハイパフォーマンスな並列計算処理環境の実現 (multi-thread)
  - **計算処理**: 機械学習やビットコインマイニング、マルチメディア処理など、ウェブはアプリケーションプラットフォームに留まらず、コンピューティングプラットフォームとしての側面も強くなっている。JavaScript エンジンの高速化や WebAssembly によるネイティブコードの実行によってシングルスレッドでの性能は向上しているが、並列処理環境としてはどうあるべきか？

### 3.1.2. Web Worker の不便な点や足りない点は何か？

ウェブでスレッドを使った処理をしたい場合、現在の選択肢は Web Worker ほぼ一択です。では Web Worker で前述の処理を行う上で不便な点や足りない点は何でしょうか？

- **通信機構**: 実行コンテキスト間の通信は postMessage によるメッセージパッシング、もしくは SharedArrayBuffer による共有メモリになる。postMessage で送れるデータは Structured Clone によるコピーか、Transferables による所有権の転送 (see: [スレッド間 (実行コンテキスト間) 通信](/2017/12/10/javascript-parallel-processing#3-%E3%82%B9%E3%83%AC%E3%83%83%E3%83%89%E9%96%93-%E5%AE%9F%E8%A1%8C%E3%82%B3%E3%83%B3%E3%83%86%E3%82%AD%E3%82%B9%E3%83%88%E9%96%93-%E9%80%9A%E4%BF%A1))。Transferables はメモリコピーのコストを抑えられるが、ごく一部のデータ型しか対応していない。postMessage で高度なメッセージングを行う場合、命令タイプを String オブジェクトにエンコードしてやりとりする必要があり、煩雑になりがち。RPC のような仕組みがあると便利？ (see: [Comlink](https://github.com/GoogleChromeLabs/comlink), [Tasklets](https://github.com/GoogleChromeLabs/tasklets))
- **ワーカーの管理とタスクディスパッチ**: [ワーカーは起動コストが大きい](/2017/12/10/javascript-parallel-processing#5-worker-%E3%81%AE%E3%82%B3%E3%82%B9%E3%83%88)ため、ライブラリなどでワーカープールを作るのがよくあるパターン。ただしライブラリ間で連携しないため、[Web Worker を作りすぎて OOM になることがある](https://crbug.com/782982)。また、どれだけ Web Worker を作ると OOM が起きるかはページやライブラリ側からは分からない。Web Worker の生成数をブラウザが制御できると良い。例えば、ブラウザ管理のワーカープール (see: [Tasklets](https://github.com/GoogleChromeLabs/tasklets))。ワーカープール内ではグローバルスコープを再利用するのか？再利用できるとグローバルスコープの起動が不要になってタスクの開始が早くなり、さらにインポート済みのライブラリが使い回せて効率が良くなるが、タスクの実行順序によっては期待した動作にならない恐れがある。ワーカープールによる抽象化は、高効率な並列計算処理をしたい場合には余計なオーバヘッドになるかもしれない。
- **モジュラリティ**: Web Worker をライブラリやフレームワーク内でも使いやすくする。Web Worker から Web Worker を作る [Nested Worker の実装](https://crbug.com/829119)や、部品を再利用しやすくするために [ES Modules をサポート](https://crbug.com/680046)する、など。
- **記法**: Web Worker はページとは別のファイルに書くか、Blob としてロードする必要がある。インラインで Web Worker を定義・実行する記法や、実行コンテキスト非依存な関数を定義してそれを共有できると良い？ (see: [Clooney](https://github.com/GoogleChromeLabs/clooney), [JS blocks](https://docs.google.com/document/d/1fcYr_yuK61eb-YUgb_gg_7pxU2j-xQ1seVhbFrsUQPs/edit#))。既にあるパーツ (Transferables, Promises, Streams, ES Modules など) を使って自然な形でスレッド処理を統合できると良さそう？
- **API の不足**: DOM 操作ができない。メディアデータやセンサー・アクチュエータを扱う API が足りていない。パーミッションの管理はどうするべきか？

通信機構やワーカープールについては、以前 [Tasklets について書いた記事](/2017/12/10/javascript-parallel-processing#tasklets-api)でも紹介しました。

### 3.1.3. 新しく API を作るとしたらどういったモデルにすべきか？

個別の問題に対応していくことも重要ですが、ウェブ全体としてどのような一貫したプログラミングモデルを提供するべきかというメタな視点からの議論も重要です。

- Web Worker のように実行コンテキストを丸ごと off-the-main-thread にして、その上で使える API を提供すべきか？それとも、呼び出しはメインスレッドだが内部処理が off-the-main-thread で実行されるような API を提供すべきか？
- スレッド専有？スレッド共有 (ワーカープール)？スレッド専有だとしたら、Web Worker のようにスレッドを抽象化したモデルにするのか？それともスレッドをそのまま扱うようなモデルにするのか？ ([WebKit によるスレッド API の提案](https://webkit.org/blog/7846/concurrent-javascript-it-can-work/))
- スレッド間・タスク間でのデータのやりとりはどうするのか？メモリ共有？メッセージパッシング？データの所有権モデル (コピーやムーブなど) はどうあるべきか？
- 実行するコードのやりとりはどうするのか？

この辺りは既に並列処理をサポートしている言語・処理系、例えば Android のバックグラウンドタスクや C# の並列処理機構などの知見を活かしたいと考えています。

## 3.2. 個人的な考え

off-the-main-thread 処理用の実行環境 (ワーカープール) と multi-thread (並列計算) 処理用の実行環境 (スレッド専有) をそれぞれ用意し、DOM を扱う API やメディア向け API などをそれらの上に適宜実装していくことになるのかなぁ、と思っています。記法に関しては JS blocks に期待。API によってはそもそもメインスレッド上では定義せず、ワーカープールやワーカースレッドで使うことを前提にして UI ジャンクが起こらないようにするかもしれません。

# 4. おわりに

本記事では off-the-main-thread とは何か、現在使える off-the-main-thread API、そして未来の off-the-main-thread / multi-thread API について紹介しました。

ウェブブラウザの開発者はウェブアプリケーションの開発者ではないため、どうしてもリアルなユースケースを想像で考えてしまいがちです。ウェブでの並列処理に関して、意見、アイディア、ユースケースをお持ちの方は是非教えてください。

# 注釈