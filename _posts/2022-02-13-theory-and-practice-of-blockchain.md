---
layout: post
title: "『ブロックチェーン技術概論 ― 理論と実践』を読んだ"
date: 2022-02-13 00:00:00 +09:00
tags: book
image: /images/book-theory-and-practice-of-blockchain.webp
---

『ブロックチェーン技術概論 ― 理論と実践』を読みました。原典とされるホワイトペーパーが出版されてから 13 年以上が経ち、最近では「Web 3.0」の中核技術としても期待されているブロックチェーン。議論のポイントを理解し、トレンドを楽しめるくらいの知識はそろそろ身につけたいと思い、ブロックチェーンの本を読むことにしました。

![表紙](/images/book-theory-and-practice-of-blockchain.webp)

数あるブロックチェーン本の中で本書を選んだ理由ですが、出版が 2021 年 6 月と比較的最近で新しい話題もカバーしていることを期待して選びました。またプロトコルレベルの技術詳細に加えて、ブロックチェーンを取り巻く社会的な問題にまで広く言及されているところに惹かれました。

# 内容

帯で「本格的教科書」と銘打ってるだけあって内容的にかなり重い本でした。序文にも次のように書かれています。

> 本書の目的は、未熟な段階にあるブロックチェーンを本物の実用技術に昇華させる人材の育成です。ブロックチェーン技術を基礎から体系的かつ実践的に学ぶための教科書です。

ブロックチェーン技術自体は分散システムや情報セキュリティ、暗号理論といった情報科学に立脚したものですが、それを応用したサービスを提供していくには金融システムや社会システムなどの学際的な知識が求められます。本書はブロックチェーンの技術詳細はもちろんのこと、それを社会実装していく上で必要な知識までも幅広くカバーしています。ただし各分野を基礎からすべて説明することはできないため、読者にはある程度の前提知識が要求されます。私が馴染みのある情報科学に関して言うと、本書を読み進めるには少なくとも学部卒もしくは修士レベルの分散システムや情報セキュリティの教科書知識がないと辛いです。実際には基礎知識があってもトランザクションアルゴリズムや暗号計算の詳細について読むのはかなり厳しく、私もだいぶ読み飛ばしました・・・

- Chapter 1　ブロックチェーン技術の原点
- Chapter 2　ブロックチェーンの概要
- Chapter 3　スマートコントラクトと分散台帳
- Chapter 4　ブロックチェーンを構成する暗号技術の基礎
- Chapter 5　ビットコインのシステム構成と仕組み
- Chapter 6　ビットコインの仕組みの詳細
- Chapter 7　P2P ネットワーク
- Chapter 8　さまざまなノード実装
- Chapter 9　トークンの表現と利用
- Chapter 10　ブロックチェーンのスケーラビリティ
- Chapter 11　暗号技術とスマートコントラクト
- Chapter 12　ブロックチェーンと匿名化技術
- Chapter 13　ブロックチェーンを利用したシステム構成
- Chapter 14　ブロックチェーン特有のリスク
- Chapter 15　ブロックチェーンのビジネスへの導入
- 付録　数学的基礎

# 雑感

- ブロックチェーンを使ったシステムのアーキテクチャやトランザクションアルゴリズムの大雑把な流れが分かり、またそれらが持つ技術的・社会的な課題や議論のポイントを押さえることができ、当初の目的であった「トレンドを楽しめるくらいの知識はそろそろ身につけたい」は達成できました。
- ブロックチェーンというと暗号資産やスマートコントラクトのイメージが強いが、それ自体はコンセンサスアルゴリズムや暗号技術を組み合わせた汎用的な情報技術であり、様々な情報システムで使える。ただし従来の中央集権的なアーキテクチャの方が優れている場合も多く、使うべきかどうかはよく検討すべき。本書はブロックチェーン技術を手放しで褒めるのではなく、それが有効な場合と使うべきではない場合について技術的な観点から中立に論じているところが良かったです。ブロックチェーンは分散システムや暗号理論の研究者にとっておもしろい研究対象なんだろうなと思いました。
- ブロックチェーンを使ったシステムを社会実装していくためには、情報科学以外にも幅広い知識・研究が求められる。例えば暗号通貨を実現するには貨幣の仕組みや社会におけるトラストのアーキテクチャといった金融・社会システムに関する深い知識が必要になる。個人的にはこのトラストのアーキテクチャの話が一番面白かったです。情報技術だけでは解決できない問題を、ゲーム理論的な状況を作り出すことで社会システムとして解決するというのはなるほどなぁと思いました。
- 仮想通貨やスマートコントラクトの価値や実用性の未来はよく分かりませんが、情報システムとしてのそれらはなかなか面白いと思いました。例えば Chapter 10 はブロックチェーンシステムのスケーラビリティについて論じている章で、伝統的な分散システムのスケーラビリティの問題に加え、ブロックチェーン固有のスケーラビリティの問題（例えばトランザクション手数料の問題）をオフチェーンやサイドチェーンといった独特な手法で解決する話が紹介されています。今までの情報システムのワークロードとはひと味違った問題を解決する必要があり、チャレンジングで楽しそうです。