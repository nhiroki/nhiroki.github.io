---
layout: post
title:  "2017 年の振り返り"
date: 2017-12-31 00:00:00 +09:00
tags: diary
image: /images/year-end-reflection-google-home.jpg
---

2017 年のいろいろな振り返り。ただの個人の日記です。今年で 31 歳になりました。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">Google Home を囲んで何やら集会が行われている <a href="https://t.co/bn0aF5Vq1m">pic.twitter.com/bn0aF5Vq1m</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/946317413192052737?ref_src=twsrc%5Etfw">2017年12月28日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# プログラミング

プログラミング欲はほぼ Chromium プロジェクトで満たされていたので、プログラミングをしたくなったら Chromium のパッチを書いてた。同僚にその話をしたら「視野が狭くなるからもっと別のことをやった方が良い」と言われ、確かにそうだなという感じなので、来年は何か別のオープンソースプロジェクトに参加できたらいいなぁ、と思ってる。Rust 使いたい。ただ、仕事以外ではなるべくプログラミング以外のことをしたいとも思っていて、その辺のバランスが難しい。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">プログラミングは楽しいけど、かけた時間に対してリターンを得るにはかなりの時間を投資しないと駄目だと思ってて、そうなるとプライベートは別のことやってプログラミング欲はなるべく業務で満たした方が幸せになれる気がしている</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/896362046974971905?ref_src=twsrc%5Etfw">2017年8月12日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Chromium プロジェクトではスレッディングインフラの整備、Web Worker や Service Worker の改良、Worklet の実装、Worker への ES6 Modules 対応などを中心にやった。調べたところ入れたパッチは 240 個くらい、レビューしたパッチは 300 個くらいらしい。Worklet は今年中には出せるかなぁ、と思ってたけど来年になってしまった。2018 年の早い時期には出ると思うのでお楽しみに。

あと Chromium 関係でいくつか記事を書いた。

- [イベント駆動型サービス実行基盤としての Service Worker](/2017/02/13/service-worker-event-driven-background-processing)
- [Chromium のソースコードの歩き方](/2017/12/01/chromium-sourcecode)
- [JavaScript のスレッド並列実行環境](/2017/12/10/javascript-parallel-processing)
- [Chromium ソースコード珍百景](/2017/12/26/chromium-funny-sourcecode)

