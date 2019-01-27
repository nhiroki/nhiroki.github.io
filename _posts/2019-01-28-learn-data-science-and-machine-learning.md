---
layout: post
title: "2019 年は統計学と機械学習を頑張る"
date: 2019-01-28 00:00:00 +09:00
tags: diary
image: /images/profile.png
---

30 代は「興味はあるけど今まで勉強できなかったこと」に時間を割いていこうと決めている。[2017 年](/2017/12/13/astro-test-2nd-grade)と [2018 年](/2018/11/27/astronomy-space-test-2018-1st-grade)は[天文宇宙検定](http://www.astro-test.org/)に向けて宇宙関係の勉強に割く時間を意識的に多くした。そのおかげで最新の話題にもだいぶついて行けるようになり、満足度が高かった。2019 年もまた違った分野で同じようなことをやりたいと思い、長年気がかりだった統計学と機械学習を重点的に勉強していこうと決めた。

# 統計学

大学生当時、確率論や統計学はデータの意味を解釈するデータサイエンティストのためのもので、ソフトウェアエンジニアはビッグデータを効率的に読み書きするシステムやフロントエンドを設計実装するのが仕事、みたいな思い込みがあった。私はもっぱら後者の仕事に興味があり、並列分散システムの一般論について授業や研究に限らずいろいろ勉強した。一方、確率論や統計学はソフトウェアエンジニアとしてキャリアを積む上で必要性も興味も見いだせず[^ml-engineer]、真面目に授業を受けていなかった[^math]。

しかしディープラーニングの躍進に端を発した第三次 AI ブームが盛り上がるにつれ、機械学習があらゆる分野で応用されるようになり、確率論や統計学の基礎をしっかり学んでこなかったことに引け目を感じるようになってきた。このモヤモヤを振り払うため、今年は確率論や統計学を基礎から学び直すことにした。

[^ml-engineer]: 機械学習エンジニアという職種は 2012 年当時メジャーではなかったはず。
[^math]: 同様に微積分や線形代数も当時はやる気が出なかった。

独学する場合は検定試験を活用してまずざっくり全体を学んでいくのが自分にとってやりやすい。調べたところ[統計検定](http://www.toukei-kentei.jp/)という試験があることがわかったので、その 4 級から勉強を始めた。既に 4 級と 3 級の教本を読み終えており、今は 3 級の問題集を解いている。最低でも 2 級の内容までは勉強して検定試験を受けようと思っている。

- [『統計検定 3 級対応「データの分析」』を読んだ - nhiroki's weblog](/2019/01/12/book-japan-statistical-society-certificate-3rd-grade-textbook)
- [『統計検定 4 級対応「資料の活用」』読了 - nhiroki's weblog](/2018/12/22/book-japan-statistical-society-certificate-4th-grade-textbook)

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">『統計検定 3 級 • 4 級 公式問題集 (2015-2017 年)』を解き始めた <a href="https://t.co/bb2pBGm4AD">https://t.co/bb2pBGm4AD</a> <a href="https://twitter.com/hashtag/nhbk?src=hash&amp;ref_src=twsrc%5Etfw">#nhbk</a> <a href="https://t.co/2xurE6PNGc">pic.twitter.com/2xurE6PNGc</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1086639916295938049?ref_src=twsrc%5Etfw">2019年1月19日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

それと並行して知識を深化させるためにいくつか大学レベルの教科書を読んでみようと思っている。その一冊目として評価の高かった東京大学出版会の『[統計学入門](http://www.utp.or.jp/book/b300857.html)』を買ってみた。

私の現在の仕事では大規模なデータを分析する機会はない。しかし、性能評価やクラッシュレポートといったちょっとしたデータを可視化し、傾向や対策を検討するぐらいの機会はある。そのような時に自信を持ってデータを読み解けるようになれると嬉しい。

# 機械学習

機械学習の理論やアプリケーションへの応用方法にも興味はあるが、それ以上に機械学習を支えるフレームワークやコンピューティングプラットフォームに強い興味がある。ニューラルネットによるワークロードにはどのような特徴があり、どのようなシステムが求められているのか、それを理解できるだけの理論知識が身に着けられたら嬉しい。前述の通り元々並列分散システムに関心があり、最近はハードウェアにも興味があることから、昨今の「自社システム用に機械学習基盤を作るぞ！」という流れはとても楽しく眺めている。それにあれこれ言えるようになりたい[^misreading]。

[^misreading]: 私のお気に入りのポッドキャスト「[Misreading Chat](https://misreading.chat/)」では機械学習の話題が出ることが多いのですが、その内容をちゃんと理解できるようになりたいという欲求もある。

機械学習の分野は近年の急激な技術革新にも関わらず、教科書や学習環境の整備がしっかり進められていて感心する。オンライン学習コースや[機械学習プロフェッショナルシリーズ](https://www.kspub.co.jp/book/series/S043.html)など、タイトルを眺めているだけで知的好奇心が湧いてくる。いくつか書籍を眺めたところ『[これならわかる深層学習入門](https://www.kspub.co.jp/book/detail/1538283.html)』という本が良さそうだったので、しばらくはこの本を理解することに注力しつつ、各種フレームワークのチュートリアルをこなしてみようと考えている。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">『これならわかる深層学習入門』を読み始めた。紙質が良くて思ってたよりも分厚い <a href="https://t.co/VQ6Jst1IyD">https://t.co/VQ6Jst1IyD</a> <a href="https://twitter.com/hashtag/nhbk?src=hash&amp;ref_src=twsrc%5Etfw">#nhbk</a> <a href="https://t.co/FK8MOtSPe6">pic.twitter.com/FK8MOtSPe6</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1089532299748683776?ref_src=twsrc%5Etfw">2019年1月27日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

まだ一歩を踏み出したばかりで手探り状態だが、2019 年が終わる頃には SNS で流れてくる最新研究の話題をそれなりに理解して楽しめるくらいの知識があると嬉しい。

# まとめ

2019 年は統計学と機械学習の勉強を頑張る。その結果として機械学習を支えるフレームワークやコンピューティングプラットフォームに対する造詣が深められると嬉しい。