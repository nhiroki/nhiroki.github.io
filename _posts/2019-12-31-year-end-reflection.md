---
layout: post
title:  "2019 年の振り返り"
date: 2019-12-31 00:00:00 +09:00
tags: diary
image: /images/profile.png
---

2019 年の雑多な振り返り兼備忘録です。今年で 33 歳になりました。

- [2018 年の振り返り](/2018/12/31/year-end-reflection)
- [2017 年の振り返り](/2017/12/31/year-end-reflection)

# ライフイベント

今年は大きな出来事もなく、わりと平穏な一年でした。

- 4 月、出張でカナダのトロントとアメリカのニューヨークに初めて行きました。この辺りのことは旅行記として記事にまとめました ([トロント旅行記](/2019/04/20/trip-to-toronto) / [ニューヨーク旅行記](/2019/05/07/trip-to-new-york))。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">カナダでカナダドライ飲むのが夢だった <a href="https://t.co/O8w42lQcoR">pic.twitter.com/O8w42lQcoR</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1114621581437747205?ref_src=twsrc%5Etfw">April 6, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- 7 月、肩から首にかけて痛みが出て寝返りが打てなくなりました。最初はただの "寝違え" だと思っていたんですが次第に発疹がたくさんでき、病院に行ったところ帯状疱疹だと分かりました。痛みと痒みでこの頃は物事に全く集中できませんでした。
- 9 月、職場が六本木から渋谷に移動しました。通勤時間はだいぶ短くなりましたが、電車は混んでるのと歩行距離が増えたのが残念。渋谷の大規模再開発を眺められるのは良いです。天気が良いと富士山が見えます。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">先日の隙間富士 <a href="https://t.co/YIB8KYhzCT">pic.twitter.com/YIB8KYhzCT</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1187702868830121986?ref_src=twsrc%5Etfw">October 25, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- オフィス移転に伴い早朝ジム通いを始めました。主にトレッドミルでゆっくりランニングをしています。アニメ観ながら走るとちょうど良い。12 月は<s>寒くて</s>体調が良くなくてあまり通えなかったので、来年 1 月からまた頑張る。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">今朝は自転車ではなくランニングマシンを試してみた。元陸上部ながら走るのはあんまり好きじゃないんだけど、ランニングマシンなら自分に合っている気がした。どうやら歩行者や信号などを気にかけながら走るのが嫌だったみたい。ランニングマシンなら無心で走れて良かった。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1176993793008029696?ref_src=twsrc%5Etfw">September 25, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- 11 月、出張でアメリカのサニーベールに行きました。いつもは車で移動するんですが、今回は電車移動してみました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">6 年ぶりくらいに Caltrain に乗ってみた。二階建てで記憶よりもめちゃくちゃデカい。前回乗ったのはアクシデントで、元々はレンタカー移動の予定だったんだけど借り主の先輩が飛行機乗れなくてやむを得ず一人で電車移動したんだよね。同じ飛行機だと思ってたら違う飛行機だったという罠。懐かしい… <a href="https://t.co/3Y0ljtZRfI">pic.twitter.com/3Y0ljtZRfI</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1194368186050723840?ref_src=twsrc%5Etfw">November 12, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

その他、家族と国内の色々なところに旅行に行きました。良い一年だった。

![海](/images/2019-year-end-reflection-ocean.jpg)

# ソフトウェアエンジニア活動

## Chromium (Chrome) の開発

仕事では引き続き Chromium プロジェクトに参加してウェブブラウザを作っていました。今年私が入れたパッチは 259 個、レビューしたパッチは 385 個でした。ちなみに 2018 年は 181/275 (育休で二ヶ月休み)、2017 年は 240/300 でした。自分の得意な領域を広げながら少しずつコントリビューションを増やせたのは良かった。ただ 2020 年からはまだ不慣れな領域を触ることになりそうなので数が減るかもしれない・・・という予防線を張っておく :p