後ろ 3 本の記事は Qiita の [Chromium Browser アドベントカレンダー](https://qiita.com/advent-calendar/2017/chromium)向けに書いた。このカレンダーは勢いで作ったんですが、予想以上に多くの方に参加・購読していただいて嬉しかった。どうやら購読者数ランキングで同率一位だったようです。これをきっかけに Chromium 開発に少しでも興味を持っていただけたら幸いです。

![Chromium Browser アドベントカレンダー](/images/year-end-reflection-advent-calendar.png)

Chromium 以外だと、4 月頃は GDB のソースコードリーディングをしていたらしい。どうやらスレッドの実行制御をどういう風にやってるのかが気になっていたらしい。当時のメモを残していなかったためにすっかり忘れてしまった。メモはしっかり残そう。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">興味が湧いたので GDB のコードをダウンロードしてビルドを始めてみた。思ってたよりビルドに時間がかかる。そして &quot;&#39;sbrk&#39; is deprecated&quot; と言われて失敗した。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/857980995957514242?ref_src=twsrc%5Etfw">2017年4月28日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

6 月には Cloudflare を使ってこのブログを HTTPS 化した。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">直した。ページは GitHub Pages でホストしてるんだけど、https 対応して、ドメインを <a href="https://t.co/mZQvlfA3BX">https://t.co/mZQvlfA3BX</a> からサブドメインを削った nhiroki.jp を使うようにした。拡張性考えてサブドメイン使ってたけど、特に使うことなさそうなので <a href="https://t.co/Qt3g8CYshE">https://t.co/Qt3g8CYshE</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/878604180683870211?ref_src=twsrc%5Etfw">2017年6月24日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

8 月頃は不揮発性メモリやそれを使ったストレージシステムについてあれこれ調べた。時間がなくて調べたことを整理しきれてないので、いずれまとめ記事を書いて体系化しておきたい。

# 本

今年はデバイスやオペレーティングシステムに関する本を読むことが多かった。とりわけ技術評論社の本は濃い内容のものが多くてよかった。来年もたくさん読みたい。

- ([ブログ記事](/2017/09/04/book-gpu-technology)) GPU を支える技術 ― 超並列ハードウェアの快進撃［技術基礎］
- ([ブログ記事](/2017/07/11/book-android)) Android を支える技術 I - 60 fps を達成するモダンな GUI システム

「Linux プログラミングインタフェース」は去年の 7 月から 9 ヶ月かけて読んだ。おかげで Linux の全体像をつかむことができて自信を持つことができた。昨年末だけど、本書の内容をとっかかりに [Linux のタスクスケジューラーのコア実装を読んで記事を書いた](/2016/12/11/linux-scheduler)り、今年も[スケジューラー関係の論文を読んで発表する機会を作った](/2017/03/21/paper-linux-scheduler)り、色々繋げられたかなと思ってる。

- ([読書メモ](https://twitter.com/nhiroki_/status/749190068376440832)) Linux プログラミングインタフェース

システムソフトウェア周りだと、オペレーティングシステム・ファイルシステム・言語処理系辺りは何となく様子が分かるけれど、データベース周りは全く分からないことが長年気がかりで、意を決してデータベースマネージメントシステムの教科書を買った。まだ一章分しか読めてないので、来年はもう少し頑張る。

- ([ブログ記事](/2017/08/25/book-database-management-systems-chapter8)) Database Management Systems

今年はウェブプラットフォームに関する本も色々出版されて、ウェブを生業とする者として嬉しかった。どの本もオススメ。

- ([ブログ記事](/2017/08/10/book-real-world-http)) Real World HTTP ― 歴史とコードに学ぶインターネットとウェブ技術
- ([読書メモ](https://twitter.com/nhiroki_/status/938933691279015936)) 超速! Web ページ速度改善ガイド ― 使いやすさは「速さ」から始まる
- ([読書メモ](https://twitter.com/nhiroki_/status/869688379167842304)) Web フロントエンドハイパフォーマンスチューニング

同僚にソフトウェア設計の学び方について聞かれ、いくつか本を紹介した。その流れで新しい本もいくつか読んだ。既に経験的に知っていることが多かったけれど、それが明文化されてラベルが付けられたおかげで理解を整理することができた。

- ([ブログ記事](/2017/10/04/book-principles-of-programming)) プリンシプル オブ プログラミング ― 3 年目までに身につけたい 一生役立つ 101 の原理原則
- ([ブログ記事](/2017/12/18/book-game-programming-patterns)) Game Programming Patterns ― ソフトウェア開発の問題解決メニュー

30 代の目標の一つが「今までやりたかったけどできなかったことを勉強しよう！」なんですが、その実践として宇宙に関する勉強を始めた。小手調べに天文宇宙検定の二級を受けた。詳しくは[受験記](/2017/12/13/astro-test-2nd-grade)を見てください。

- ([読書メモ](https://twitter.com/nhiroki_/status/945645386844086273)) 宇宙の大地図帳
- ([読書メモ](https://twitter.com/nhiroki_/status/916095616366891008)) パラレルワールド ― 11 次元の宇宙から超空間へ
- ([読書メモ](https://twitter.com/nhiroki_/status/883742870422798336)) 天文宇宙検定公式問題集 2 級 (2016-2017 年版)
- ([読書メモ](https://twitter.com/nhiroki_/status/883742870422798336)) 天文宇宙検定公式テキスト 2 級 (2017 - 2018 年版)

後述する「イース VIII」をプレイして地球史に興味を持ったので図鑑を買って読んだ。もっと色々読みたい。

- ([読書メモ](https://twitter.com/nhiroki_/status/830073945017585664)) 地球・生命の大進化 ― 46億年の物語 大人のための図鑑

2 月頃は米国史に興味があって、カーン・アカデミーの動画 ([視聴ログ](https://twitter.com/nhiroki_/status/831271680198586368)) を見ながら本を読んだ。読みかけなので来年は読み切りたい。

- ([読書メモ](https://twitter.com/nhiroki_/status/836243094353793024)) アメリカの小学生が学ぶ歴史教科書

子育てに関する本もいくつか読んだ。さらに何冊か積んでるので、早めに読んで我が家に活かせるところは活かしていきたい。「頭がいい子の家のリビングには必ず「辞書」「地図」「図鑑」がある」を参考に地図と図鑑を置いたら息子が勝手に見るようになったので良かった。

- ([読書メモ](https://twitter.com/nhiroki_/status/919538261877391360)) 世界のトップ 1% に育てる親の習慣ベスト 45
- ([読書メモ](https://twitter.com/nhiroki_/status/864853180806332416)) 頭がいい子の家のリビングには必ず「辞書」「地図」「図鑑」がある

その他にランダムで読んだ本。

- ([読書メモ](https://twitter.com/nhiroki_/status/830903270759501826)) こんなにスゴイ！地図作りの現場
- ([読書メモ](https://twitter.com/nhiroki_/status/915502376429031424)) シリコンバレー式 最強の育て方 ― 人材マネジメントの新しい常識 1 on 1 ミーティング

Kindle 版で買ったけど、ソースコードが見づらくて放置されてる本。PDF 版で書い直そうかな・・・

- ([読書メモ](https://twitter.com/nhiroki_/status/902273093782999040)) ガベージコレクション ― 自動的メモリ管理を構成する理論と実装
- ([読書メモ](https://twitter.com/nhiroki_/status/908165006025408512)) Linux のブートプロセスをみる

# ゲーム

年明け頃はずっと「イース VIII -Lacrimosa of DANA-」をやってた。ダーナ最高だった。サントラ買って仕事中ずっと聴いてたので、今年の進捗はイースによってもたらされたと言っても過言ではない。

3 月に PS4 Pro を買った。本当は Switch を狙ってたんだけど買えなかったのでカッとなって買った。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">届いた！ <a href="https://t.co/FZNc0Y3cN9">pic.twitter.com/FZNc0Y3cN9</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/837618487820312576?ref_src=twsrc%5Etfw">2017年3月3日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

「バイオハザード 7」をやろうと思ったけど、体験版があまりに怖かったので「NieR:Automata」にした。荒廃的な雰囲気の世界観、ストーリー、システム、音楽、どれも素晴らしかった。[オリジナルサウンドトラック](http://www.square-enix.co.jp/music/sem/page/nier/automata/)を買って仕事中ずっと聴いてたので、今年の進捗は「NieR:Automata」によってもたらされたと言っても過言ではない。最近出た[アレンジサウンドトラック](http://www.square-enix.co.jp/music/sem/page/nier/automata_arrange/)も買った。コノママジャダメ。

後日 Switch も買った。「スプラトゥーン 2」と「マリオカート」をやってる。「マリオカート」は息子と一緒にプレイできて楽しい。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">酔った勢いで Prime Now いじってたらスプラトゥーン同梱セットが買えてしまった :D</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/888049463884828674?ref_src=twsrc%5Etfw">2017年7月20日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">回収完了。夜やる <a href="https://t.co/ffMQIqUjIX">pic.twitter.com/ffMQIqUjIX</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/888204331903721472?ref_src=twsrc%5Etfw">2017年7月21日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">初めて A+ になった！ <a href="https://t.co/MT3Zi1bVqO">pic.twitter.com/MT3Zi1bVqO</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/922439810286886912?ref_src=twsrc%5Etfw">2017年10月23日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

真・女神転生好きとしては「Deep Strange Journey」の発売が嬉しかった。ちなみにまだクリアしてない。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">ディープストレンジジャーニー届いた！ <a href="https://t.co/HnZ5O3Gkap">pic.twitter.com/HnZ5O3Gkap</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/923541360233156608?ref_src=twsrc%5Etfw">2017年10月26日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

コツコツと beatmania IIDX もプレイし続けている。今年は DP に本腰を入れて頑張った。その甲斐あってか九段を取得して DPA 12 をいくつかクリアすることができた。左手が以前よりも思い通りに動くようになってきたので、この調子で CANNON BALLERS の間に十段、あわよくば中伝目指して頑張りたい。あと SPA のクロペンをイージー付きだけどクリアできたのが嬉しかった。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">sinobuz やり納め。最後に quasar SPA をやったら 1 good でスコアがめっちゃ伸びた。次回作では DP で中伝ぐらいまでいけると良いな <a href="https://twitter.com/hashtag/iidx?src=hash&amp;ref_src=twsrc%5Etfw">#iidx</a> <a href="https://t.co/gLPZWgE9Qj">pic.twitter.com/gLPZWgE9Qj</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/943452737261338626?ref_src=twsrc%5Etfw">2017年12月20日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# アニメ

今年は消費が供給に追いつかず、多くのアニメを積んでしまった。一通り最終回まで観たアニメは次の通り。

- この素晴らしい世界に祝福を！ 2 
- 政宗くんのリベンジ
- ガヴリールドロップアウト
- ACCA13区監察課
- CHAOS;CHILD
- ゼロから始める魔法の書
- 武装少女マキャヴェリズム
- 正解するカド
- プリンセス・プリンシパル
- キノの旅
- 魔法使いの嫁

他には「BLAME!」を映画館で観てきた。

<blockquote class="twitter-tweet" data-cards="hidden" data-lang="ja"><p lang="ja" dir="ltr">映画「BLAME!」観てきた。重力子放射線射出装置の発射音が最高に格好良かったし、駆除系の不気味さも最高だった。構造体の中を歩いてるだけのシーンがあと数時間分くらいあるとさらに最高なんだけど :) <a href="https://t.co/jnNogwIuyv">https://t.co/jnNogwIuyv</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/866160208720023552?ref_src=twsrc%5Etfw">2017年5月21日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

あと、今年の作品じゃないけど「STEINS;GATE」と「楽園追放 -Expelled from Paradise-」を観た。「STEINS;GATE」は後半しか観たことがなかったので「CHAOS;CHILD」のついでに一気観。妻がハマってた。

# 参加したイベント

今年もレンダリングエンジン Blink のカンファレンスに参加した。BlinkOn 8 は初めての日本開催だった。

- 09/20-21 [BlinkOn 8 - Tokyo, Japan](https://docs.google.com/document/d/11Y1MK-jVQl_xlhFS8dds_6FsC70jQ_9aOtcWALBiz5k/preview)
- 01/31-02/02 [BlinkOn 7 - SF Bay Area](https://docs.google.com/document/d/1jlpsfv0kXCveOEX5l75aATgRXbcAvwyse4Tn6jVprWs/edit)

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">ウェブブラウザのレンダリングエンジン Blink (Chromium) のテクニカルカンファレンス BlinkOn 8 が明日明後日の二日間 Google Tokyo オフィスで行われます。海外から多くのビジターが来るので何だか慌ただしい雰囲気。 <a href="https://t.co/U90qNHUe8j">pic.twitter.com/U90qNHUe8j</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/909997286784040963?ref_src=twsrc%5Etfw">2017年9月19日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

ウェブ系の勉強会にも参加した。

- 04/21 [html5j Webプラットフォーム部 第17回勉強会 (アニメーション)](https://html5j-webplat.connpass.com/event/54040/)
- 02/22 [第68回 HTML5とか勉強会「Webパフォーマンス」](https://html5j.connpass.com/event/50524/)

友人が始めたシステム系輪講会に参加させてもらった。普段アカデミアの方と交流する機会が少ないので、研究の話を色々聞けて視野が広がり楽しかった。第 1 回では私も Linux スケジューラーに関する論文を発表させてもらったり、第 2 回で教えてもらった ZMap の論文を読んで記事にまとめたりした。

- 12/14 [第4回 システム系輪講会](https://systems-research.connpass.com/event/72688/)
- 08/02 [第3回 システム系輪講会](https://systems-research.connpass.com/event/62128/)
- 05/30 [第2回 システム系輪講会](https://systems-research.connpass.com/event/55738/)
  - [[論文] ZMap: Fast Internet-Wide Scanning and its Security Applications (2013)](/2017/06/27/paper-zmap)
- 03/21 [第1回 システム系輪講会](https://systems-research.connpass.com/event/52323/)
  - [[論文] The Linux Scheduler: A Decade of Wasted Cores (2016)](/2017/03/21/paper-linux-scheduler)

Rust を勉強しようと思って勉強会に行ったりしたけど、結局全然書いてない。

- 03/23 [Rust プログラマーミートアップ](https://rust.connpass.com/event/49304/)
- 03/01 [RustのLT会！ Rust入門者の集い #2](https://rust.connpass.com/event/48826/) ([参加メモ](https://twitter.com/nhiroki_/status/836882649721483265))

オペレーティングシステムの集まりにも参加した。

- 04/13 [独自OS委員会第一回公聴会](https://fpgastartup.connpass.com/event/53052/?utm_campaign=event_participate_to_follower&utm_medium=twitter&utm_source=notifications) ([参加メモ](https://twitter.com/nhiroki_/status/852464538922369024))

# その他

今年買ったものだと Google Home と 4K ビデオカメラ (Sony FDR-AX55) の満足度が特に高かった。Google Home はハンズフリーで音楽を再生したり、YouTube 動画を再生できて子育てにとても役立っている。ビデオカメラは適当に撮った動画を再生するだけでも楽しい。

<iframe style="width:120px;height:240px;display:block;margin-left:auto;margin-right:auto;" marginwidth="0" marginheight="0" scrolling="no" frameborder="0" src="//rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=FFFFFF&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=nhiroki-22&o=9&p=8&l=as4&m=amazon&f=ifr&ref=as_ss_li_til&asins=B01AN6BB8A&linkId=7cafc88f3e0e3886a0af64432e55dc2f"></iframe>

人生で初めてぎっくり腰になった。あまりの痛さに動けなくなった。しばらくの間サポーターを付けて生活していた。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">昨晩はぎっくり腰の痛みと寒気が強くて寝るの大変で、朝起きたら汗びっしょりかいててびっくりした。風邪引いたときみたいだった。今は歩けるくらいには回復した。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/840019486124072960?ref_src=twsrc%5Etfw">2017年3月10日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

息子の幼稚園入園のために初めて親子面接を受けた。めちゃくちゃ緊張した。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">息子の幼稚園が無事に決まった。ここ数週間ずっと気がかりだったのでホッとした。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/925714546357968896?ref_src=twsrc%5Etfw">2017年11月1日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

悲しい別れもあった。一年ほど入院していた祖母が 3 月に亡くなった。危篤状態という連絡を受けて職場から病院に急行し、幸いにも最期を看取ることができた。天国でも安らかに過ごして欲しい。

# まとめ

楽しいことがあったり悲しいことがあったり色々あったけど、総じて幸せな一年だった。来年は家族が増えるのでさらに気合入れて人生エンジョイしていきたい。