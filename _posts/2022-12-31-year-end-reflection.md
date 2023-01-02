---
layout: post
title:  "2022 年の振り返り"
date: 2022-12-31 00:00:00 +09:00
tags: text
image: /images/2022-year-end-reflection-miyako.webp
---

2022 年の雑多な振り返り兼備忘録です。今年で 36 歳になりました。

![宮古島](/images/2022-year-end-reflection-miyako.webp)

<p class="caption">クリスマスを宮古島で過ごしました。今年は異例の寒さでした。</p>

- [2021 年の振り返り](/2021/12/31/year-end-reflection)
- [2020 年の振り返り](/2020/12/31/year-end-reflection)
- [2019 年の振り返り](/2019/12/31/year-end-reflection)
- [2018 年の振り返り](/2018/12/31/year-end-reflection)
- [2017 年の振り返り](/2017/12/31/year-end-reflection)

# 生活

- 家族が一匹増えました（ハムスター）

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">買ったばかりのハムスターの餌、自転車のかごに載せてたらカラスに袋食いちぎられた…</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1535954540029419520?ref_src=twsrc%5Etfw">June 12, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

- 上の子が小学校二年生、下の子が幼稚園年中になり、子育て的には比較的安定した年でした。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">子どもと近所の広場まで皆既月食を見に行って、宇宙談義を楽しんだ。周りにも人がたくさんいて、さながらお祭りみたいだった。子ども曰く、あれはどんぐりらしい。自分には鈴カステラに見える。 <a href="https://t.co/KgpFwiVKpT">pic.twitter.com/KgpFwiVKpT</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1589946880704020480?ref_src=twsrc%5Etfw">November 8, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

- 閃輝暗点を伴う偏頭痛を頻発するようになり、薬を飲み始めました。初めて閃輝暗点になったときはびっくり＆とても怖くて、もしや脳卒中なんじゃないかって心配になったんですが、MRI を撮った結果脳に異常は見つからなくて一安心しました。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">視野がキラキラグネグネして見えにくくなって頭痛がする症状に二晩連続で悩まされ病院へ。検査した結果どうやら偏頭痛らしい。偏頭痛ってその前兆で閃輝暗点っていう視野異常も起きるとは知りませんでした。カラフルなギザギザがいっぱい見えて、文字や空間がうまく認識できなくなってすごい怖かった</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1527202898379440128?ref_src=twsrc%5Etfw">May 19, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

- YouTube のゲーム配信を定期的に観るようになりました。特に音ゲー配信 (BEMANI PRO LEAGUE や元プロの配信) やマイクラ配信をよく観ていました。うまい人のプレイを観ていると自分のモチベーションも上がります。

# 仕事

- 2022 年 4 月で就職してから丸 10 年経ちました。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">大学院を修了し、現職に入社してから丸 10 年経ちました。1 年を振り返るとあっという間に感じますが、10 年を振り返るとその長さや変化の大きさに驚きます。次の 10 年も毎日できることをちょっとずつ、けれど振り返るとその変化に驚くようなものにしたいです。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1509696455606685698?ref_src=twsrc%5Etfw">April 1, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

