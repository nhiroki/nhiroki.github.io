---
layout: post
title: "『プリンシプル オブ プログラミング』読了"
date: 2017-10-04 00:00:00 +09:00
tags: book
image: /images/book-principles-of-programming.jpg
---

『[プリンシプル オブ プログラミング ― 3 年目までに身につけたい 一生役立つ 101 の原理原則](http://www.shuwasystem.co.jp/products/7980html/4614.html)』を読み終わったのでその感想とメモです。

![プリンシプル オブ プログラミング](/images/book-principles-of-programming.jpg)

- [読書メモ on Twitter](https://twitter.com/nhiroki_/status/910764988645597186)

# 読んだ動機

「コードは書けるがクラスや関数の設計スキルがまだまだ未熟な新人ソフトウェアエンジニアが、設計について体系的に学べる本はないか」と聞かれて思い浮かんだうちの一冊が本書でした (この時点では未読)。書評サイト等での評価が高いのと目次をさらっと眺めた感じではきっとお薦めできるだろうと思いましたが、人に薦める以上はまずは自分が読まなくてはと思い、読み始めました。

コード設計に関する本は、他にもデザインパターンやリファクタリングの本なども思い浮かんだのですが、デザインパターンは具体的な課題があって、それを解決するために見るカタログのようなものだと思っていて、適用範囲が限定的だと感じています。そして何よりデザインパターンの本はつまらないので、それよりはもっと汎用的に適用できる考え方を学べる本が良いと考えました。一方、リファクタリングの本も今回の質問に対する答えとしてはやや具体的すぎると感じたので見送りました (実はリファクタリングの本はまともに読んだことがないので実際は違うかもしれません。そのうち読みたい)。

- **2017/12/18 追記：デザインパターンの本を読みました / [「Game Programming Patterns」読了](/2017/12/18/book-game-programming-patterns)**

ちなみに、他にはリーダブルコードと Clean Coder が思い浮かんだのですが、リーダブルコードは読んだけど内容を全く覚えておらず、Clean Coder は目次を見た限りだとコード設計よりもソフトウェア開発プロセスに重点を置いているように見えました。リーダブルコードの目次を見返したところ、なかなか良さそうだったのでもう一度読もうかな。なんで覚えてないんだろ？

# 感想

デザインパターンの本が実践的なデザインのカタログなのに対し、本書はデザインやソフトウェア開発をする上での思想のカタログみたいな本でした。

サブタイトルの通り、3 年くらいソフトウェアエンジニアをやっていれば、見聞きしたり、独学でたどり着いたり、自然と経験できそうなことが多く書かれています。これは本書の内容が役に立たないという意味ではなく、エンジニアがゆくゆくは実践しているべき考え方のエッセンスが網羅されていて、それを一気に学ぶことができるということです。また、たとえ経験的に知っていることでもそれが明文化されてラベルが付けられると理解が整理され[^JTP]、ソフトウェアエンジニア間のコミュニケーションを円滑にするための共通言語としても使えるようになります。例えば、コードレビューで設計方針について意見する際に、詳細を言わずとも原則名を言えば伝わるようになり、余計な波風をたてずに済むかもしれません。

一方で、本書は網羅性を高めようとしたためか、名前は違えど重複する思想が何度も登場するのと、時々浮世離れした思想が登場したりして読むのがだんだんしんどくなってくるのが残念でした。そういうのを適当に読み流せる能力が必要かもしれません。

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">第 3 章「思想〜プログラミングのイデオロギー〜」を読み終わった。色々な思想が簡潔にまとまってて最初は面白いんだけど、各思想で同じようなことを言ってるので、読み進めていくとだんだん飽きてくる。あと抽象的な話が多いので、ある程度開発経験がないと読みにくいかも <a href="https://twitter.com/hashtag/nhbk?src=hash&amp;ref_src=twsrc%5Etfw">#nhbk</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/912812696327798784?ref_src=twsrc%5Etfw">2017年9月26日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

[^JTP]: この辺の感想は前半の章を読んでるときに書いたんですが、後にこれが「ジョシュアツリーの法則」と呼ばれていることを知りました (7.6 節)。

# 結局新人にお薦めできるのか？

本書にはコードによる説明は一切なく、話も抽象的なものが多くなっているので、実践経験が少ない人がいきなりこれを読んでもよく分からないと思います。ある程度コードを書いてきたけれど、設計がいまいちうまくいかなかった経験をした人にお薦めしたいです。冒頭に登場した「コードは書けるがクラスや関数の設計スキルがまだまだ未熟な新人ソフトウェアエンジニア」にはおすすめできます。ただし、いくつか浮世離れした話題があるので、その辺りを真に受けないように配慮する必要はあるかもしれません。いきなりアリストテレスとか関係主義の話をされたらビビる (6.6  節)。

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">本書のサブタイトルには「3 年目までに身につけたい」とあるけど、これは 0 日目から読むべき本という意味ではなく、1-2 年くらい経った人が今までの経験の答え合わせ的に読むと納得しやすいのかな、という印象 <a href="https://twitter.com/hashtag/nhbk?src=hash&amp;ref_src=twsrc%5Etfw">#nhbk</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/912815059629252608?ref_src=twsrc%5Etfw">2017年9月26日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

# 学び

- SLAP (Single Level of Abstraction Principle) という考え方は知らなかった。関数内では同一レベルの抽象度の関数呼び出しや処理だけをすることで、コードの要約性や閲覧性を高めること。普段から無意識に実践してるけど、明文化・ラベル付けされると理解が整理される (2.5 節)
- SLAP のように他の関数を呼び出すコードで構成された関数を複合関数 (Composed Method) と呼ぶらしい (2.5 節)
- 「プリミティブ性」を「純粋」というのは腹落ちした / "プリミティブ性とは、モジュールが表現しようとしている抽象が、すべて純粋であるかどうか、ということです。" (3.17 節)
- これは常に意識していたい / "コードは、自分がそこに来た時よりもきれいにしてから、その場を去ります。最初にそのコードを書いたのが誰であるかに関係なく、少しずつでもコードを改善する努力を続けます。" (5.2 節)
- 「アーキテクチャは組織構成に従う」という有名なコンウェイの法則ですが、それを踏まえて「良いアーキテクチャを先に設計して、それに組織構成を従わせる」という考えはなかったので参考になった (7.2 節)

# 注釈