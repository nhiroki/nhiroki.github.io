---
layout: post
title:  "2021 年の振り返り"
date: 2021-12-31 00:00:00 +09:00
tags: text
image: /images/profile.png
---

2021 年の雑多な振り返り兼備忘録です。今年で 35 歳になりました。

- [2020 年の振り返り](/2020/12/31/year-end-reflection)
- [2019 年の振り返り](/2019/12/31/year-end-reflection)
- [2018 年の振り返り](/2018/12/31/year-end-reflection)
- [2017 年の振り返り](/2017/12/31/year-end-reflection)

# 目次

- [主な出来事](#主な出来事)
- [ソフトウェアエンジニアとしての活動](#ソフトウェアエンジニアとしての活動)
- [子育て](#子育て)
- [趣味の活動](#趣味の活動)
- [今年買ったもの](#今年買ったもの)
- [おわりに](#おわりに)

# 主な出来事

仕事は相変わらずソフトウェアエンジニアとして Chromium ウェブブラウザの開発をしていて、完全リモートワークで家から働いていました。今年はなんと自分の上司が三人（上司・上司の上司・上司の上司の上司）もいなくなるという波乱の一年でしたが、その分自分の責任範囲や裁量が広がって多くのことを学べた一年でもありました。

## Q1

- 仕事では昨年の Q3 後半から参加している [Prerender API (Speculation Rules API)](https://web.dev/speculative-prerendering/) の開発チームで、プリレンダリングを行うコア部分の実装を主導していました [[doc](https://docs.google.com/document/d/1_l1LDUALf8PbZIz5y_UYNGZmYCMKNpePDLBHn-gTsTM/edit?usp=sharing)]。Q1 で既存のナビゲーションアーキテクチャ上で一通り動くものを作り、その後に別チームが作っていた新しいナビゲーションアーキテクチャの上に載せ直す作業をしたり、Web Platform Tests（ブラウザ共有のテストスイート）で Prerender API のテストをするためのインフラづくりなどをしていました。
- 趣味の時間では[量子コンピュータ上でのプログラミングを学んだり](/2021/01/22/book-programming-quantum-computers)、電験三種の勉強をしたりしていました。
- ３月は上の子の卒園・入学準備と下の子の入園準備でてんてこ舞いでした。卒園式が終わってようやく楽になりましたが、それと同時に上の子の園服姿はもう見れないんだなぁってちょっぴり物悲しい気持ちになりました。

## Q2

- 長男が小学校に入学し、次男が幼稚園に入園しました。下の子が日中家にいなくなったことで以前よりも集中してリモートワークできるようになり、また妻と二人で日常的にゆっくりランチできるようになりました（子どもが生まれたとき以来ぶり！）
- 仕事では今まで実装してきたものがいよいよフィールドトライアルを行う段階になり、最後のブロッカー潰しをチーム一丸となってやってました。特にプリレンダリング中のページを表示するタイミングでプリレンダリングが継続できなくなった場合のリスタート処理の設計・実装に悪戦苦闘 [[doc1](https://docs.google.com/document/d/1kaZd_MbDWxaRUycEmpw4hq9-QpFpr2znuez2WmYRUM8/edit?usp=sharing), [doc2](https://docs.google.com/document/d/1B2ljj53rMHKCOCyWV3ekpul8CR4BsIxOZdaK4kXV8pM/edit?usp=sharing)]。プロセスをまたいだ並行並列処理難しすぎる。

<blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">今書いてる patch と設計変更により放棄した previous patch を一つと見なすと、これが人生で一番書くのに時間がかかっている patch な気がする。ずっと書いてる。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1412267900136673285?ref_src=twsrc%5Etfw">July 6, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Q3

Q3 はいろいろあって今年一番しんどかったです。

- 子どもたちが夏休みに入り、平穏なリモートワークは終わりを告げました😭
- コロナウイルスのワクチンを接種しました。一回目は大したことなかったけど、二回目は歯痛と副作用が重なってしまってめっちゃしんどかった。副作用による発熱だと分かっていても熱がある状態で歯医者に行けない苦しみ。痛み止めを飲んでしのぎました。

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="ja" dir="ltr">接種してから 50 時間以上経つけどまだ熱が 38 ℃ 超えてる。歯痛の方も歯茎が少し腫れてて実は熱はこっちが原因じゃないかって気もしてきた。しかしこのままだと歯医者に行けない…</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1424236433108209667?ref_src=twsrc%5Etfw">August 8, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- 長期インターン（約三ヶ月）の学生さんをホストし、Chromium の開発（Prerender API の内部情報を表示する機能の開発）に参加してもらいました。国内からの参加とはいえフルリモートでのインターンということもあり、オンボーディングやナレッジの共有で苦労しました。学生さんには困難な環境で頑張ってもらってとても感謝しています。

<blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">今日からホストするインターンの方が来たんだけど、まず最初に「IIDX 皆伝なんですか？」って聞かれて笑った😂</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1414450324568375300?ref_src=twsrc%5Etfw">July 12, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- 上司と上司の上司がたまたま同時に別チームへ移動することになり、新しい上司のもとで私がリードとしてチームを引き継ぐことになりました。今いる会社では「新しいことをやりたくなった上司がある日突然移籍して下の人がステップアップする」というのはよくある話で、それは組織の新陳代謝の観点からとても健全だと思っているんですが、いざ自分の身に降りかかるととても動揺しますね。はじめは不安でしたが、新しい上司やチームメンバーに支えられて何とかチームを再始動できました。そんなこんなでこの時期はチーム構成やタスクの振り分け、新しいロードマップについてひたすら考えていました。今までも TL としてプロジェクトのロードマップを引いたりすることはありましたが、それと比較して規模や関わる人数がかなり大きくなり、自分の裁量もぐっと広がりました。おかげでより高いレベルでの学びがたくさんありました。正直長年つよつよ上司たちの庇護下であぐらをかいていた自覚があったので、今回のような外圧でコンフォートゾーンを飛び出せて良かったと思っています。
- 「プログラミングおよびプログラミング言語ワークショップ（PPL）」が開催したサマースクールにて、JavaScript 処理系 V8 とレンダリングエンジン Blink のアーキテクチャについて話をしてきました [[blog](/2021/09/04/talk-browser-architecture)]。

## Q4

- ロードマップをもとに粛々とプロジェクトの推進とチームの運営を行っていました。Q4 途中までは急激なミーティング数の増加やチームメンバーのフォローでかなり忙しかったですが、後半になるにつれて自分の役割にも慣れ、チームも自走できるようになって一気に楽になりました。新体制になってから目標としていたマイルストーンも無事に達成でき、ホッと一安心。
- 天文宇宙検定一級を受験し、落ちました。例年 Q3, Q4 は試験に向けて勉強をしているんですが、今年は仕事が忙しかったこともあってモチベーションが上がらず全然勉強できなかったのが悔やまれる [[blog](/2021/11/25/astronomy-space-test-2021-1st-grade)]。

<blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">最近あらゆる物事に対するモチベーションの管理に苦慮してるんだけど、もしやこれが噂のミドルエイジクライシスってやつなのでは・・・</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1457331356095057923?ref_src=twsrc%5Etfw">November 7, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# ソフトウェアエンジニアとしての活動

## Chromium (Chrome) の開発

今年私が入れたパッチは 172 個、レビューしたパッチは 385 個でした。チーム運営やロードマップの策定といったメタな役割の比重が増えたことで、昨年よりもコミット数が減ってしまった。

![Chromium CLs](/images/2021-year-end-reflection-chromium-cls.png)

||コミット数|レビュー数|備考|
|2021|172|385|リーダーシップロールの比重の増加|
|2020|187|358|コロナウイルスによるコードフリーズ、プロジェクト変更|
|2019|259|385||
|2018|181|275|育児休暇 (二ヶ月)|
|2017|240|300||

Prerender API (Speculation Rules API) については [Origin Trial](https://groups.google.com/a/chromium.org/g/blink-dev/c/tcbtZoQIlvI/m/K26gVfbUAQAJ) を始めました。2021 年も引き続きトライアルを続けつつ、prerendering の可能性を広げていきたいと思っています。興味がある方はご連絡ください。

<blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">今まで Filesystem API とか Worker API とか、非ソフトウェアエンジニアな家族に見せても「ふーん」って言われがちな機能ばかり作ってたけど、Prerendering は目に見えて違いが分かる（ページ遷移が瞬時に起こる）ので家族に見せたら「すごいね」と言われてちょっと嬉しい😊</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1458792004767612929?ref_src=twsrc%5Etfw">November 11, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

そのほかにも自分が code ownership を持っている Worker / Worklet APIs や Loading 周りのメンテナンスや技術コンサル的なことをやってました。自分がサポートしていたプロジェクトの一例として、 [Chrome 91 から Service Worker 上で ES Modules が使える](https://web.dev/es-modules-in-sw/)ようになりました。

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="ja" dir="ltr">Storage API をやってた頃は「ユーザの環境差異がやばいし状態同期は複雑だし Chromium で最難」って思ってたし、Service Worker をやってた頃は「スレッドやプロセスを縦横無尽にまたぐ、最難」って思ってたし、Navigation をやってる今は「並行に遷移する状態の塊、最難」って思ってる。どこもむずい</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1400327780223516676?ref_src=twsrc%5Etfw">June 3, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 技術記事

ローディング周りで深堀りした内容や、いまやってる Prerender API とその周辺に関する情報を整理したいと思い、いくつか重めの記事を書きました。

- [リソースの読み込みを助けるウェブブラウザ API の世界](/2021/05/06/resource-loading-apis) (05/06)
- [crossorigin 属性の仕様を読み解く](/2021/01/07/crossorigin-attribute) (01/07)

## 対外発表

プログラミングおよびプログラミング言語ワークショップ（PPL）が開催したサマースクール「[JavaScript 処理系と Chrome ブラウザの実装技術](http://ppl.jssst.or.jp/index.php?ss2021)」にて、JavaScript 処理系 V8 とレンダリングエンジン Blink のアーキテクチャについて話をしてきました [[blog](/2021/09/04/talk-browser-architecture)]。

<div class="youtube"><iframe src="https://docs.google.com/presentation/d/e/2PACX-1vTbELnS3VWyK6sxxdTwcMWTNouiWm1wgOXBa_4214YOcz5coRTZW04U54DKk7jE2mIb5A31C4kYAxyN/embed?start=false&loop=false&delayms=3000" frameborder="0" width="960" height="569" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe></div>

本発表では、主にブラウザ JavaScript に関する歴史や最新動向に関する概説と、ウェブ特有の実行モデルをどのようにブラウザアーキテクチャへ落とし込んでいるか紹介しました [[slide](https://docs.google.com/presentation/d/e/2PACX-1vTbELnS3VWyK6sxxdTwcMWTNouiWm1wgOXBa_4214YOcz5coRTZW04U54DKk7jE2mIb5A31C4kYAxyN/pub)]。ウェブブラウザ開発に少しでも興味を持っていただければ幸いです！

## キャリアアドバイス

[昨年の大学でのレクチャー](/2020/12/04/web-and-browser)や PPL での講演といった対外発表をする機会を頂いたおかげで、学生さんからソフトウェアエンジニアとしてのキャリアや学生生活についてのアドバイスを求められる機会が増えました。少しでもお役に立てたなら幸いですし、自分を振り返るよいきっかけにもなりました。

- [「大学の授業は大切ですか？」への回答 (情報系)](/2021/04/17/lectures-at-university) (04/17)

## コミュニティ活動

toyoshim さんが[立ち上げられた](https://twitter.com/toyoshim/status/1470681009901682689)「Chromium プロジェクトへの貢献に興味がある人をサポートするコミュニティ」のお手伝いを始めました。長年「日本から Chromium 開発に参加する人が増えるといいなー」と思い、[Chromium のアドベントカレンダーを作ったり](https://qiita.com/advent-calendar/2017/chromium)、プロジェクト紹介記事を書いたりして、細々と布教活動をしていました。今後はこのコミュニティを通してより直接的にコントリビューターの方々を支援していければと考えています。

## 言語化

普段ぼんやり考えていることをちゃんと言語化して残していきたいと思い、いくつか記事を書きました。今年はソフトウェア開発に関するものばかりでしたが、来年は分野問わず考えていることをもっと気軽に言葉にして残していきたいです。

- [アクティビティログの記録](/2021/01/19/logging-activities) (01/19)
- [Design Docs への思い](/2021/03/31/design-docs) (03/31)
- [チームにいると頼りになるソフトウェアエンジニア](/2021/04/30/reliable-software-engineers) (04/30)
- [設計に悩みすぎる前に手を動かしてみる話](/2021/09/24/coding-before-overthinking) (09/24)

# 子育て

リモートワークあるある、ミーティングへの子どもたちの乱入が多かったです。

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="ja" dir="ltr">今朝の次男は「まだミーティングやってるの？早く終わりなさい」と言いながら乱入し、終わったあとに様子見に行ったら「早くミーティングやりなさい」と怒られた😂</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1417305293759811619?ref_src=twsrc%5Etfw">July 20, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">ミーティング中に「でぶかなりー」って言うのを聞いた長男、私が同僚に悪口を言ってると思ったらしく諌められた（私が言ったのは Dev / Canary でした😂）</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1456145848425795588?ref_src=twsrc%5Etfw">November 4, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

リモートワークであることを活用して仕事の合間に子どもの勉強を見たり習い事の送り迎えをしているんですが、出社しなくちゃいけなくなると生活基盤がまたぶっ壊れそうでどうしたもんかなぁ、と少し悩み中。

<blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">仕事しながら子どもの宿題見たりしてるんだけど、出社になったら生活基盤がまたぶっ壊れそうだな・・・</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1457592252230864897?ref_src=twsrc%5Etfw">November 8, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

コロナ禍で閉塞的な一年でしたが、その中で少しでも子どもの思い出に残るようなことができていればいいな。

<blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">もしまた何十年か後に日本でオリンピックをやることになって、その時またブルーインパルスが飛行するとしたら、子どもたちは「前回は親父と観に行ったなぁ。すごい暑かった」なんて思い出すのかな…</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1418430922219692036?ref_src=twsrc%5Etfw">July 23, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# 趣味の活動

## ゲーム

**beatmania IIDX**：コロナ禍で昨年はほとんどできませんでしたが、今年は INFINITAS の環境を整えて家で遊べるようにしました。詳しくは「[おうち beatmania IIDX 環境を整えた話](/2021/07/20/iidx-at-home)」に書きました。

![IIDX プレイ環境](/images/iidx-at-home-environment.webp)

楽しすぎて無限にやっていたいけどプレイ時間を確保するのが難しい。DP は念願の十段に合格しました。

![2021 IIDX リザルト](/images/2021-year-end-reflection-iidx-result.webp)

**NieR Replicant ver.1.22474487139...**：NieR:Automata はプレイ済みだったけど NieR Replicant は未プレイで、バージョンアップ版の制作が発表されてからずっと楽しみにしてました。端的に言って最高でした。サントラもヘビーローテーションしてます。是非 NieR:Automata もバージョンアップして作って欲しい。

- [『NieR Replicant ver.1.22474487139...』をクリアした](/2021/08/15/game-nier-replicant) (08/15)

**ゼルダ無双 厄災の黙示録**：初めての無双シリーズ。爽快感があって、気分転換に気楽にプレイできて楽しかった。夜寝る前にちまちま進めてクリアしました。ミファー最強。追加コンテンツはまだやれてない。

- [『ゼルダ無双 厄災の黙示録』をクリアした](/2021/05/14/game-hyrule-warriors-age-of-calamity) (05/14)

**あつまれどうぶつの森**：行事イベントやアップデートが入ったときにプレイしてました。家族で野菜を交換したり、はにわを集めたり楽しかった。家族の協力を得てついに化石をコンプリートしました。

**Minecraft**：ずっと長男と一緒にやってましたが、年末には次男も自分の Switch を手に入れて三人でやるようになりました。Caves & Cliffs Update が入ってから世界が一気に広がって楽しみが増えました。来年の The Wild Update も期待してます！

この他にも『ゼルダの伝説 スカイウォードソード HD』と『アクトレイザー・ルネサンス』も買ったけど、ほとんどできてない。ゲームのリメイクは懐かしさからついつい買ってしまうけど、FF7 Remake くらい劇的な変化がないと途中でモチベーションが下がってきてしまいますね。

## 天文宇宙検定

今年も天文宇宙検定の一級を受けたんですが残念ながら不合格でした。去年は参考書などを丁寧にやり込んで受験したんですが、今年は Q3 が特に忙しくてモチベーションが上がらず勉強が全くできませんでした。来年は忙しくなる前から勉強を始めて次こそ合格したいです。

- [天文宇宙検定 1 級受験記 (4th try)](/2021/11/25/astronomy-space-test-2021-1st-grade)

![クリアケース](/images/astronomy-space-test-2021-1st-grade-clear-folder.webp)

## 電験三種

電磁気学の勉強をしたく、どうせなら実用的な検定試験も受けたいなと思い、２〜３月頃は電験三種の勉強をしていました。しかし結局忙しくて勉強が中途半端になってしまいました。ぜひ再開したいけどしばらく難しそう。

![電験三種教科書・問題集](/images/book-3rd-class-electric-chief-engineer.webp)

## 読書

今年読み終えた本。ジャンルにだいぶ偏りがあるので、来年はもっと幅広い分野の本を読めると良いな。

- [Team Geek ― Google のギークたちはいかにしてチームを作るのか』を読んだ](/2021/12/25/book-team-geek)
- [この一冊で全部わかる Web 制作と運用の基本](/2021/11/18/book-basics-of-website-development-and-operation)
- [仮想空間と VR](/2021/11/28/book-virtual-reality)
- [宮沢賢治と学ぶ宇宙と地球の科学１ ― 宇宙と天体](/2021/11/07/book-miyazawa-kenji-space-and-earth-science-1)
- [連星からみた宇宙 ― 超新星からブラックホール、重力波まで](/2021/09/12/book-binary-star)
- [認知バイアス ― 心に潜むふしぎな働き](/2021/09/05/book-cognitive-bias)
- [木星・土星ガイドブック](/2021/06/18/book-jupiter-saturn-guidebook)
- [数学に魅せられて、科学を見失う ― 物理学と「美しさ」の罠](https://twitter.com/nhiroki_/status/1387633230992400386)
- [核融合エネルギーのきほん](/2021/05/30/book-basics-of-fusion-energy)
- [More Effective Agile ― ソフトウェアリーダーになるための 28 の道標](/2021/04/18/book-more-effective-agile)
- [Web ブラウザセキュリティ ― Web アプリケーションの安全性を支える仕組みを整理する](/2021/02/06/book-web-browser-security)
- [動かして学ぶ量子コンピュータプログラミング ― シミュレータとサンプルコートで理解する基本アルゴリズム](/2021/01/22/book-programming-quantum-computers)
- [こんなにおもしろい宅地建物取引士の仕事 [第 2 版]](/2021/01/18/book-work-of-real-estate-notary)
- [イラストでわかる Docker と Kubernetes](/2021/01/04/book-docker-and-kubernetes)

## アニメ

アニメは心の栄養。今年は色々観れて良かった。来年はもっと観たい。

新作で観たアニメ：

- Re:ゼロから始める異世界生活（Season 2 後半）
- 進撃の巨人（The Final Season）
- Dr.STONE（Season 2）
- Vivy -Fluorite Eye's Song-
- 戦闘員、派遣します！
- 86 -エイティシックス-
- 聖女の魔力は万能です
- 鬼滅の刃　無限列車編
- 魔法科高校の優等生
- 転生したらスライムだった件
- 最果てのパラディン
- 無職転生

旧作で観たアニメ：

- Re:ゼロから始める異世界生活 氷結の絆
- この素晴らしい世界に祝福を！ 紅伝説
- メイドインアビス 深き魂の黎明
- ACCA 13 区監察課 Regards
- 幼女戦記　閑話『砂漠のパスタ大作戦』
- BLACK LAGOON
- BLACK LAGOON Roberta's Blood Trail
- Ergo Proxy
- 鬼滅の刃　立志編
- シン・エヴァンゲリオン劇場版
- 進撃の巨人 OAD イルゼの手帳 -ある調査兵団員の手記-

## 漫画

今年読んだ漫画。ベルセルク、完結まで読みたかった・・・

- 王様ランキング
- エリスの聖杯
- 葬送のフリーレン
- トリリオンゲーム
- SOLEIL～ソレイユ～
- 紛争でしたら八田まで
- サマータイムレンダ
- ベルセルク
- 真の仲間じゃないと勇者のパーティーを追い出されたので、辺境でスローライフすることにしました

# 今年買ったもの

## ゲーミング PC

子どもが Minecraft RTX をやりたがっていたので 13 年ぶりくらいに Windows マシンを買いました。子どもはあっという間にパソコンの使い方を覚えてしまい、今では mod やコマンドブロックを駆使してずっとマイクラをやっています。

![ゲーミング PC](/images/2021-year-end-reflection-gaming-pc.webp)

mod やリソースパックの自作をもっとやっていきたい。

<blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">長男がマイクラで mod や resource pack を作りたいというので今日初めて resource pack を作ってみたんだけど、簡単に作れて良かった。自分が描いた絵がマイクラの世界に反映されて子どもも嬉しいみたい。ポーションがカラフルになりました。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1395343449478418439?ref_src=twsrc%5Etfw">May 20, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

長男、攻略本を読んだり YouTube 動画を観てコマンドを実装することはできるようになりましたが、それらで紹介されていないような複雑な仕組みを「インターネットで検索しながら実装する」ということがまだできないので、来年は自分の知りたいことを検索する方法について教えていきたいです（そして[自分の負担](https://twitter.com/nhiroki_/status/1450437308822409216)を減らしたい・・・）

せっかく新しいマシンを買ったので INFINITAS もやってみようと思い立ったのが、上述のおうち beatmania IIDX 環境を整え始めたきっかけでした。

## 複合機

格安で買って８年以上使ったプリンタが寿命を迎えつつあったので新しいものを物色していたんですが、サイズや予算がちょうど良かった EPSON PX-M6711FT を買いました。子どものドリルやプリントのスキャン・コピーに使いたかったので A3 対応の複合機にしました。毎日大活躍していて買って本当に良かったです。

![複合機](/images/2021-year-end-reflection-printer.webp)

## ルーター

詳しくは「[自宅ルーターを ASUS ZenWiFi AX Mini (XD4) に替えた](/2021/10/31/zenwifi)」に書きました。以前のルーターではビデオ会議の接続が不安定でしたが、ルーターを変えて安定しました。

![ZenWiFi パッケージ](/images/zenwifi-package.webp)

## テレビ

我が家のテレビは 10 年くらい前に買った 42 型のものでそれに Chromecast を挿して使っていたんですが、家族から「YouTube や Amazon Prime Video をテレビでもっと手軽に観れるようにしたい」という要望がありました。どうせなら Google TV 内蔵で子どもがリモコン操作できるようにして画面も大きくしようと思って、4K 有機 EL 55 型の BRAVIA XRJ-55A90J を買いました。以前のものより圧倒的に映像が綺麗で迫力があるし、動作も軽快でとても満足しています。

![BRAVIA](/images/2021-year-end-reflection-tv.webp)

# おわりに

2021 年は計画通りにいかないことが多く、心身の不調も多かった一年でしたが、それでも少しずつ前に進めている手応えを感じる年でもありました。2022 年もできることを日々一つずつ着実にこなし、家族みんなが健やかに暮らせるようがんばりたいです。