- 引き続きソフトウェアエンジニアとして Chrome (Chromium) ウェブブラウザの開発をしていました。 主に Prerender2 という機能を開発していたんですが、二年ほどの開発期間を経てようやくリリースすることができました。Prerender2 はユーザが次に訪れそうなウェブページをプリレンダリング（事前に読み込み・実行）することで、実際のナビゲーションを即座に行う機能です。Core Web Vitals の中で特に Largest Contentful Paint (LCP) に対して大きなインパクトが期待できます。ウェブページからは Speculation Rules という API で事前読み込みする URL を指定できます。他にもブラウザの URL バーへの入力に対して Chrome が自動的にプリレンダリングすることもあります。詳しくは[こちらの記事](https://developer.chrome.com/blog/prerender-pages/)を見てください。2023 年は Prerender2 の改良と適用できるケースを増やしていき、ウェブ全体の LCP 向上に寄与できるよう開発を進めていきます。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">Prerender2 (Speculation Rules API もしくは Omnibox からの prerendering) について今日現在世界で一番詳しい記事が公開されました。是非読んで試してフィードバックを頂けると嬉しいです！！ / &quot;Prerender pages in Chrome for instant page navigations&quot; <a href="https://t.co/buFtFJLLE0">https://t.co/buFtFJLLE0</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1598666130650206208?ref_src=twsrc%5Etfw">December 2, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

- 4 月から Tech Lead Manager (TLM) になりました。TLM は Tech Lead (TL) をやりつつ Engineering Manager (EM) もこなすポジションで、いわゆるプレイングマネージャーです。人事・技術の二つの視点からチームをリードすることが期待され、さらに Individual Contributor (IC) としての成果も引き続き求められます。まさか自分がマネージャーをやる日が来るだなんて全く想像していませんでしたが、いざやってみると毎日新しい学びがわんさかあってとても面白いです。目下の目標は、自分がチームに一番貢献できる EM/TL/IC 業のバランスを模索し、メンバーの幸福感とパフォーマンスを最大化するためのサポートをしていくことです。もう少し思考整理できたら記事を書きたいと思っています。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">年々自分で手を動かすんじゃなくて他の人に作業をお願いする機会が増えているんだけど、お願いした後も継続的に壁打ち相手になるにはドメイン知識をキャッチアップし続ける必要があり、一方で自分が手を動かしていないドメインの知識をキャッチアップし続けるのはなかなか難しく。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1582016238448152579?ref_src=twsrc%5Etfw">October 17, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

- 完全リモートワークからハイブリッドワークに移行し、基本的にはオフィスで仕事をするようになりました。そのおかげで以前よりも出歩く機会が増え、生活のリズムもとりやすくなりました。
- 今年もインターンの学生さんをホストし、三ヶ月に渡って Chrome の開発に参加していただきました。来年もインターンの募集をすると思うので、Chrome の開発に興味のある学生さんは是非最新情報をチェックしてご応募ください。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">ホストしていたインターンの方の発表が無事に終わって一気に気が抜けた。毎回言ってるけど、ただ発表聞いてるだけのホストもめちゃ緊張するんですよね😂</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1585564235677184000?ref_src=twsrc%5Etfw">October 27, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

- 今年私が Chromium にコミットしたパッチは 131 個、レビューしたコミット済みパッチは 519 個でした。TLM になって作文やディスカッションをする時間が増え、もっとコミット数が減ってると思っていましたが、トリビアルなものが多いとはいえ意外とキープできていて良かったです。

![Chromium CLs](/images/2022-year-end-reflection-chromium-cls.png)

||コミット数|レビュー数|備考|
|2022|131|519|マネージャーになった|
|2021|172|385|チームリードになった|
|2020|187|358|コロナウイルスによるコードフリーズ、プロジェクト変更|
|2019|259|385||
|2018|181|275|育児休暇 (二ヶ月)|
|2017|240|300||

# ブログ記事

ウェブ関連仕様を読み解いて分かったことを記事にまとめました。

- [fetch() が data URL へリダイレクトしたときの仕様を読み解く](/2022/11/12/redirect-to-data-url) (11/12)
- [Service Worker の FetchEvent がネットワークフォールバックした場合の仕様を読み解く](/2022/09/10/service-worker-fetch-event-network-fallback) (09/10)

普段ぼんやり考えていることをちゃんと言語化してみる記事を書きました。

- [百聞は今も一見に如かずなのか？](/2022/05/06/hyakubun-ikken) (05/06)
- [他人ではなく過去の自分と比較する](/2022/03/29/stop-comparing-yourself-to-others) (03/29)

書きかけのネタはいっぱいあるけど、遅筆でなかなか公開まで持っていけず。毎年言ってる気がするけど、来年はもっと頑張りましょう。

# 天文宇宙検定

今年も天文宇宙検定の一級を受けたんですが残念ながら不合格でした。来年も受けます。

- [天文宇宙検定 1 級受験記 (6th try)](/2022/11/21/astronomy-space-test-2022-14-1st-grade)
- [天文宇宙検定 1 級受験記 (5th try)](/2022/05/30/astronomy-space-test-2022-1st-grade)

<blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">天文宇宙検定一級受けてきたんだけど、今回の問題セットはいつも以上に分からなかったです。現場からは以上です。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1594232256780185600?ref_src=twsrc%5Etfw">November 20, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# 今年読み終えた本

コンピュータサイエンス系の本をもっと色々読んでたはずなんですが、読み途中で積まれているものが多数。来年は読み終えることをもう少し意識したい。

- 33 地域の暮らしと文化が丸わかり！ 中国大陸大全
- [宇宙飛行士](/2022/12/21/book-astronaut)
- [マインドセット ― 「やればできる！」の研究](/2022/12/11/book-the-new-psychology-of-success)
- [A Philosophy of Software Design](/2022/11/23/book-a-philosophy-of-software-design)
- [コーチングよりも大切なカウンセリングの技術](/2022/09/25/book-counseling-techniques)
- [これからのマネジャーの教科書](/2022/09/18/book-middle-manager-textbook)
- [コンサル一年目が学ぶこと](/2022/09/04/book-things-first-year-consultants-learn)
- [2030 半導体の地政学 ― 戦略物資を支配するのは誰か](/2022/07/20/book-geopolitics-of-semiconductors)
- [聞こえているのに聞き取れない 聴覚情報処理障害 APD がラクになる本](/2022/07/16/book-auditory-processing-disorder)
- 決算書がスラスラわかる 財務 3 表一体理解法
- [複利で伸びる 1 つの習慣](/2022/07/11/book-atomic-habits)
- 難しい数式はまったくわかりませんが、相対性理論をおしえてください！
- [へんな星たち ― 天体物理学が挑んだ 10 の恒星](/2022/06/29/book-strange-stars)
- [宇宙を支配する「定数」 ― 万有引力から光速、プランク定数まで](/2022/05/14/book-constants-to-rule-the-universe)
- [女性と天文学](/2022/05/01/book-women-in-astronomy)
- オードリー・タンの思考 ― IQ よりも大切なこと
- [ブロックチェーン技術概論 ― 理論と実践](/2022/02/13/theory-and-practice-of-blockchain)
- [どんな仕事も「25 分 + 5 分」で結果が出る ― ポモドーロ・テクニック入門](/2022/01/22/book-the-pomodoro-technique)
- [リーンスタートアップ ― ムダのない起業プロセスでイノベーションを生みだす](/2022/01/16/book-the-lean-startup)
- [こどもサピエンス史 ― 生命の始まりから AI まで](/2022/01/02/book-history-of-sapiens-for-children)

『プロジェクト・ヘイル・メアリー』はものすごく久しぶりに読んだ小説でした。面白かった。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">すごく久しぶりに小説を読んだけど、没入感みたいなものがあってとても良かった。時間さえあればもっとたくさん読みたいんだけど、なかなか難しい😪</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1545188398209859585?ref_src=twsrc%5Etfw">July 7, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

あと定期購読している『日経サイエンス』を読みました。今年はあまり積まずに消化できました。

# 今年観終わったアニメ

結構いっぱい観ましたが、それを上回る早さで観たいアニメが積み上がってるのでどんどん消化したいです。

- アキバ冥途戦争
- アークナイツ【黎明前奏/PRELUDE TO DAWN】
- 天才王子の赤字国家再生術
- 失格紋の最強賢者
- 世界最高の暗殺者、異世界貴族に転生する
- ダンジョンに出会いを求めるのは 間違っているだろうか IV
- メイドインアビス 烈日の黄金郷
- リコリス・リコイル
- サイバーパンク エッジランナーズ
- Dr. Stone 龍水
- 盾の勇者の成り上がり (第二期)
- 攻殻機動隊 SAC_2045 (第二期)
- 勇者、辞めます
- 史上最強の大魔王、村人Aに転生する
- とある魔術の禁書目録 III
- 真の仲間じゃないと勇者のパーティーを追い出されたので、辺境でスローライフすることにしました
- リアデイルの大地にて
- ありふれた職業で世界最強 (第二期)
- 進撃の巨人 The Final Season Part 2
- 86-エイティシックス- (第二期)
- GATE 自衛隊 彼の地にて、斯く戦えり (第二期)
- 鬼滅の刃 遊郭編
- コードギアス 反逆のルルーシュ I 興道
- コードギアス 反逆のルルーシュ II 叛道
- コードギアス 反逆のルルーシュ III 皇道
- コードギアス 復活のルルーシュ
- 劇場版 ソードアート・オンライン -オーディナル・スケール-
- 魔法科高校の劣等生 追憶編

この中だと『サイバーパンク エッジランナーズ』が一番心を揺さぶられました。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">いいもの観た。記憶なくしてもう一回観たい・・・</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1574811937774641153?ref_src=twsrc%5Etfw">September 27, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

# 今年やったゲーム

音ゲー以外は家族みんなでできるゲームを中心にやってました。

## beatmania IIDX

ライフワーク。INFINITAS（家庭用）では新たに専用マシン (ThinkCentre M75q Tiny Gen2) を買い、SP12 のランプ埋めと AAA 狙いを頑張りました。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">Thor&#39;s Hammer のハードが付いたので、未クリア未ハードクリアが 3+5 になった <a href="https://twitter.com/hashtag/iidx?src=hash&amp;ref_src=twsrc%5Etfw">#iidx</a> <a href="https://twitter.com/hashtag/infinitas?src=hash&amp;ref_src=twsrc%5Etfw">#infinitas</a> <a href="https://t.co/R1j4qvKBGG">pic.twitter.com/R1j4qvKBGG</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1598632124185448448?ref_src=twsrc%5Etfw">December 2, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

ついに AA の A の AAA が出せて嬉しかった！

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">AA の A の AAA きた！Lift で判定調整したら一発で出せた。後半緊張したー <a href="https://twitter.com/hashtag/iidx?src=hash&amp;ref_src=twsrc%5Etfw">#iidx</a> <a href="https://t.co/uhiiZNwqQi">pic.twitter.com/uhiiZNwqQi</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1568420023462690818?ref_src=twsrc%5Etfw">September 10, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

悲願だったクロペンのハードクリア！

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">悲願だったクロペン (SPA) ハードクリア！クリアしそうになると鼓動が早くなって苦しくなる呪いにかかって挫けかけてたのでめっちゃ嬉しい！ハイスピ少し上げて「なんかいけそう？」って他人事のように考えながらワチャワチャやってたら抜けられました。これでようやくスコア上げに集中できる😭 <a href="https://twitter.com/hashtag/iidx?src=hash&amp;ref_src=twsrc%5Etfw">#iidx</a> <a href="https://t.co/ieOZNhaJ8j">https://t.co/ieOZNhaJ8j</a> <a href="https://t.co/eEO5QvNMsc">pic.twitter.com/eEO5QvNMsc</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1524711830924718082?ref_src=twsrc%5Etfw">May 12, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

出社再開により、アーケードの方も再開しました。[INFINITAS で使っている PHOENIX WAN](/2021/07/20/iidx-at-home) のスクラッチが反応しなくなり、家で DP ができなくなってしまったので、アーケードではもっぱら DP を中心にやっていました。

## ポケットモンスター　スカーレット・バイオレット

家族みんなでやってます。面白いです。[感想記事](/2022/12/22/game-pokemon-scarlet)を書きました。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">妻と協力して何とかリザードン倒せた！私はフラージェス、妻はニンフィアで、バフかけたあとはひたすらムーンフォースもしくはドレインキッスで削りました。フラージェスのテラスタイプが揃えられればもう少し楽だった。 <a href="https://t.co/AfKpng9NsA">pic.twitter.com/AfKpng9NsA</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1604521527994908675?ref_src=twsrc%5Etfw">December 18, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

## ポケットモンスター　アルセウス

アルセウスも家族みんなでやってました。家族でポケモンの生息域の情報を教え合ったり、図鑑の完成度を競ったりしてとても楽しかったです。感想記事書きそこねた！

## ゼルダ無双 厄災の黙示録（エキスパンション・パス）

[本編は昨年クリアした](/2021/05/14/game-hyrule-warriors-age-of-calamity)んですが、エキスパンション・パスをやっていませんでした。今年長男が本編をクリアしたので、それを機にエキスパンション・パスを買ってやりはじめました。まだ最後までやってないんですが、ストーリーを補完するクエストや高難易度クエストがあって楽しいです。

長男はブレスオブザワイルドの方にもハマっていて、YouTube でグリッチ技を仕入れて実践していて楽しそうです。来年はティアーズオブザキングダムも出るので楽しみです。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">長男がゼルダの祠の名前を無限に連呼し続けるせいで頭が爆発しそう😂</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1605133752757166080?ref_src=twsrc%5Etfw">December 20, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

## Minecraft

今年も子どもたちと一緒に遊びました。The Wild Update 楽しかったです。次期大型アップデートも楽しみ。

<div class="twitter-wrapper"><blockquote class="twitter-tweet" data-dnt="true"><p lang="ja" dir="ltr">長男、マイクラのコマンドブロックはすっかり自由自在に使えるようになり、分からないことは YouTube 動画やネット記事などで知らぬ間に勉強して色々作るようになった。コマンドの動作原理を尋ねると順序立てて説明してくれる。自走できるようになるとどんどん勝手に伸びていくな。</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1515368567025258496?ref_src=twsrc%5Etfw">April 16, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script></div>

# おわりに

2022 年はマネージャーになったり、偏頭痛などの体調不良が多かったりでしたが、比較的落ち着いた一年でした。2023 年も穏やかに楽しく過ごせるといいですね。