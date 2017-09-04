---
layout: post
title: "「GPU を支える技術 ― 超並列ハードウェアの快進撃 [技術基礎]」読了"
date: 2017-09-04 00:00:00 +09:00
tags: book
---

「[GPU を支える技術 ― 超並列ハードウェアの快進撃 [技術基礎]](http://gihyo.jp/book/2017/978-4-7741-9056-3)」を読み終わった。

![GPU を支える技術](/images/book-gpu-technology.jpg)

# 感想

- GPU 周辺の知識を一通り学べる良書。とても勉強になった。GPU の仕組みやプログラミング、グラフィックスに関する前提知識は一切必要ないと思うけど、CPU に関しては学部生のコンピュータアーキテクチャくらいの知識はないと読むのがしんどいと思う。まえがきにもそう書いてある。

> 本書では、汎用 CPU の基本的な処理方法の概念はおおよそ理解があるという想定で解説を行っています。

- ハードウェア主眼の本だと思って読み始めたけど、3D グラフィックス周りの基礎知識もいっぱい書いてあって、それらも一緒に学べて良かった (**Graphics** Processing Unit の本だから書いてあって当たり前？)
- ハードウェアの詳しい説明の後に CUDA や OpenCL におけるサンプルが提示されているため、システム構成を想像しながらサンプルコードが読めて良かった。いきなりサンプルコードが提示されても挙動がよく分からなかったと思う。
- GPU プログラミングは並列処理の苦しみに加えてヘテロジニアスなシステムによる苦しみがあってとてもつらそうだし、まだまだ発展途上に感じた。ユニファイドメモリなんかはホモジニアスっぽく見せてプログラミングしやすくする仕組みだと思うんだけど、それはそれで最適化 (性能予測) しにくそう。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">最近やっと PC Watch の &quot;後藤弘茂 Weekly 海外ニュース&quot; の記事が読めるくらいの知識がついてきた気がする</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/898351262290255873">2017年8月18日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

# 読書メモ

メモというよりも、読みながら書いたまとめと、追加で調べたことと、感想が混ざりあったもの。間違いがあったら [GitHub Issue](https://github.com/nhiroki/nhiroki.github.io/issues) もしくは [Twitter](https://twitter.com/nhiroki_) で指摘してもらえると助かります。

[Twitter でのメモ書き](https://twitter.com/nhiroki_/status/890958845320744960)

## 第 1 章　[入門] プロセッサと GPU

ディスプレイに画像を表示する仕組み (フレームバッファ、VRAM、ラスタースキャン、ディスプレイインタフェース)、3D グラフィックスの歴史、モデリング技法 (モデルの作成、グローバル・ローカル座標系、トランスポーズ、ライティング)、CPU と GPU の違いとシステム構成、モバイル用 GPU、CPU と GPU のメモリ空間、並列実行方式の違い (SIMD/SIMT) など。第 2 章以降のための基礎知識の解説。

GPU では超並列処理を行うため、性能を最大限発揮するためには　CPU よりも高いメモリバンド幅が求められる。そのため、CPU とは独立したハイエンドな GPU (ディスクリート GPU) には専用の高バンド幅を持つデバイスメモリ (GDDR メモリ) が載っている。両者のアドレス空間は独立しているため、GPU 上で処理を行う場合はメインメモリからデバイスメモリへのデータ転送が必要となる[^main-device-memory-copy]。後続の章で詳しく解説されるが、このメモリ構成の分断が GPU プログラミングを難しくしており、それを改善する工夫が各社で行われている。

一方、性能よりも省電力やサイズが重視されるモバイルデバイスでは CPU と GPU が同一のチップに乗っていて、メモリ空間の共通化が実現されている。また Intel の Core プロセッサシリーズや AMD の APU (Accelerated Processing Unit)[^APU] などでもメモリ空間が共通化されている。この構成では CPU と GPU でメモリバンド幅の取り合いとなるため、それを改善する仕組みが必要になる。例えば Intel Core プロセッサでは大容量の eDRAM L4 キャッシュを載せることで、GPU が求める高いメモリバンド幅を実現している[^eDRAM]。

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">&quot;discrete&quot; の意味は知ってたけど、&quot;integrated&quot; の対義語って認識はなかった <a href="https://twitter.com/hashtag/nhbk?src=hash">#nhbk</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/891804078090604545">2017年7月30日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

[^main-device-memory-copy]: [GPUプログラミングを難しくするCPUとGPUのメモリの分散](http://news.mynavi.jp/column/architecture/359/) / マイナビニュース - コンピュータアーキテクチャの話 (359) (2016/07/29)
[^APU]: [AMD Accelerated Processing Unit](https://ja.wikipedia.org/wiki/AMD_Accelerated_Processing_Unit) / Wikipedia
[^eDRAM]: [Haswellの高性能グラフィックスのカギ「Intel内製eDRAM」の詳細](http://pc.watch.impress.co.jp/docs/column/kaigai/638791.html) / PC Watch - 後藤弘茂のWeekly海外ニュース (2014/03/10)

## 第 2 章　GPU と計算処理の変遷

グラフィックアクセラレータの歴史、スプライト処理、BitBLT 処理、シェーダ、科学技術計算への応用、演算精度、CUDA と OpenCL、並列処理のパラダイム (MIMD, SIMD, SIMT) など。

グラフィックアクセラレータは元々アーケードゲームや CAD ワークステーションなどでの利用を想定して開発されたが、次第に汎用コンピュータのグラフィックス処理にも使われるようになり、1999 年の NVIDIA GeForce 256 の登場に伴って GPU (Graphics Processing Unit) と名付けられた。その性能特性が科学技術計算やニューラルネットワークの計算に適していることが分かると、さらに様々な分野で活用されるようになっていった。

GPU におけるレンダリングパイプラインは大きく分けると頂点シェーダ (トランスポーズ) とピクセルシェーダ (ライティング) に分けられる。初期の GPU ではこのパイプラインが別々に実装されていたため、どちらかの処理負荷が大きくなると他方が待たされてしまうことがあった。そこで両者を共通化したユニファイドシェーダという構造が用いられるようになり、処理負荷に応じて演算器の利用方法を調整することができるようになった。

GPU 上でプログラミングをする場合は専用のライブラリやコンパイラツールチェーンが必要になる。グラフィックス処理では OpenGL (Open Graphics Library) が使われているが、計算処理には NVIDIA の CUDA (Compute Unified Device Architecture) や業界団体によって仕様定義されている OpenCL (Open Computing Language) などが使われる。

- 本章の SIMD と SIMT の説明は分かりやすかった。
- BitBLT (Bit Block Transfer)[^BitBLT] が分かったような分からないようなって感じなんだけど、オフスクリーンバッファを VRAM に転送するのが主目的で、オフスクリーンバッファ操作時に入力画像の論理演算処理もできるって感じなのかな？
- シェーダという名前に反してシェーディング以外のことをしてるの門外漢からするととても分かりにくいと長年思っている。

> "なお、ライティングの方は光を当てたときの反射を計算するので「shade」(影)と言うのは良いのですが、トランスポーズを「shade」と言うのは変な感じがします。" (p.46)

[^BitBLT]: [BitBLT (Bit Block Transfer)](https://ja.wikipedia.org/wiki/Bit_Block_Transfer) / Wikipedia

## 第 3 章　[基礎知識] GPU と計算処理

OpenGL のレンダリングパイプライン、ラスタライズ、Z バッファ、テクスチャマッピング、ライティング、シェーディング、Intel HD Graphics の構造、ゲームグラフィックスの歴史 (Voodoo, DirectX, プログラマブルシェーダ)、法線マッピング、影生成、HDR レンダリング、浮動小数点計算、デバイスメモリ、エラー訂正など。グラフィックス処理を中心とした解説の章。

OpenGL のレンダリングパイプラインは、頂点シェーダ、テッセレーション、ジオメトリシェーダ、頂点ポストプロセス、プリミティブアセンブリ、ラスタライズ、フラグメントシェーダ (ピクセルシェーダ) で構成されている[^OpenGL-pipeline]。各ステージでどのような処理をしているのか分かりやすく解説されている。また、Intel HD Graphics において各ステージがどのように実装されているか紹介されている。

この他、ゲームにおける建物やキャラクターの影生成の話が面白かった。

[^OpenGL-pipeline]: [Rendering Pipeline Overview](https://www.khronos.org/opengl/wiki/Rendering_Pipeline_Overview) / OpenGL Wiki

## 第 4 章　[詳説] GPU の超並列処理

SIMD/SIMT、各社 GPU における並列スレッド数、プレディケート、NVIDIA GPU の構造 (ワープスケジューラ、ダイナミックパラレリズム、メモリシステム、プレディケート)、AMD/ARM GPU、ユニファイドメモリアクセス、細粒度プリエンプション、エラー検出と訂正など。

前章がグラフィックス処理を中心にした解説だったのに対し、本章はハードウェアを中心とした解説の章。読んでいて一番面白い章だった。特にユニファイドメモリやメモリシステムの解説がとてもわかりやすくて良かった。一方、ワープスケジューラの挙動は一度読んだだけじゃよく分からなかった。

### ユニファイドメモリ

CPU と GPU ではメモリ空間が分断されているため、GPU 上で処理を行う場合はメインメモリからデバイスメモリへのデータ転送を明示的に行う必要がある[^main-device-memory-copy]。これが GPU プログラミングを難しくしていた。

NVIDIA Pascal GPU ではユニファイドメモリ (Unified Memory) という機構によってこれを改善している。ユニファイドメモリではメインメモリとデバイスメモリでアドレス空間が共有されており、ページフォールトを駆使して CPU と GPU 間でメモリ転送を行う。例えば、もし GPU がアクセスしようとしたページがメインメモリ側に存在する場合はページフォールトを発生させてデバイスメモリ側にコピーし、メインメモリ上のページを invalidate する。逆の場合も同様にページフォールトを使ってデバイスメモリからメインメモリにページの転送を行う[^unified-memory]。これによってプログラムからはメモリアクセスを透過的に扱うことができ、アドレス空間を意識せずにプログラミングを行うことができる。

ユニファイドメモリでは実際にアクセスしたタイミングでメモリ転送を行うため、余分な転送によるオーバヘッドを避けることができる (これの具体例としては、PG-Strom[^pg-strom] におけるユニファイドメモリ活用に関するブログ記事[^pg-strom-unified-memory]が面白かった)。一方、従来の明示的なデータ転送ではデータをバルクで送れるため、この方がデータ転送と処理を並列化させやすかったりするらしい。この辺りは後続の章で紹介されるストリーム[^cuda-stream]という機能を使うことで改善できる。また CUDA 8 にはプリフェッチやメモリアクセスのヒントを与える API がある[^gputc1] [^gputc2]。

[^unified-memory]: [CPUとGPUで処理を分担する場合のメモリのコピー手法](http://news.mynavi.jp/column/architecture/360/) / マイナビニュース - コンピュータアーキテクチャの話 (360) (2016/08/12)
[^pg-strom]: [PG-Strom](http://strom.kaigai.gr.jp/index.html#page-top)
[^pg-strom-unified-memory]: [Pascal以降のUnified Memoryを使いたおす。](http://kaigai.hatenablog.com/entry/2017/08/15/002500) / KaiGaiの俺メモ (2017-08-15)
[^cuda-stream]: [ストリームを用いたコンカレントカーネルプログラミングと最適化 (PDF)](http://on-demand.gputechconf.com/gtc/2014/jp/sessions/4004.pdf) / GPU Technology Conference (2014/07/16)
[^gputc1]: [CUDA 8.0 新機能のご紹介 (PDF)](https://www.gputechconf.jp/assets/files/1010.pdf) / GPU Technology Conference (2016/10/05)
[^gputc2]: [Pascal世代で進化するUnified Memory (PDF)](https://www.gputechconf.jp/assets/files/1012.pdf) / GPU Technology Conference (2016/10/05)

### 並列実行方式

SIMT のスレッドの管理単位は GPU 毎に異なる。少ないスレッド数でまとめた方がアイドル状態のスレッドが発生しにくく利用効率を高めやすいが、その分命令読み出しの機構などが増えてしまう。大量のスレッドを実行するハイエンド GPU では並列スレッド数を多くし、実行スレッド数が少ないスマートフォンなどでは並列スレッド数が少なめに設計されている。管理単位はメーカーによって次のように異なる:

- Warp[^warp] (NVIDIA) : 32 threads
- Wavefront (AMD) : 64 threads
- Quad (ARM) : 4 threads

SIMT のプレディケート機構による分岐命令の処理方法がよくできていると感じた。SIMT では各スレッドは同一の命令列を実行するが、分岐命令を処理することもできる。どうやっているかというと、自分が処理する必要のない分岐ブロック、例えば if-else 文の else 側ブロックではプレディケートによって命令実行をマスクするようになっている。プレディケートはスレッド毎の分岐処理をしない分ハードウェア実装が簡単になるが、全スレッドが分岐命令の両方を実行した分だけのサイクル数がかかってしまうため、できる限り分岐命令内での処理を少なくすることが重要になる。

その他の実行に関する用語など。

- ダイナミックパラレリズム (Dynamic Parallelism) : カーネルプログラムから直接カーネルプログラムを起動する仕組み。以前はカーネルプログラムはホストプログラムからしか起動できなかった。
- リプレイ (Replay) : 一度のメモリアクセスで 1 ワープの実行に必要なメモリを読み込めない場合、メモリアクセスを繰り返し行う。これをリプレイと呼ぶ。
- スレッドダイバージェンス (Thread Divergence) : プレディケートによってワープ内でのスレッドの実行が揃っていない状態。

NVIDIA GPU の並列実行アーキテクチャの進化は「PascalまでのNVIDIAの4世代のアーキテクチャの進化」という記事[^nvidia-arch]が参考になった。

[^warp]: p.119 の注釈にある "Warp" の説明では "横糸" とあるけど、辞書を見る "縦糸" になってて、"横糸" は "woof" もしくは "weft" っぽい？ [https://eow.alc.co.jp/search?q=warp](https://eow.alc.co.jp/search?q=warp)
[^nvidia-arch]: [PascalまでのNVIDIAの4世代のアーキテクチャの進化](http://pc.watch.impress.co.jp/docs/column/kaigai/755994.html) / PC Watch - 後藤弘茂のWeekly海外ニュース (2016/05/06)

### メモリシステム

CPU と GPU ではメモリ構成が異なる。CPU の場合、

```
CPU Core / L1 / L2 / L3 / クロスバー / DRAM メモリ
```

の順番で並ぶが、GPU の場合、

```
GPU SM / L1 / クロスバー / L2 キャッシュスライス / GDDR メモリ
```

の順番になる。GPU では L2 キャッシュがメモリ寄りにあるのでメモリサイドキャッシュ (Memory-side Cache) と呼ばれる。各 L2 キャッシュは対応する GDDR メモリのデータをキャッシュするようになっている。各デバイスメモリにはユニークなアドレスが付いているので、L2 キャッシュ間でコヒーレンスの問題が起こることはない。プログラムが使用するアドレスが一つの GDDR メモリに偏ると、他のメモリとキャッシュが遊んでしまう。これを防ぐためにアドレスをハッシュして、プログラムからは連続アドレスをアクセスしても、物理アドレスでは各デバイスメモリに分散されるようになっている。

### その他の感想とか

- SIMD/SIMT とかプレディケート実行の話が毎章のように出てくるから、読んでるだけで自然と復習できてだいぶ理解できてきた。分かってる人にはちょっとくどい？
- AMD Polaris GPU には NAND Flash SSD を載せてデバイスメモリを拡張する SSG (Solid State Graphics) という機能があるらしい。将来的には SSG を Persistent Memory に入れ替えるのかな？
- ニューラルネットワーク向けに GPU を使う場合は演算精度が必要にならない（むしろ学習する際にノイズを混ぜたりする）ので、半精度浮動小数点演算のサポートが増えているって話は興味深い。

## 第 5 章　GPU プログラミングの基本

GPU プログラムの互換性 (機械語、アセンブラ、言語)、CUDA によるプログラミング (ストリーム、メモリフェンス、スレッド同期、ユニファイドメモリ)、OpenCL、GPU プログラムの最適化、OpenMP/OpenAAC によるプログラミングなど。

CUDA や OpenCL では CPU と GPU の間のメモリ転送などを明示的に記述する必要があり、プログラミングが難しい (ユニファイドメモリを使うことで緩和される)。一方、OpenMP や OpenACC は C 言語などで書かれた並列化されていない普通のプログラムにディレクティブ指示文を書くことで、コンパイラが並列化をしてくれる。

前章まででかなりのページ数がハードウェアの説明に割かれているので、システム構成を想像しながらサンプルコードが読めて良かった。それにしても GPU プログラミングは並列処理の苦しみに加えてヘテロジニアスなシステムによる苦しみがあってとてもつらそうだし、まだまだ発展途上に感じる。ユニファイドメモリなんかはヘテロジニアスなシステムをホモジニアスっぽく見せてプログラミングしやすくする仕組みだと思うんだけど、それはそれで最適化 (性能予測) しにくそう。理想的には性能を落とさずにハードウェアレベルで完全統合できればいいんだろうけどね。

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">p.210 の注釈 &quot;FORTRAN では逆に列方向が連続アドレスになります&quot;。知らなかった。row-major / column-major っていうのね。column- な言語リストを見ると、軒並み触ったことがないやつだった <a href="https://t.co/ylL1lX3w2Q">https://t.co/ylL1lX3w2Q</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/900138217738878977">2017年8月22日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## 第 6 章　GPU の周辺技術

デバイスメモリの物理的な仕組み、 CPU/GPU と GPU/GPU 間のデータ転送 (PCIe, NVLink, CAPI)、SSD 搭載 GPU など。

GPU 間転送の仕組みと CAPI によるコヒーレンス維持の仕組みが面白い。

> ユニファイドメモリはページ単位でコヒーレンスを維持するものですが、IBM の CAPI はプロセッサコア間と同様に、キャッシュライン単位でコヒーレンスを維持します。 (p.244)

自分はどうやら DMA (Direct Memory Access) 周りをよく理解していないことが分かったので、その辺を整理したい。

## 第 7 章　GPU 活用の最前線

第 7 章「GPU 活用の最前線」を読んだ。ニューラルネットワークの仕組み、画像認識 CNN、ASIC / FPGA、ディープラーニングでの GPU の活用、自動運転・3D モデリング・人の支援での活用、GPU 仮想化、スパコンのアクセラレータとしての利用、など

## 第 8 章　プロセッサと GPU の技術動向

用途毎の CPU/GPU の動向、機械学習を使った分岐予測、CPU/GPU 分離メモリの解消、メモリインターコネクト、Xeon Phi (Knights Landing)、Pezy-SC、カスタムロジックなどによる消費電力の削減、AI サポートなど。

AMD では HSA (Heterogeneous System Architecture)[^HSA1] [^HSA2] という考え方で CPU/GPU 分離メモリの解消に取り組んでいる[^APU]。HSA では CPU だけでなく、GPU や DSP といった各種アクセラレータも MMU を持ち、共通メモリにアクセスしつつキャッシュコヒーレンシの問題をクリアするというアプローチを取っている。

微細化技術によってトランジスタの集積度は大幅に向上しているが、消費電力の観点からチップ状のすべての回路を動かすことができず、一部を電源オフにしておかなければならないダークシリコン (Dark Silicon) 問題がある[^dark-silicon]。この使われていないトランジスタを活用するため、ビデオのエンコード処理などのカスタムロジックを作り、それを必要なときだけ起動する GPU が増えている。

[^HSA1]: [AMD，次期主力APU「Kaveri」で対応する新技術「hUMA」を発表。CPUとGPUが同じメモリ空間を共有可能に](http://www.4gamer.net/games/147/G014731/20130428008/) / 4Gamer.net (2013/04/30)
[^HSA2]: [AMD，GPUとCPUで同じプログラムが動く「HSA」の最新動向を公表。Javaへの対応計画も明らかに](http://www.4gamer.net/games/147/G014731/20130827001/) / 4Gamer.net (2013/08/27)
[^dark-silicon]: [モバイルSoCにおけるダークシリコンの呪縛](http://pc.watch.impress.co.jp/docs/column/kaigai/549137.html) / PC Watch - 後藤弘茂のWeekly海外ニュース (2012/07/26)

---

# 注釈