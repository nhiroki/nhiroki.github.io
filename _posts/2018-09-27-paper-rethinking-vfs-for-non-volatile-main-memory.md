---
layout: post
title: "論文「Caching or Not: Rethinking Virtual File System for Non-Volatile Main Memory」(2018)"
date: 2018-09-27 00:00:00 +09:00
tags: paper
image: /images/profile.png
---

「[Caching or Not: Rethinking Virtual File System for Non-Volatile Main Memory](https://www.usenix.org/conference/hotstorage18/presentation/wang)」という USENIX HotStorage 2018 で発表された論文を読みました。これはそのメモ書きです。

**免責** 精読しておらず内容が間違っている可能性があります。正確な情報が欲しい方は必ず論文を読んでください。誤りの指摘や補足、議論などは [GitHub Issue](https://github.com/nhiroki/nhiroki.github.io/issues) へお願いします。

# 読書状況

ざっくり全体を流し読み。

# 概要

Persistent Memory ファイルシステムにおいて、VFS のメタデータキャッシュがオーバヘッドとなることが分かった。そこで VFS のキャッシングレイヤーをバイパスして直接 Persistent Memory ファイルシステムのメタデータにアクセスできるようにする ByVFS を提案。

- dentry cache (dcache) はバイパスするが、inode cache (icache) はバイパスしない。NVM の write latency が大きいため、頻繁に更新される inode はキャッシュした方が良いという判断。 
- dcache は自身の icache を持つため、バイパスするとその icache が落ちてしまう。代わりにファイルシステム側でその icache を持つ変更を加えた (non-volatile なポインタとして実装)。システムクラッシュなどによって無効なポインタが残ってしまう可能性があるため、そのバリデーションをするためのバージョン機能を導入。

# 評価

- NVM のエミュレータを実装して評価。ファイルシステムは NOVA を使用。ByVFS を使った場合と、VFS をそのまま使ってメタデータキャッシュが cold / warm の場合を測定し、比較。
- ファイル関係のシステムコールでベンチマーク。Cold キャッシュとの比較で性能が向上。Warm キャッシュとの比較でも性能向上、もしくは匹敵するような性能が出た。これはキャッシュのバイパスと icache の Persistent Memory 内キャッシュのおかげ。
- 典型的なコマンドラインツール (e.g., ls) でベンチマーク。Cold キャッシュとの比較で性能向上が確認できたが、いくつかのツールでは若干悪くなった。これは、それらが VFS 内のキャッシュヒットに強く依存しており、実行中に warm キャッシュとなるため。

# 課題

メタデータキャッシュをバイパスしたことで既存 Persistent Memory ファイルシステムの dentry に依存する機能がいくつか壊れたのでワークアラウンドを入れた (フルパスの取得や並行制御など)。これを適切に実装するのは今後の課題。

# 関連研究

Persistent memory based file system:

- BPFS, Aerie, PMFS, SCMFS, SIMFS: ブロックレイヤーやバッファキャッシュのバイパス、インデックス構造の最適化など。VFS のメタデータをバイパスするものではない。
- PMFS, NOVA: 低オーバヘッドで強い一貫性を提供。
- NOVA-Fortis: NOVA に対してスナップショットとフォールトトレランスを提供。
- HinFS: Write Buffer を導入することで NVM の長い write latency を隠蔽。

Optimizing metadata operations:

- 色々あるらしい。

# 疑問・感想

- そもそもなんで VFS のメタデータキャッシュがオーバヘッドになるんだっけ？
  - NVM は DRAM と似た read latency のため、cold cache の場合にキャッシュ有無を見に行く時間が無駄になる。
- Persistent Memory 側が忙しい場合は VFS レイヤーにキャッシュを持った方が良さそうな気がするけどどうなんだろう？