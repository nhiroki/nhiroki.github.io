---
layout: post
title:  "2018 年の振り返り"
date: 2018-12-31 00:00:00 +09:00
tags: diary
image: /images/paternity-leave-fingers.jpg
---

2018 年の雑多な振り返りです。今年で 32 歳になりました。

- [2017 年の振り返り](/2017/12/31/year-end-reflection)

# ライフイベント

## 次男誕生

年明け早々に次男が生まれ、二ヶ月間育児休暇を取りました。育児休暇中の様子については「[育児休暇で二ヶ月会社を休んだ話](/2018/03/12/paternity-leave)」という記事を書きました。

![子どもたちの手](/images/paternity-leave-fingers.jpg)

生まれてすぐの頃は「いかに子ども二人の面倒を見るか」ってことばかり考えていました。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">子ども二人いると両方からアラートが上がった時のトリアージプロセスが重要になるな。次男はまだ新生児なので、大した問題じゃなければ長男を優先。一人で二つのアラートを処理するのは大変なので、平常業務（家事）を放り出してでも夫婦二人でそれぞれアラートに対応できる体制にすることを話し合った</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/959069019893415941?ref_src=twsrc%5Etfw">2018年2月1日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 長男の幼稚園入園

4 月から長男が幼稚園に通い始めました。通い始めた頃は初めての団体生活・子ども社会に不安そうな様子でしたが、親しい友だちもできて今は毎日とても楽しそうです。運動会や発表会では息子の成長に感動しました。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">長男の幼稚園の入園式が終わって一息ついた。ついに我が子も幼稚園かぁ・・・しみじみ</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/982538623374393344?ref_src=twsrc%5Etfw">2018年4月7日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

また弟の誕生によって兄としての意識も芽生えてきたようです。頼もしい。

# ソフトウェアエンジニア活動

## Chromium コミッター

Chromium プロジェクトでは昨年から引き続き Web Worker や Worklet 周りの実装を担当していました。調べたところ、今年私が入れたパッチは 181 個、レビューしたパッチは 275 個でした。育児休暇や大きめのパッチを書くことが多かったこともあり、昨年の 240/300 から減ってしまった。

今年は主に以下のことに取り組みました。

