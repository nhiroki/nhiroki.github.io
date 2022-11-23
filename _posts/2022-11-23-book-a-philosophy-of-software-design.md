---
layout: post
title: "『A Philosophy of Software Design』を読んだ"
date: 2022-11-23 00:00:00 +09:00
tags: book
image: /images/book-a-philosophy-of-software-design.webp
---

『A Philosophy of Software Design』を読みました。

![表紙](/images/book-a-philosophy-of-software-design.webp)

# 概要

本書はソフトウェアデザインの複雑さ (complexity) をテーマにした本です。ソフトウェアデザインの複雑さとは何が原因でどのように表れるのか？複雑さを抑えるための考え方やテクニックにはどんなものがあるのか？著者がソフトウェア（Tcl スクリプト言語やストレージシステム）の開発やソフトウェアデザインに関する大学講義を通して得た知見が余すことなく紹介されています。

著者曰く、ソフトウェアの複雑さとはシステムの理解や変更を難しくする要素を指し、これは些細な変更が広範な修正を必要とする状態 (change amplification)、利用者や開発者の認知負荷 (cognitive load) の増加、認識するのが難しく最も厄介な「未知の未知 (unknown unknowns)」といった症状によって顕在化します。システムが複雑化する原因は、主に依存関係 (dependencies) の肥大化や不明瞭なものごと (obscurity) の増加です。これを避けるには、コードをシンプルで自明にする、複雑なものは隠蔽する、そしてソフトウェア開発のサイクルを通してより良いデザインの模索を繰り返し行っていくことが重要であると主張しています。

おそらく熟練のソフトウェアエンジニアなら感覚的に実践しているものも多く、読みながら「そうだよねー」と感じるものも多いと思います。一方で、ソフトウェアデザインの「複雑さ」をこれほど明瞭に言語化できる人はあまりいないのではないでしょうか。本書の優れた点の一つは、デザインを議論する上でソフトウェアエンジニアが共通して持つべき知識や言語を整理しているところだと思いました。これにより、設計のベストプラクティスを普段から意識的（ここ重要）に実践できるようになり、設計上の「怪しい部分」（本書では red flags と呼んでいます）について他のソフトウェアエンジニアと共通言語で明確に指摘しあえるようになります。設計の経験が少ない方が読むにはちょっとしんどいかもしれませんが、シニアがコードレビューなどを通して徐々に伝授していき、どこかのタイミングで本書を手渡して体系的に腹落ちしてもらうといいのかな、などと思いました。

この本、邦訳される予定はないんですかね？

# メモ

考え方やテクニックの詳細は実際に本を読んでもらうとして、ここでは個人的に特になるほどと思ったことをメモしておきます。

- deep な module と shallow な module
  - 「使いやすい（例えば適切な information hiding が行われている）モジュールは deep である」というのは、「モジュールやメソッドは最小の機能毎に細かく分割した方が良い」というおそらく広く一般的に知られている原則とは相反していて面白かったです。
  - 異なるレイヤーでは異なる抽象化をすべき。単なる pass-through を避ける。
  - interface と implementation の違い、特に interface コメントと implementation コメントの違いを意識すべき。
  - 時系列順にメソッドやクラスを切り分けてしまう temporal decomposition に気をつける（やりがち・・・）
- コードの書き方
  - モジュールの作り手が作りやすいコードではなく、使い手が使いやすいコードを書くべき。
  - 特殊なケースではなく、広く一般的なケースで使いやすくするべき。
  - 書きやすいコードではなく、読みやすいコードにすべき。
- 例外処理
  - 第 10 章の Define Erros Out Of Existence が面白かった。
  - 例外を投げるのは簡単だが処理するのは難しい。これが複雑さを高める原因になる。例外を投げるケースを極力少なくすることで複雑にならないようにする。といっても単に例外を握りつぶすわけではなく、自然な形で処理する。
    - 例：Java と Python の substring メソッド、Windows と Unix におけるファイル削除
  - 他にも masking や aggregation、例外ではなくクラッシュによるリカバリといったテクニックを駆使する。
- インクリメンタル（アジャイル）に開発すべきは機能ではなく抽象化。動くものの完成を急ぐ tactical programming ではなく、設計を重視した strategic programming を意識する。これには先を見据えた investment mindset が重要になる。
- 著者はたびたび『Clean Code』本の主張に反論している。『Clean Code』本は未読なのでちょっと読んでみたくなった。

# 目次

1. Introduction
2. The Nature of Complexity
3. Working Code Isn’t Enough
4. Modules Should Be Deep
5. Information Hiding (and Leakage)
6. General-Purpose Modules are Deeper
7. Different Layer, Different Abstraction
8. Pull Complexity Downwards
9. Better Together or Better Apart?
10. Define Erros Out Of Existence
11. Design it Twice
12. Why Write Comments? The Four Excuses
13. Comments Should Describe Things that Aren’t Obvious from the Code
14. Choosing Names
15. Write The Comments First (Use Comments As Part Of The Design Process)
16. Modifying Existing Code
17. Consistency
18. Code Should Be Obvious
19. Software Trends
20. Designing For Performance
21. Decide What Matters
22. Conclusions