![Chromium CLs](/images/2019-year-end-reflection-chromium-cls.png)

具体的には昨年と同じくウェブのスレッディング API である Web Worker や PWA の要である Service Worker のアーキテクチャ改善を主に担当していました。[Chrome 80 から Web Worker (Dedicated Worker) で ES Modules を使えるようにした](/2019/12/05/es-modules-for-dedicated-workers)ことと、Web Worker の内部実装の [off-the-main-thread 化](https://twitter.com/nhiroki_/status/1155692693013422080)を進めることで Service Worker 最大の技術的負債だった WorkerShadowPage を消せたことが個人的ハイライトです。

<blockquote class="twitter-tweet" data-conversation="none"><p lang="ja" dir="ltr">そしてついに Blink から WorkerShadowPage が完全に消えた。特に Service Workers にとって最大の技術的負債でした。issue が立ってから 4 年かかった。これをマイルストーンにやってきたのでとても嬉しい。kudos to <a href="https://twitter.com/bashik7?ref_src=twsrc%5Etfw">@bashik7</a> hiroshige-san and others who worked on!! <a href="https://t.co/gPqj8fecK3">https://t.co/gPqj8fecK3</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1162179796295536640?ref_src=twsrc%5Etfw">August 16, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

並行して [PlzDedicatedWorker](https://docs.google.com/document/d/1fWsD0oIa5sNDfUFWGJZ41pDo3zzsbFGyQSNdV8nOG4I/edit?usp=sharing) というプロジェクトを進めていたのですが、こちらはあと一歩のところで完遂できなかったので引き続き頑張ります。

あと新たに Blink の core コンポーネントのコードオーナーになりました。core コンポーネントはその名の通りウェブブラウザ実装のコア部分なので、オーナーの仲間入りを果たせてとても嬉しいです。

![Blink Core Owner](/images/2019-year-end-reflection-core-owner.png)

年間を通してマルチスレッドに起因するバグに悩まされました。マルチスレッド難しすぎる。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">nested workers の次は nested message loop のコードレビューをしてるんだけど、両者が併用されたときの挙動を脳内シミュレーションするの無理ゲーすぎる。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1126101612479504386?ref_src=twsrc%5Etfw">May 8, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 技術記事

わずかですが Chromium 関係の記事を書きました。誰かの参考になれば幸いです。来年は自分がやっていることをもう少しまめに記事に残して、あわよくば読んだ人が Chromium プロジェクトに興味を持ってくれたら嬉しいなぁ。

- [Chrome 80 から Web Worker (Dedicated Worker) で ES Modules が使えます](/2019/12/05/es-modules-for-dedicated-workers) (12/05)
- [Chrome 79 から Worklet.addModule() が詳細なエラーを返すようになった話](/2019/10/16/worklet-addmodule-error) (10/16)

今年は explicit memory allocator の論文をいくつか精読して記事にしました。なぜか explicit memory allocator って心惹かれるんですよね。サーベイ記事みたいなものを書きたいと思っていたら 2019 年が終わってしまいました。

- [論文「Reconsidering Custom Memory Allocation」(OOPSLA 2002)](/2019/07/22/paper-reconsidering-custom-memory-allocation) (07/22)
- [論文「snmalloc: A Message Passing Allocator」(ISMM 2019)](/2019/07/08/paper-snmalloc-a-message-passing-allocator) (07/08)
- [論文「MESH: Compacting Memory Management for C/C++ Applications」(PLDI 2019)](/2019/02/26/paper-mesh-compacting-memory-management) (02/26)

他にも深層学習関連のシステム系論文やウェブブラウザのアーキテクチャに関する論文など色々流し読みしました。が、あまり記事に残せなかったのが残念。書籍の読書ログのようにもっと気楽にツイートしながら読むといいのかな？論文は書籍以上に「精読しなきゃ！」という強迫観念がかかってしまって読書コストが高い。

技術記事ではありませんが、[同僚の流れ](https://togetter.com/li/1331865)に便乗して「[新卒のソフトウェアエンジニアになるまで](/2019/04/02/how-i-became-a-software-engineer)」という記事を書きました。自分が学生の頃どんなことをしていたのかずらずら書いた記事です。大学で何をやったらいいか悩んでいる学生へのヒントになると幸いです。

昨年二人目の子どもが生まれて以来、子どもと触れ合う時間を減らさずに妻の負担を増やさずに自分のやりたいことをやる時間を確保する方法についてあれこれ悩んできました。早朝や深夜などにまとまった時間を確保することももちろん重要ですが、自分なりの結論としては「わずかな時間であってもやるべきことを少しずつやる。牛歩であることを受け入れそれでも毎日前に進むことが重要」だと考えています。それに関連して「[日々の進捗の出し方](/2019/02/14/make-progress)」という記事を書きました。

## 趣味プログラミング

今年は統計学や天文学などの勉強を優先したため、趣味時間にプログラミングはほとんどしませんでした。昨年の振り返りで

 > 一応プログラミング的なこととしては、年末に [LeetCode](https://leetcode.com/) を始めました。コーディングインタビューを受ける予定は全くないんですが、この手のスキルは時折磨いておかないとすぐに錆びついてしまうし、気分転換になるのでたまに解いています。
> (from [2018 年の振り返り - nhiroki.jp](/2018/12/31/year-end-reflection))

と書いたんですが、結局今年は LeetCode もほとんどやってないです。

## 参加した技術系イベント

今年は全く発表をしなかったので来年はどこかで何か話したい気もする。

- 11/14-15 [BlinkOn 11](https://www.youtube.com/playlist?list=PL9ioqAuyl6UI6MmaMnRWHl2jHzflPcmA6) (サニーベール)
- 09/18 [W3F Web Developer Meetup #1](https://w3f.connpass.com/event/144725/) (福岡)
- 09/16-20 [W3C TPAC Fukuoka](https://www.w3.org/2019/09/TPAC/) (福岡)
- 04/09-10 [BlinkOn 10](https://docs.google.com/spreadsheets/d/1M9lsx7VXVY3cF7e6PbSdhFR9YYHuTOhb1IEhGsIWN-E/edit#gid=1132537555) (トロント)
- 01/29 [HTTP3Study](https://http2study.connpass.com/event/116857/) (東京)

## その他

[Misreading chat](https://misreading.chat/) が好きでよく聴いていました。技術的な視野が広がってとてもおもしろいです。来年も楽しみにしています！

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">Misreading Chat は話者の専門外の論文であってもちゃんと要点や周辺情報をまとめてきて紹介しているのがすごい。聴者からすると色んな分野の話題をざっくり理解できてとてもありがたい。こういう風に物事を紹介できるようになりたい。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1082979730993762304?ref_src=twsrc%5Etfw">January 9, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

3 月頃はディープラーニング用コンパイラについてサーベイしようとして泥沼にはまって力尽きました。興味ある分野なので来年こそは研究動向が理解できるようになるといいな。

<blockquote class="twitter-tweet" data-conversation="none"><p lang="ja" dir="ltr">ニューラルネットにおける Halide の利用について興味があって気になってたんだけど、結局全然チェックできてないや。グラフコンパイラ周りの論文を眺めてるとよく関連研究に出てくるイメージ。この辺の話を深掘りしてみたいんだけど時間が足りない。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1134111588141547521?ref_src=twsrc%5Etfw">May 30, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">最近、画像処理や深層学習用のコンパイラが気になるから調べてるんだけど、入れ子ループの最適化の話で polyhedral model というのがよく出てくるので調べ始めたら沼に嵌ってきた。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1102573392882892803?ref_src=twsrc%5Etfw">March 4, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

同様にディープラーニング周りのシステム論文を読んでは力尽きるということを繰り返してました。

# 趣味

## 英語

今年も週一でグループディスカッションのレッスンを受けました。集中的に英語アウトプットをする時間を作るためにもっとレッスンを取れると良いのですが、子どもの世話や他の勉強時間の確保などを考えると厳しい。

<blockquote class="twitter-tweet" data-conversation="none"><p lang="ja" dir="ltr">グループディスカッションのコースなんだけど、人によって使う表現の癖みたいなものが違って勉強になるし、逆に自分が使った表現もすぐに他の人が真似してて面白い。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1167077307107831808?ref_src=twsrc%5Etfw">August 29, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

あと語彙力をあげようと iKnow! で単語帳を作って (なるべく) 毎日勉強するようにしました。最近英語の検定試験を受けていないので、来年は何か目標を決めて受験してみるのも良いかもしれないなぁ、と考えてます。

## 天文宇宙検定

昨年落ちた天文宇宙検定 1 級に今年も挑戦しました。残念ながら不合格だったんですが、昨年よりもスコアが上がって準 1 級合格扱いになりました。詳しくは「[天文宇宙検定 1 級受験記 (2nd try)](/2019/12/21/astronomy-space-test-2019-1st-grade)」を参照。宇宙大好きなので引き続き勉強しつつ、来年こそは 1 級に合格できるように頑張ります。

![準 1 級認定証](/images/astronomy-space-test-2019-1st-grade-score.jpg)

## 統計検定

今年の頭に「[2019 年は統計学と機械学習を頑張る](/2019/01/28/learn-data-science-and-machine-learning)」という記事を書いて、統計学と機械学習の勉強をする決意表明をしました。

> 確率論や統計学をしっかり学んでこなかったことに引け目を感じるようになってきた。このモヤモヤを振り払うため、今年は確率論や統計学を基礎から学び直すことにした。独学する場合は検定試験を活用してまずざっくり全体を学んでいくのが自分にとってやりやすい。調べたところ統計検定という試験があることがわかったので、その 4 級から勉強を始めた。

その一環で統計検定 4 級の勉強から始めて、6 月に 2 級を受けて合格しました。詳しくは「[統計検定 2 級受験記](/2019/06/21/japan-statistical-society-certificate-2nd-grade)」という記事にまとめたのでそちらを見てください。来年は準 1 級と 1 級を受けたいと思っています。

![統計検定 2 級 問題 (2019 年 6 月 16 日実施)](/images/japan-statistical-society-certificate-2nd-grade-questions.jpg)

一方、機械学習の方はまったく勉強できませんでした。今の感じだと多分来年も機械学習の勉強には力を入れないかもしれない。もっと根源的な科目 (数学など) の勉強に集中したいと思っています。

## 読書

2019 年に読んだ本は以下の通り。ひたすら統計検定と天文宇宙検定の勉強をしていたのであまり本を読めなかった。リンク先は読書メモです。

- [5G ワールドへようこそ！](/2019/12/16/book-welcome-to-5g-world)
- [ヤバい宇宙図鑑](/2019/08/05/book-yabai-uchu-zukan)
- [トコトンやさしい宇宙ロケットの本](/2019/07/01/book-space-rocket)
- [超・宇宙を解く―現代天文学演習]()
- [意味がわかる統計解析](/2019/06/11/book-understanding-statistical-analysis)
- [統計学入門](/2019/03/20/book-introduction-to-statistics)
- [統計検定 2 級 公式問題集 (2016-2018 年)](/2019/06/18/book-japan-statistical-society-certificate-2nd-grade-questions)
- [統計検定 2 級対応「統計学基礎」](/2019/05/24/book-japan-statistical-society-certificate-2nd-grade-textbook)
- [統計検定 3 級・4 級 公式問題集 (2015-2017 年)](/2019/02/17/book-japan-statistical-society-certificate-3rd-and-4th-grade-questions)
- [統計検定 3 級対応「データの分析」](/2019/01/12/book-japan-statistical-society-certificate-3rd-grade-textbook)

現在進行系で以下の本を読んでます。

- [実践 Rust 入門](https://twitter.com/nhiroki_/status/1126868469503283201)
- [驚異の量子コンピュータ ― 宇宙最強マシンへの挑戦](https://twitter.com/nhiroki_/status/1206445992574275584)

あと日経サイエンスとか月刊ラムダノートとか雑誌系を色々読みました。続々積まれているのでどうにかしたいけど多分どうにもならない・・・。

## beatmania IIDX

今年も beatmania IIDX をたくさんやりました。写真は 26 Rootage の最終ステータス。

<blockquote class="twitter-tweet" data-conversation="none"><p lang="ja" dir="ltr"><a href="https://twitter.com/hashtag/iidx?src=hash&amp;ref_src=twsrc%5Etfw">#iidx</a> Rootage やり納めた。好みなテクノ曲が多くて楽しいバージョンでした。SP は X-DEN 以外は未クリアが増えなかったのでホッと一安心。12 のクリアランプと AAA 埋めが捗って良かった。DP は相変わらず中伝どころか十段も取れず。上達はしているので引き続き頑張ります <a href="https://t.co/uJwbrI0pNL">https://t.co/uJwbrI0pNL</a> <a href="https://t.co/IeeS4s4oBW">pic.twitter.com/IeeS4s4oBW</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1184104499020881920?ref_src=twsrc%5Etfw">October 15, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

SP はレベル 12 の各種クリアランプ埋めや AAA 狙いを中心にやっていました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">「レベル 12 の ExH 埋めなんて無理だしやらないぞ」って思ってた時期もあったけど、最近は「下位中位くらいの曲なら結構できるぞ」ってメンタルモデルが変容してきた。この調子で「これくらいできて当然」のレベルを上げていきたい💪😤 <a href="https://twitter.com/hashtag/iidx?src=hash&amp;ref_src=twsrc%5Etfw">#iidx</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1088088257340919808?ref_src=twsrc%5Etfw">January 23, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

DP 上達のために以前よりも意識的に DP のプレイ頻度を上げていたんですが、気づいたら SP をプレイしていることが多かったです。そのせい (？) で結局今年も DP 十段は取れず。DP をやってると SP もうまくなるし、来年はさらに DP をやるように心がけたい。

<blockquote class="twitter-tweet" data-conversation="none"><p lang="ja" dir="ltr">DP のおかげで左手が強くなって対称固定ができるようになってきたし、1048 式や 3:5 半固定からの切り替えもスムーズになった。3:5 半固定だと右手の負荷が強くて超高密度譜面だとかなりしんどいんだけど、対称固定ができると右手に余力ができてスコアやコンボを狙いやすい。もっと練習したい <a href="https://twitter.com/hashtag/iidx?src=hash&amp;ref_src=twsrc%5Etfw">#iidx</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1108011745279270919?ref_src=twsrc%5Etfw">March 19, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

レベル 12 じゃないけど蠍火 (SPH11) をトリコンできたのがめっちゃ嬉しかったです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr"><a href="https://twitter.com/hashtag/iidx?src=hash&amp;ref_src=twsrc%5Etfw">#iidx</a> 進捗。ついに蠍火 (SPH11) をフルコン！正規譜面でした。曲も譜面も大好きで RED の頃からやり込んできたのでめっちゃ嬉しい。最後の連打抜けた時は思わず叫んだ😄 <a href="https://t.co/ysIMfSXABl">pic.twitter.com/ysIMfSXABl</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1112997139762733056?ref_src=twsrc%5Etfw">April 2, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## PlayStation VR

家庭用 VR の今を体験しようと Amazon の初売りセールで PlayStation VR のセットを買いました。

![PS VR](/images/2019-year-end-reflection-psvr.jpg)

以前新宿の VR ZONE で VR 体験 (機種忘れた) したときは解像度の低さが気になったんですが、PlayStation VR は予想よりも解像度が高めに感じられて良かったです。

最初は同梱されていた PlayStation VR WORLDS をやりました。これには VR 入門的なゲームがいくつか入っています。その中の Ocean Descent (海を潜っていくライド風のゲーム) でなるほどと思ったのが、海の中って現実でも水中ゴーグルをかけるから VR ゴーグルに違和感ないし、水中ゴーグルで視界がぼやける感じなので若干解像度が荒くてもそれが逆にリアルに感じたところでした。

その他にシューティングコントローラーを買って Farpoint という FPS ゲームをやりました。これが最高に面白くてしばらく睡眠時間を削ってプレイしてました。

<blockquote class="twitter-tweet" data-conversation="none"><p lang="ja" dir="ltr">PlayStation VR シューティングコントローラーを買った。想像より大きい。Farpoint をやったんだけど、没入感がすごいね。宇宙に浮かぶ惑星を見たときに感動した。銃の持ち心地とか、照準器を覗き込んで撃ったりとかリアルっぽい。あと、キャラクターの手元の時計が実時間を表してるのが地味に良かった <a href="https://t.co/x05U10i7VK">pic.twitter.com/x05U10i7VK</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1084809380577435650?ref_src=twsrc%5Etfw">January 14, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## マインクラフト

長男 (4 歳) にマインクラフトをやらせてみたら大ハマリし、毎日のように一緒にやっていました。最初は慣れないコントローラー操作やユーザインタフェースに困惑していましたが、あっという間に上達して今では私よりも上手に操作し、私よりも色々なことを知っています。サバイバルモードよりもクリエイティブモードで自由に作るのが好きで、色々な建物や装置を作って見せてくれます。そのたびに創造力や発想力に驚かされています。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">よく一緒に YouTube のプレイ動画を観てるんだけど、そこで仕入れた情報をすぐに自分の世界で試しててすごい。同じ動画を何度も観て復習してるし、学習サイクルが出来上がっている。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1121719560602673153?ref_src=twsrc%5Etfw">April 26, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

マイクラが好きすぎて特に教えていないカタカナも読めるようになりました。マイクラすごい。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">長男にマイクラの攻略本を買ってあげたらすっかり気に入ったらしく、ゲームできない時間もひたすら読み込んでる。本人曰く「勉強してる」らしい。本もすっかりボロボロ。本屋で悩んで選んだかいがあった。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1135885330542764037?ref_src=twsrc%5Etfw">June 4, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

マイクラを通して地学や工学にも興味を持ってくれました。

<blockquote class="twitter-tweet" data-conversation="none"><p lang="ja" dir="ltr">「マグマには色々な物質が溶けている」という話を長男にしたら「どうしてマイクラでは鉄のバケツで溶岩を汲めるの？溶けないの？」と聞かれた。鋭い。地表に噴出した溶岩の温度 (1000 ℃ 前後？) だと鉄の融点 (1538 ℃) には届かないから大丈夫そう。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1199329553862127616?ref_src=twsrc%5Etfw">November 26, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

ただし知識に偏りが生じています :p

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">長男、色々な鉱石を知ってるのに銅を知らなかったりする (銅はマイクラに出てこない)</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1185516151675056128?ref_src=twsrc%5Etfw">October 19, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">長男、YouTube 動画を参考にレッドストーン回路を組んで隠し部屋とか作ってて楽しそう。この流れでプログラミングにも興味を持ってくれると良いな😁</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1199298598350884864?ref_src=twsrc%5Etfw">November 26, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## アニメと映画

アニメが好きなのでジムでのランニング中や夜に観ています。今年の新作で一通り観たアニメは次の通りでした。

- 進撃の巨人 Season 3
- ワンパンマン Season 2
- 盾の勇者の成り上がり
- とある科学の一方通行
- ありふれた職業で世界最強
- コップクラフト
- バビロン
- まちカドまぞく
- Dr.STONE
- この世の果てで恋を唄う少女YU-NO
- 甲鉄城のカバネリ 海門決戦
- ダンジョンに出会いを求めるのは間違っているだろうか II

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">はー、今週の進撃の巨人は特に素晴らしかった。55 話も観続けてると登場人物への感情移入が半端ないわけだけど、特に今回の話は彼らの心揺さぶられる様子がスポンジのように染み込んできて思わず「兵長！！」と叫ばざるを得なかった。子ども起きるから叫ばなかったけど。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1135915101611995137?ref_src=twsrc%5Etfw">June 4, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

旧作ですが以下のアニメも一通り観ました。

- 機動戦士ガンダム 逆襲のシャア (二回目)
- アルドノア・ゼロ (二回目)
- 最弱無敗の神装機竜 (二回目)
- きかんしゃトーマス とびだせ！友情の大冒険
- それいけ!アンパンマン ブルブルの宝探し大冒険!
- ソード・オラトリア ダンジョンに出会いを求めるのは間違っているだろうか外伝

実写映画は次の通り。飛行機に乗ったときに観ています。

- パイレーツオブカリビアン／最後の海賊
- 踊る大捜査線 THE MOVIE 3 ヤツらを解放せよ！
- トゥームレイダー ファースト・ミッション
- アイ・アム・マザー

# 今年買ってよかった物

## PlayStation VR

上述。

## Sony WF-1000XM3

![Sony WF-1000XM3](/images/2019-year-end-reflection-earphones.jpg)

Sony WF-1000XM3 というノイズキャンセリング付き Bluetooth イヤフォンを買いました。初めてのワイヤレス、初めてのノイズキャンセリングで、この価格帯のイヤフォンも初めて (以前は SHURE SE112 で 5000 円くらい) なので比較評価しにくいですが、控えめに言って最高でした。付属イヤーピースでもかなり満足なんですが、私は COMPLY TG-200 に交換して使っています。密着感が強く遮音性が高まり、細かい音までしっかり聴こえます。これをつけて作業してるとかなり集中できます。

## Chromebook

Chromebook を買いました。元々は長男のオンライン学習教材用に買ったんですが、最近は次男が YouTube を観るのによく使っています。親のデバイスを貸すと何をされるか分からない (SNS で勝手にいいねボタンを押されてたりする) ので、子どもが自由に使える専用デバイスがあると安心。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">先日息子のオンライン学習教材用に Chromebook を買ったんだけど、安いし操作も軽快でとても良かった。スタイラスペンでお絵かきも楽しんでる。タブレットも考えたんだけど、物理キーボードに触れる機会を作りたかったのでフリップできる Chromebook にした。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1178237135649853440?ref_src=twsrc%5Etfw">September 29, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 科学のなぜ？新辞典

子どもに新しいことを教えてあげるきっかけづくりに買いました。おすすめです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">最近子どもに新しいことを教えてあげられていない気がしたので、話のきっかけづくりに『科学のなぜ？新辞典』という本を買ってみたんだけど良かった。オールカラーの写真やイラストが多くて分かりやすいし、トピックも読んでみたくなるものが多い。大人も楽しめる <a href="https://t.co/lWQHWZef4f">https://t.co/lWQHWZef4f</a> <a href="https://twitter.com/hashtag/nhbk?src=hash&amp;ref_src=twsrc%5Etfw">#nhbk</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1180990391408943104?ref_src=twsrc%5Etfw">October 6, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# 今年の言葉

<blockquote class="twitter-tweet" data-conversation="none"><p lang="ja" dir="ltr">コンビニで酒とジャッキーカルパスを買ってきて飲み食いしてたら気分良くなってきた。人の言うことに一喜一憂している場合じゃない！俺は俺のやりたいことをやるんや！！＼(^o^)／</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1145685256848072704?ref_src=twsrc%5Etfw">July 1, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# まとめ

総じて今年も幸せな一年でした。来年もみんなで楽しく健康に過ごしたいです。