- Worklet の実装と Paint Worklet / Audio Worklet のリリース (Chrome 65 / 66) ([slides](https://docs.google.com/presentation/d/1GZJ3VnLIO_Pw0jr9nRw6_-trg68ol-AkliMxJ6jo6Bo/edit?usp=sharing))
- Web Worker の ES Modules 対応の設計と実装 ([design doc](https://docs.google.com/document/d/1IMGWAK7Wq37mLehwkbysNRBBnhQBo3z2MbYyMkViEnY/edit?usp=sharing), [issue](https://bugs.chromium.org/p/chromium/issues/detail?id=680046))
- WebSocket on Web Worker の off-the-main-thread 化 (Chrome 68) ([issue](https://bugs.chromium.org/p/chromium/issues/detail?id=825740))
- Worker スクリプトのローディングパスの再設計とその実装
  - Off-the-main-thread worker script loading ([issue](https://bugs.chromium.org/p/chromium/issues/detail?id=835717))
  - Browser-initiated dedicated worker script loading (PlzDedicatedWorker) ([design doc](https://docs.google.com/document/d/1fWsD0oIa5sNDfUFWGJZ41pDo3zzsbFGyQSNdV8nOG4I/edit?usp=sharing), [issue](https://bugs.chromium.org/p/chromium/issues/detail?id=906991))
- Web Worker 実装の仕様互換性の向上 (Web Platform Tests のパス率の向上)

Web Worker の ES Modules は基本実装は完了済みなんですが、いくつか Web Worker 自体の互換性の問題からリリースが遅れています。来年の早い時期には出せるように引き続き頑張ります。

昨年まで Web Worker や Worklet はほぼ一人で面倒を見ていたのですが、今年に入って開発に参加してくれる人が増え、コードオーナーも私一人から四人に増えました。おかげで色々なことに取り組めました。その一つとして off-the-main-thread のための新しい API の設計実装[^chrome-dev-summit]をしているのですが、今までの経験からアドバイスをするコンサル的なことをやったりしていました。

[^chrome-dev-summit]: 詳しくは Chrome Dev Summit 2018 のセッション "A Quest to Guarantee Responsiveness: Scheduling On and Off the Main Thread" を参照 ([動画](https://www.youtube.com/watch?v=mDdgfyRB5kg))

Web Worker といえば、育児休暇で休んでる間に Spectre や Meltdown といった脆弱性が公表されて SharedArrayBuffer が無効化されたのはかなりの衝撃でした。[Out-of-process iframe (OOPIF)](https://www.chromium.org/developers/design-documents/oop-iframes) といったセキュリティ機構によって問題が回避されたことで再び有効化されましたが、セキュリティに関する意識が一層高まった一件でした。

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">WHATWG の HTML Standard に投げた PR がマージされて、貢献者リストに名前を載せてもらった！何年もウェブブラウザ開発してて今更感あるけどすごく嬉しい :D <a href="https://t.co/6eJ72XcMUg">https://t.co/6eJ72XcMUg</a> <a href="https://twitter.com/hashtag/nhspec?src=hash&amp;ref_src=twsrc%5Etfw">#nhspec</a> <a href="https://t.co/JnnwTouPe5">pic.twitter.com/JnnwTouPe5</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/999107453282209793?ref_src=twsrc%5Etfw">2018年5月23日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 技術記事

今年も Chromium 関係でいくつか記事を書きました。ウェブ開発者向けの内容に限らず、ブラウザの内部実装に関する話題も書くように心がけました。少しでもブラウザ開発に興味を持ってもらえたら幸いです。

- [Chrome 69 で Web Worker から Web Worker を作れるようになった話](/2018/10/29/nested-workers) (10/29)
- [JavaScript のスクリプトインポートを正しく使い分けようという話](/2018/09/07/javascript-import) (09/07)
- [Service Worker 上での未インストールスクリプトに対する importScripts()
](/2018/08/15/service-worker-import-scripts-after-installation) (08/15)
- [ネットワーク API のメインスレッド依存をなくす話](/2018/06/15/blink-off-the-main-thread-loading) (06/15)
- [ウェブブラウザの off-the-main-thread API の話](/2018/05/07/off-the-main-thread-api) (05/07)
- [Service Worker スクリプトのインストールと更新処理](/2018/02/15/service-worker-install-and-update-scripts) (02/15)

あとは読んだ論文の読書メモを書きました。読みっぱなしにしている論文が多いので、そういうのもちゃんと公開メモとして残せるように来年は頑張りたい。

- [論文「Breaking Apart the VFS for Managing File Systems」(2018)](/2018/10/05/paper-breaking-apart-the-vfs-for-managing-file-systems) (10/05)
- [論文「Caching or Not: Rethinking Virtual File System for Non-Volatile Main Memory」(2018)](/2018/09/27/paper-rethinking-vfs-for-non-volatile-main-memory) (09/27)

## インタビュー

技術評論社の WEB+DB PRESS Vol. 107 にてインタビュー記事を掲載していただきました。

![article header](/images/webdb-press-107-interview-article-header.jpg)

主に私のこれまでのキャリアや技術的に興味があること、Chromium コミッターとしての活動や Chromium プロジェクトの最近のトレンドなどについて個人的な立場から話をさせていただきました。詳しくは「[「WEB+DB PRESS Vol. 107」にインタビュー記事が掲載されました](/2018/10/24/webdb-press-107-interview)」を見てください。技術評論社のページで[記事全文](https://gihyo.jp/dev/serial/01/at-the-front/0001)が公開されています。

## 趣味プログラミング

昨年次のように書いたんですが、結局趣味の時間はほぼプログラミング以外のことをしていました。

> プログラミング欲はほぼ Chromium プロジェクトで満たされていたので、プログラミングをしたくなったら Chromium のパッチを書いてた。同僚にその話をしたら「視野が狭くなるからもっと別のことをやった方が良い」と言われ、確かにそうだなという感じなので、来年は何か別のオープンソースプロジェクトに参加できたらいいなぁ、と思ってる。Rust 使いたい。ただ、仕事以外ではなるべくプログラミング以外のことをしたいとも思っていて、その辺のバランスが難しい。
> ([2017 年の振り返り - nhiroki.jp](/2017/12/31/year-end-reflection))

一応プログラミング的なこととしては、年末に [LeetCode](https://leetcode.com/) を始めました。コーディングインタビューを受ける予定は全くないんですが、この手のスキルは時折磨いておかないとすぐに錆びついてしまうし、気分転換になるのでたまに解いています。現時点で解いた問題数は次の通り。

![LeetCode status](/images/2018-year-end-reflection-leetcode.png)

アルゴリズムの解説を読み、エディタでコードを書き、正しく書けているかテストする、という一連の流れが全てオンライン上で完結するのは学習体験としてとても良いですね。私も学生の頃にこういうシステムを使って勉強したかった。

## キャリアの振り返り

育児休暇の間に自分の今までのキャリアについてぼんやりと考えていたのですが、細かいことがだんだん思い出せなくなっていることに気づきました。そこで当時の気持ちを忘れないうちに今までの振り返りをしようと思い、「[就職して 6 年過ぎた](/2018/04/06/six-years-reflection)」という記事を書きました。

## 参加した技術系イベント

今年はほぼ登壇する機会がなかったので、来年はもう少し何か発表したい。

- 11/26 ["GO GLOBAL" meetup #1](https://go-global.connpass.com/event/108021/)
- 10/10 [NVM Casual Talks #1](https://nvmct.connpass.com/event/98375/)
(https://http2study.connpass.com/event/81469/)
- 04/18 - 04/19 [BlinkOn 9](https://docs.google.com/document/d/e/2PACX-1vSUiIugjqPJgKeKqn0uVJ9gCSwG8CquMQPhkM4G6RITiv2YliJ_98e3909Z2kEXUSQ1YEaWMSuANpd9/pub)
  - Worker/Worklet の ES Modules 対応状況について発表しました。
- 04/13 [CDN Study (Akamai/Fastly)](https://http2study.connpass.com/event/81469/)
- 02/23 [webpackaging_study](https://web-study.connpass.com/event/78978/)

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">BlinkOn 9 に来た。メイン会場は二階席もあってデカい！ <a href="https://t.co/HfDjA3TXcM">pic.twitter.com/HfDjA3TXcM</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/986645797931384833?ref_src=twsrc%5Etfw">2018年4月18日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# 趣味関係

## 英会話レッスン

7 月から英会話レッスンを受け始めました。以前別の会社のビジネス英会話やオンライン英会話を受講していたんですが、レッスン内容が楽しくなかったり、受講スタイルが合わなかったりでしばらく止めていました。今回は週一通いのカジュアルなグループディスカッションのレッスンを受けています。英語表現や英会話の学習になるのはもちろん、気分転換にもなってとても楽しいです。しばらく続けようと思ってます。

## 天文宇宙検定

今年も 10 月に天文宇宙検定に挑戦しました。昨年は二級に合格したので、今年は最上位級である一級を受験しました。受験体験記は [「天文宇宙検定」一級受験期 (1st try)](/2018/11/27/astronomy-space-test-2018-1st-grade)」という記事にまとめました。残念ながら不合格だったので、来年リベンジしたいです。

![看板](/images/astronomy-space-test-2018-1st-grade-signboard.jpg)

## 統計検定

天文宇宙検定が終わり、次に何の勉強をしようかと考えた結果、12 月から統計検定の勉強を始めました。大学で確率論や統計学の授業を適当に受けていたせいで苦手意識があり、それが長年気がかりだったのが勉強を始めた動機です。基礎からしっかりやろうと一番簡単な四級の教材から勉強しています。これについては別途記事を書こうと思ってます。

## 読書

2018 年に読み終えた本は以下の通り。リンク先は読書メモです。

コンピュータサイエンス関係で読んだ本。

- [7 つの言語 7 つの世界](/2018/05/29/book-seven-languages-seven-worlds)
- [江添亮の詳説 C++17](/2018/05/10/book-cpp17)
- [量子コンピュータが人工知能を加速する](/2018/01/27/book-quantum-annealer-accelerates-artificial-intelligence)
- [[試して理解] Linux のしくみ ― 実験と図解で学ぶ OS とハードウェアの基礎知識](/2018/03/02/book-mechanism-of-linux)
- [並行コンピューティング技法 ― 実践マルチコア/マルチスレッドプログラミング (The Art of Concurrency)](/2018/06/21/book-the-art-of-concurrency)
- [アカマイ ― 知られざるインターネットの巨人](/2018/04/21/book-akamai)
- [エンジニアのためのマネジメントキャリアパス ― テックリードから CTO までマネジメントスキル向上ガイド](/2018/11/21/book-the-managers-path)
- [サイバー攻撃 ― ネット世界の裏側で起きていること](/2018/09/04/book-cyber-attack)
- [エンジニアリング組織論への招待 ― 不確実性に向き合う思考と組織のリファクタリング](/2018/04/17/book-engineering-organization-theory)

宇宙や天文宇宙検定関係の本。

- [超・宇宙を解く―現代天文学演習](https://twitter.com/nhiroki_/status/1048394714334945280)
- [天文宇宙検定公式問題集 1 級 (2018-2019 年版)](https://twitter.com/nhiroki_/status/1048213827131596800)
- [天文宇宙検定公式問題集 1 級 (2016-2017 年版)](https://twitter.com/nhiroki_/status/952900695413305344)
- [宇宙背景放射 ― 「ビッグバン以前」の痕跡を探る](/2018/10/01/book-cosmic-microwave-background)
- [天文の世界史](/2018/09/17/book-the-world-history-of-astronomy)
- [宇宙ビジネスの衝撃 ― 21 世紀の黄金をめぐる新時代のゴールドラッシュ](/2018/08/31/book-impact-of-aerospace-business)
- [クェーサーの謎 ― 宇宙でもっともミステリアスな天体](https://twitter.com/nhiroki_/status/963051916082126849)

統計検定の本。

- [統計検定 4 級対応「資料の活用」](https://twitter.com/nhiroki_/status/1071403140035923969)

子どもの教育に関する本。次男の育児休暇中に読んでました。

- [自分からどんどん勉強する子になる方法](/2018/02/18/book-child-self-study)
- [ほんとうに頭がよくなる世界最高の子ども英語](/2018/02/18/book-child-english)
- [言うこと聞かない！落ち着きない！ 男の子のしつけに悩んだら読む本](/2018/01/27/book-how-to-bring-up-boys)
- [ルポ塾歴社会　日本のエリート教育を牛耳る「鉄緑会」と「サピックス」の正体](/2018/01/23/book-tetsuryokukai-and-sapix)
- [なぜ、東大生の3人に1人が公文式なのか？](/2018/01/19/book-kumon)
- [3男1女 東大理III合格百発百中 絶対やるべき勉強法](/2018/01/14/book-ut-risan)

その他、雑多に読んだ本。

- [はじめての確定拠出年金投資](https://twitter.com/nhiroki_/status/958356246406488065)
- [ピアニストならだれでも知っておきたい「からだ」のこと](/2018/12/09/book-what-every-pianist-needs-to-know-about-the-body)
- [Google をつくった 3 人の男](/2018/11/14/book-founders-of-google)

## ゲーム

今年も beatmania IIDX をたくさんやりました。昨年の振り返りで

> 左手が以前よりも思い通りに動くようになってきたので、この調子で CANNON BALLERS の間に十段、あわよくば中伝目指して頑張りたい。

と書いたんですが、結局 CANNON BALLERS の間に十段を取れませんでした。DP12 のクリアランプもだいぶ増えてきて成長を実感してはいるので、2019 年こそ DP 十段取得を達成したいです。

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">CANNON BALLERS やり納めた。今作の振り返り: SP はクロペンと Plan 8 のクリアと 3y3s のハードクリアができたのがとても嬉しかった。DP は十段を取れなかったけど日々成長を実感しているので、次回作では中伝までいけるよう頑張りたい。CB は印象に残る曲が多くてとても楽しかったです <a href="https://twitter.com/hashtag/iidx?src=hash&amp;ref_src=twsrc%5Etfw">#iidx</a> <a href="https://t.co/dMd4OOZTtW">pic.twitter.com/dMd4OOZTtW</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1059773197476741121?ref_src=twsrc%5Etfw">2018年11月6日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

SP の方はクロペンと Plan 8 の SPA ノーマルゲージクリア、3y3s のハードクリアができたのがハイライト。あと、TROOPERS と mosaic で SP12 初フルコンボできた。前作のクロペンの結果と並べてみると、スコアがだいぶ上がってることが分かる。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">クロペン SPA をイージークリアしてから早一年、ついにノーマルクリアした。乱付き。気まぐれにハイスピを上げたら譜面がよく見えた。ハイスピ上げてこう <a href="https://t.co/4Yj5h77EUZ">https://t.co/4Yj5h77EUZ</a> <a href="https://twitter.com/hashtag/iidx?src=hash&amp;ref_src=twsrc%5Etfw">#iidx</a> <a href="https://t.co/R92sJyr1f4">pic.twitter.com/R92sJyr1f4</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1030065355215986688?ref_src=twsrc%5Etfw">2018年8月16日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

一年遅れですが『ゼルダの伝説 ブレス オブ ザ ワイルド』にどハマリして一時期ずっとやってました。一応エンディングまでプレイしたんですが、冒険を終わりにするのが寂しくて祠などの回収やエキスパンションパス (DLC) にまだ手を付けてないので、来年もチマチマやりたい。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">ゼルダをやり始めた。わりと自由に動き回れるようになったのでいっちょハイラル城目指すかと思って草原を駆け抜けてたらガーディアンに焼き払われた</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/993483507765657600?ref_src=twsrc%5Etfw">2018年5月7日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## アニメと映画

次男誕生により、アニメを観る時間がさらに減ってしまった一年でした。アニメもっと観たい。新作で一通り最終回まで観たアニメは次の通りでした。

- ゴブリンスレイヤー
- とある魔術の禁書目録 III
- 進撃の巨人 Season 3
- PERSONA5 the Animation
- シュタインズ・ゲート ゼロ
- ロクでなし魔術講師と禁忌教典
- クジラの子らは砂上に歌う

旧作ですが以下のアニメも一通り観ました。ヨルムンガンドは名作。

- ヨルムンガンド PERFECT ORDER　(二週目)
- ヨルムンガンド (二週目)
- ソードアート・オンライン オルタナティブ ガンゲイル・オンライン
- serial experiments lain
- 攻殻機動隊 (新劇場版)
- 映画かいけつゾロリ まもるぜ!きょうりゅうのたまご

実写映画は次の通り。全然観てないな。

- ブレードランナー
- ブレードランナー 2049
- ゴースト・イン・ザ・シェル (実写版)

# 今年買ってよかった物

## 結露取りワイパー

長男に渡したら毎朝窓の結露を取ってくれるようになりました。楽しみながら家族内での役割を果たすことができてとても嬉しそうです。

## 食洗機

据え置き型の食洗機を導入しました。スペースの都合で中サイズの食洗機にしたため収納できるお皿の大きさに限りがあり、一回の洗浄であまり多くは洗うことはできませんが、それでもスプーンやコップといった細々としたものを一々手洗いする必要がないだけでかなりの負担軽減になりました。また食器を食洗機内にどのように並べるかパズル的な要素もあり、食器洗いが楽しいことになったのも良かったです。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">紆余曲折を経て、ついに我が家に食洗機が導入された。期待以上に綺麗に洗ってくれて良い感じ！</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/967365094790217729?ref_src=twsrc%5Etfw">2018年2月24日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Pixel 3

ポートレートモードや夜景モードで写真が綺麗に撮れて楽しい。一眼レフカメラを持ち出す機会が減りそうです。バッテリーがもっと保つとなお嬉しい。

## 日経サイエンス

キャンペーンで安くなってるときに日経サイエンスの二年間定期購読を申し込みました。一番の目当ては宇宙関係の記事なんですが、それ以外の分野の記事を読むきっかけにもなって興味の幅が広がりました。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">日経サイエンス、毎月買うのめんどいので二年間の定期購読を申し込んだ。元から普通に買うより安いんだけど、3/30 までさらに 10% 安かった <a href="https://t.co/6gQFi4gjDf">https://t.co/6gQFi4gjDf</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/977895699093204992?ref_src=twsrc%5Etfw">2018年3月25日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# まとめ

子どもが二人になって大変でしたが、幸せな一年でした。来年も楽しく健康に過ごしたいです。