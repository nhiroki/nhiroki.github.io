---
layout: post
title: "「Database Management Systems / Chapter 8 - Overview of Storage and Indexing」読了"
date: 2017-08-25 00:00:00 +09:00
tags: book
---

システムソフトウェア周りだと、オペレーティングシステム・ファイルシステム・言語処理系なんかはそれなりに知識があるけれど、データベース周りは全く分からないことが長年気がかりで、意を決してデータベースシステムの本を読むことにしました。同僚に相談したところ、古典的な教科書だと「[Database Management Systems](http://amzn.to/2w85FJY)」がおすすめとのことで、これの第三版をちまちま読み進めています。

![Database Management Systems](/images/book-database-management-systems.jpg)

[先に本書を読まれた方のブログ](http://d.hatena.ne.jp/nowokay/20120323)を参考にして、私も第 8 章から読むことにしました。それ以前の章はリレーショナルモデル や SQL について扱っていて、そこら辺は今のところ興味が無いのと、日本語の本でも十分カバーできると思って飛ばします。後半の章もデータマイニングとか XML データベースの話はスキップする予定。

いつも通り [Twitter でメモ書き](https://twitter.com/nhiroki_/status/884547606562713600)しながら読みました。

# 読書メモ

第 8 章の目次は次の通りです。

- 8.1 Data on External Storage
- 8.2 File Organizations and Indexing
- 8.3 Index Data Structures
- 8.4 Comparison of File Organizations
- 8.5 Indexes and Performance Tuning
- 8.6 Review Questions

第 8 章では file organization (ファイル編成法) を扱っています。File organization とはディスク上のファイルにどのようにデータベースレコードを格納するか定めたものです。File organization によってレコードの検索や更新のしやすさが変わってくるため、クエリのパターンに応じて適切な方式を選ぶ必要があります。

本章では似たような用語がたくさん出てくるため、適宜用語を整理することが重要だと感じました。私が特に混乱しがちだったのが次のものです。

- **Data entry / Data record**: データエントリはインデックスのデータで、データレコードはデータそのもの。データエントリはインデックスエントリと呼んだ方が分かりやすそう。
- **Page**: 仮想メモリのページングと関係があるのかないのか分からず混乱しました。ここでのページは DBMS 側が決めてるもので、仮想メモリのページングとは別物です (ストレージレイヤーの特性に合わせてページサイズを決めたりするので全く関係ないわけではない)。DBMS はページ単位でデータを管理する。この辺は第 9 章 "Storing Data: Disks and Files" を読むと書いてあるのかな？
- **Sorted file / Heap file**: Sorted file はレコードが何らかのフィールドでソートされた状態で格納されたファイル。一方 heap file はレコードがランダムな順番で格納されたファイル。Heap というとメモリ領域やデータ構造を思い浮かべてしまうので、なぜこれに heap という言葉が割り当てられたのか不思議。Heap には "かたまり" という意味があるので、レコードの乱雑なまとまりのイメージ？Unordered で良い気がするし、実際最初の用語定義では "The simplest file structure is an unordered file, or heap file." とあるので、unordered で問題なさそう (けれど本書内では heap という用語で統一されている)。
- **Clustered index / Unclustered index**: Clustered index はデータエントリとデータレコードが同じ順番に並ぶインデックスのこと。Unclustered index はそのような制約を持たないインデックス。

各項目のまとめを書こうと意気込んで読んでたんですが、章末の review questions に答えるとまとめが出来上がることが分かったので、その回答例を載せておきます。**私が一人で答えたものなので正しい保証はないです。**間違いがあったら [GitHub Issue](https://github.com/nhiroki/nhiroki.github.io/issues) もしくは [Twitter](https://twitter.com/nhiroki_) で指摘してもらえると助かります。さすがに exercises までやる気力はなかったけど、review questions は各項目の重要な点を整理するのにとても役に立ちました。先に review questions に目を通して、それに答えながら本文を読むと理解しやすいかもしれない。

## Review Questions の回答例

> Where does a DBMS store persistent data? How does it bring data into main memory for processing? What DBMS component reads and writes data from main memory, and what is the unit of I/O (Section 8.1)

外部ストレージ（ディスクやテープなど）に保存される。データは必要になってからメモリに読み出される。データの読み書きは Disk Space Manager によって行われる。データユニットは page で、そのサイズは DB のパラメータによるが典型的には 4KB もしくは 8KB。

> What is a "file organization"? What is an "index"? What is the relationship between files and indexes? Can we have several indexes on a single file of records (i.e., act as a file)? (Section 8.2)

- file organization はディスク上のファイルに格納されたデータレコードを並べる方法。file organization 毎に得意な読み書き処理が異なる。
- index はディスク上のデータレコードを管理するためのデータ構造で、特定の読み出し処理を最適化する。
- files と index は別の場合もあれば、一緒の場合もある。一緒の場合 indexed file organization と呼ぶ。別の場合は index が files 内のレコードへのポインタを持ち、一緒の場合は index 内にデータを保持する。
- 一つの file に対して複数の index を持つことができる。ただし、indexed file organization はデータを内包するため、データ重複を許さない限り一つしか持つことができない。

> What is the search key for an index? What is a "data entry" in an index? (Section 8.2)

index 内のフィールドが search key になる。"data entry" は index 内に格納されたデータのこと（例えば data record へのポインタ）。

> What is a "clustered index"? What is a "primary index"? How many clustered indexes can you build on a file? How many unclustered indexes can you build? (Section 8.2.1)

- clustered index は data entry と data record が同じ並びをするようなインデックス。
- primary index は primary key を含むインデックス。
- clustered index は file 上に一つしか作れない。なぜならデータレコードの並び順を複数持つことはできないから。
- unclustered index は並び順に対する制約がないため file 上に複数作れる。

> How is data organized in a hash-based index? When would you use a hash-based index? (Section 8.3.1)

Hash-based index はハッシュ関数を使ったインデックス。一致検索が速いが範囲検索は遅く、フルスキャンよりも遅い。

> How is data organized in a tree-based index? When would you use a tree-based index? (Section 8.4)

Tree-based index は木構造（B+ 木が多い）を使ったインデックス。Hash-based index に比べると一致検索が遅いが、一致検索・範囲検索ともに速い。

> Consider the following operations: scans, equality and range selections, inserts, and deletes, and the following file organizations: heap files, sorted files, clustered files, heap files with an unclustered tree index on the search key, and heap files with an unclustered hash index. Which file organization is best suited for each operation? (Section 8.4)

- (Section 8.4 を読むべし)
- heap files: データレコードがランダムな順番に格納されたファイル
- sorted files: データレコードが特定のフィールドでソートされた状態で格納されたファイル
- clustered files: データレコードとデータエントリが同じ順番で格納されたファイル

> What are the main contributors to the cost of database operations? Discuss a simple cost model that reflects this. (Section 8.4.1)

ディスク I/O のコストが支配的。

> How does the expected workload influence physical database design decisions such as what indexes to build? Why is the choice of indexes a central aspect of physical database design? (Section 8.5)

Section 8.5.1 に書いてある。

ワークロードで特に重要なのが一致検索と範囲検索で、どちらワークロードが多いかによってインデックスを選ぶ。Hash-based index は一致検索は速いが、範囲検索は遅い。一方、Tree-based index は Hash-based index に比べると一致検索が遅いが、一致検索・範囲検索ともに速く行える。そのほかのワークロード (挿入、削除、更新) はどちらのインデックスも効率的に行える。

Tree-based index はデータをソートして維持しつつ、sorted file よりも効率的に挿入や削除を行ったり、検索を行うことができる (sorted file だと二分探索ができるが、二分探索よりも B+ tree のサーチの方が速い)。一方、sorted file はデータがディスク上でシーケンシャルに並ぶため、シーケンシャルリードが速いというメリットがある。B+ tree の亜種である [ISAM (Indexed Sequential Access Method)](https://en.wikipedia.org/wiki/ISAM) は、sorted file のシーケンシャルなページアロケーションのメリットと Tree-based index の高速な検索の両方を併せ持つ。

> What issues are considered in using clustered indexes? What is an index only evaluation method? What is its primary advantage? (Section 8.5.2)

前述の通り、clustered index はデータ重複を避けるために一つしか持てない。また、clustered index は sorted file に比べれば維持コストが低いが、それでも依然として維持コストが高い。例えば、新しいレコードを挿入する場合、新しいページを割り当てたり、既存のレコードを新しいページに移動させたりする必要がある。また、移動させたレコードを参照しているデータをすべて更新しなくてはいけない。そのため、clustering は必要に応じて控えめに使うべきである。特にハッシュを使う場合は範囲検索をしないため clustering すべきではない。

上述のような制限を避けるために、インデックスの search key 自体にクエリへの応答に必要な情報を含めて、それを使って検索する方法がある。これを index-only evaluation と呼ぶ。これによりデータレコードを見ずに、index のデータエントリを見るだけで応答を返すことができる。Unclustered index でこれを行えば、clustered index の制約を受けずにクエリの高速化を行うことができる。

> What is a composite search key? What are the pros and cons of composite search keys? (Section 8.5.3)

Index の search key は複数のフィールドを持つことができる。これらキーを composite search keys もしくは concatenated keys と呼ぶ。

Composite keys は index が持つ情報量を増やすため、より多くの検索方法に対応でき、index-only evaluation を適用できる機会も増える。一方、composite keys に含まれるフィールドが更新されるたびに index を更新する必要が生じ、さらに index のデータサイズも大きくなってしまう。

> What SQL commands support index creation? (Section 8.5.4)

```CREATE INDEX``` 文を使う。