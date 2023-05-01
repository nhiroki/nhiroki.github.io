---
layout: page
title: About
date: 2023/05/01
permalink: /about/
---

![Profile](/images/profile.png)

Hiroki Nakagawa (nhiroki)

Staff Software Engineer / Tech Lead Manager (TLM) @ Google。ウェブブラウザ Chrome (Chromium) 開発者。情報理工学修士。2 児の父親。

---

**振り返り**

* 大学入学から大学院卒業までの話: 「[新卒のソフトウェアエンジニアになるまで](/2019/04/02/how-i-became-a-software-engineer)」
* グーグルに新卒入社してから 6 年間の話: 「[就職して 6 年過ぎた](/2018/04/06/six-years-reflection)」

**興味のあること**

世の中の色々なシステムの仕組みを知ることに興味があります。

- コンピュータサイエンス
  - ウェブブラウザ / 仕事で作っているので。
  - システムソフトウェア（オペレーティングシステム、ファイルシステム、プログラミング言語処理系、メモリ管理機構） / 仕組みを知るのが面白い。
  - コンピュータアーキテクチャ / [ソフトウェアとハードウェアの境界](/2017/07/26/software-and-hardware)に興味があり。上を知ったら下も知りたい。
  - 並列・並行プログラミング / スレッドやプロセスをどのように協調動作させるか考えるのが楽しい。
  - よく使うのは C++ と JavaScript。ウェブアプリケーションやちょっとしたツールには Ruby。勉強中なのは Rust。その前に勉強していたのは Haskell と Golang。ご無沙汰なのは Java と PHP。もう読めないのは Perl。エディタは Vim です。
- 物理学 / 特に宇宙物理学に興味があります。

**その他のこと**

- 二児の父親です（2015 年と 2018 年生まれの男の子）
- 音ゲー（beatmania IIDX）好きです。INFINITAS で [SP 皆伝](https://twitter.com/nhiroki_/status/1440637465790857216) / [DP 十段](https://twitter.com/nhiroki_/status/1449359776312266752)でした。
- 大学では学園祭実行委員会に所属していました（副委員長・総務局長）
- 中学高校では陸上競技部に所属していました（部長）。専門は短距離（200m, 400m）でした。

# 仕事

**2012.04 -- 現在　グーグル合同会社（旧：グーグル株式会社） / シニアソフトウェアエンジニア**

ウェブブラウザ Chrome の開発。Chrome のオープンソースプロジェクト Chromium のコミッター。主に以下のことをしていました。

* Prerender API の設計と実装 (TL / Manager)
  - [リソースの読み込みを助けるウェブブラウザ API の世界](/2021/05/06/resource-loading-apis)
* レンダリングエンジンに最適化ヒントを与える基盤の設計と実装 (IC)
* 103 Early Hints の評価メトリクスの設計と実装 (IC)
* Off-the-main-thread リソースローディングの設計と実装 (TL)
  - [ネットワーク API のメインスレッド依存をなくす話](/2018/06/15/blink-off-the-main-thread-loading)
  - [ウェブブラウザの off-the-main-thread API の話](/2018/05/07/off-the-main-thread-api)
* ES Modules for Workers API の設計と実装 (TL)
  - [Chrome 80 から Web Worker (Dedicated Worker) で ES Modules が使えます](/2019/12/05/es-modules-for-dedicated-workers)
  - [JavaScript のスクリプトインポートを正しく使い分けようという話](/2018/09/07/javascript-import)
* [Worklets API](https://drafts.css-houdini.org/worklets/) の設計と実装 (TL)
  - [Chrome 79 から Worklet.addModule() が詳細なエラーを返すようになった話](/2019/10/16/worklet-addmodule-error)
  - [WFH で Audio Worklets の使用率が増えている話](/2020/05/12/usage-of-audio-worklets)
* [Workers](https://html.spec.whatwg.org/multipage/workers.html#workers) / [Worklets](https://drafts.css-houdini.org/worklets/) API のスレッディング基盤の設計と実装 (TL)
  - [Chrome 69 で Web Worker から Web Worker を作れるようになった話](/2018/10/29/nested-workers)
  - [JavaScript のスレッド並列実行環境](/2017/12/10/javascript-parallel-processing)
* [CacheStorage API](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#cache-objects) の実装 (IC)
  - [Chrome 46 に Cache.addAll() を実装した](/2015/09/02/cache-storage-addall)
* [ServiceWorker API](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/) の設計と実装 (IC)
  - [Service Worker スクリプトのインストールと更新処理](/2018/02/15/service-worker-install-and-update-scripts)
  - [イベント駆動型サービス実行基盤としての Service Worker](/2017/02/13/service-worker-event-driven-background-processing)
  - [Service Worker のスコープとページコントロール](/2015/02/28/service-worker-scope-and-page-control)
* [chrome.syncFileSystem API](https://developer.chrome.com/apps/syncFileSystem) の設計と実装 (IC)
* PPAPI 用 FileSystem API の設計と実装 (IC)
* [FileSystem API](https://www.w3.org/TR/file-system-api/) / [QuotaManagement API](http://w3c.github.io/quota-api/) のメンテナンス (IC)

私が加えた変更の一覧は[こちら（2017 年 6 月以降）](https://chromium-review.googlesource.com/q/owner:nhiroki%2540chromium.org)と[こちら（2017 年 5 月以前）](https://codereview.chromium.org/search?closed=1&owner=nhiroki%40chromium.org&reviewer=&cc=&repo_guid=&base=&project=&private=1&commit=1&created_before=&created_after=&modified_before=&modified_after=&order=&format=html&keys_only=False&with_messages=False&cursor=&limit=200)で見ることができます。また、上記コンポーネントとレンダリングエンジン Blink の Core コンポーネントの[コードオーナーシップ](https://www.chromium.org/developers/owners-files)を持っています。主な使用言語は C++ / JavaScript です。

<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">WHATWG の HTML Standard に投げた PR がマージされて、貢献者リストに名前を載せてもらった！何年もウェブブラウザ開発してて今更感あるけどすごく嬉しい :D <a href="https://t.co/6eJ72XcMUg">https://t.co/6eJ72XcMUg</a> <a href="https://twitter.com/hashtag/nhspec?src=hash&amp;ref_src=twsrc%5Etfw">#nhspec</a> <a href="https://t.co/JnnwTouPe5">pic.twitter.com/JnnwTouPe5</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/999107453282209793?ref_src=twsrc%5Etfw">2018年5月23日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

**2011.02 -- 2012.03　アシアル株式会社 / プログラマー、システムエンジニア（アルバイト）**

HTML5 ハイブリッドアプリ開発プラットフォーム [Monaca](https://ja.monaca.io/) の開発チームで、iOS 用のリモートコンパイル環境の構築、リモートデバッガ・テンプレートエンジン・UI フレームワークの実装などをしました。使用言語は Objective-C です。

* 2012.02.06　[xcodebuild コマンドで iOS アプリの自動ビルド](http://blog.asial.co.jp/953)
* 2011.11.26　[Monaca デバッグログ機能で快適デバッグ](http://blog.asial.co.jp/942)
* 2011.11.05　[Monaca ネットワークインストール機能で快適実機検証](http://blog.asial.co.jp/936)
* 2011.09.17　[Monaca リモートフォルダ機能で快適プロジェクト管理](http://blog.asial.co.jp/926)

**2010.08 -- 2010.10　グーグル株式会社 / ソフトウェアエンジニア（インターン）**

Chrome OS の文字入力変換ウィンドウの UI 改善。Chromium とか iBus とか Mozc とかを変更しました。主な使用言語は C++ / Python です。

**それ以前**

ウェブクローラーの開発 (Perl / PHP)、Drupal を使った社内ポータルサイトの構築 (PHP)、OpenLDAP / OpenAM を使ったシングルサインオン環境の構築と SNS への組み込み (PHP)、E-Learning アプリケーションの開発 (Ruby on Rails) などをしていました。その他に二年ほど高校生の家庭教師をしていました。

# 勉強会などでの発表

**インタビュー**

* 2018.10.24　[Chromium から見る Web の未来](/2018/10/24/webdb-press-107-interview)（WEB+DB PRESS Vol. 107）

**発表**

* 2021.08.31　JavaScript 処理系 V8 とレンダリングエンジン Blink のアーキテクチャ（PPL サマースクール 2021）[[article](/2021/09/04/talk-browser-architecture), [slide](https://docs.google.com/presentation/d/e/2PACX-1vTbELnS3VWyK6sxxdTwcMWTNouiWm1wgOXBa_4214YOcz5coRTZW04U54DKk7jE2mIb5A31C4kYAxyN/pub)]
* 2020.12.04　ウェブの進化とウェブブラウザ開発の最前線（慶應義塾大学 情報工学特別講義）[[article](/2020/12/04/web-and-browser), [slide](https://docs.google.com/presentation/d/e/2PACX-1vQA7761ZtEk8uqs6wQ3sOsXo2B-IGpsvRHHftseFDoPTcE4Jq0TPCuX92GrLeL0D4wkJNhUk0jVxE3V/pub?start=false&loop=false&delayms=3000)]
* 2017.03.21　[論文紹介] The Linux Scheduler: A Decade of Wasted Cores (EuroSys 2016)（[システム系輪講会](https://connpass.com/event/52323/)）[[slide](https://docs.google.com/presentation/d/1B9lC6uPxHBzWm9Elhn8cvvQXy7ykIe7EPgFe0i0hAYk/pub?start=false&loop=false&slide=id.p)]
* 2016.02.06　IoT 時代の Web エンジニア（IT 分科会 #2）
* 2015.08.02　Mobile Web and Native App -Past, Present, Future-（IT 分科会 #1）
* 2015.04.22　#17 Service Worker（mozaic.fm）[[podcast](http://mozaic.fm/post/117004083098/17-service-worker)]
* 2015.04.04　Service Worker Primer（[Service Worker ハッカソン](https://developers-jp.googleblog.com/2015/03/service-worker.html)）[[slide](https://docs.google.com/presentation/d/1WiL241gQYOSAV6yVlM2_hloX-fDwzWHIZXqWhuEzdX0/pub?start=false&loop=false&delayms=3000), [video](https://www.youtube.com/watch?feature=player_embedded&v=FnS0MdLM5ZU#t=285)]
* 2014.06.27　LevelDB のちょっとした話（天下一 InfluxDB 勉強会）[[slide](https://docs.google.com/a/chromium.org/presentation/d/1rqB-7G1CD0PQ74UGr2OKqpwOxVenXYeJIU_FzgskvKA/edit#slide=id.p), [video](https://www.youtube.com/watch?v=gU42aRdohhM)]
* 2013.06.11　syncFileSystem API（Web オフライン技術研究会 #01、Web アーキテクチャ技術研究会 #01 by html5jえんぷら部）
* 2013.02.28　Introduction to NativeClient（第 4 回若手 Web エンジニア交流会）
* 2012.07.13　Web ブラウザが支える技術（第 1 回若手 Web エンジニア交流会）

# 大学

**2010.04 -- 2012.03　東京大学大学院　情報理工学系研究科創造情報学専攻 (笹田研究室)**

* Ruby オブジェクトの効率的なプロセス間転送・共有機構の設計と実装
* Ruby で使われている正規表現エンジン鬼車用の AOT コンパイラの実装

**2006.04 -- 2010.03　慶應義塾大学　理工学部情報工学科 (寺岡研究室)**

* ユーザモビリティを考慮した分散ファイルシステムの設計と実装
* 学園祭実行委員会 (副委員長 / 総務局長)

# 研究

査読付き論文

* 2012.09　中川 博貴, 笹田 耕一「Ruby オブジェクトの効率的なプロセス間転送・共有機構の設計と実装」（情報処理学会論文誌（PRO）Vol.5, No.4, pp.1-16）

口頭発表

* 2012.03　中川 博貴，笹田 耕一: Ruby オブジェクトの効率的なプロセス間転送・共有機構の設計と実装（情報処理学会 第88回 プログラミング研究発表会）
* 2011.09　中川 博貴，笹田 耕一: Teleporter: Ruby オブジェクトの効率的なプロセス間転送・共有機構（ソフトウェア科学会 第28回大会 講演論文集）
* 2011.04　中川 博貴，笹田 耕一: Rubyの文字列処理の手軽な高速化（情報処理学会 第83回 プログラミング研究発表会）

# 資格・検定

* 2019　天文宇宙検定準 1 級（[受験記](/2019/12/21/astronomy-space-test-2019-1st-grade)）
* 2019　統計検定 2 級（[受験記](/2019/06/21/japan-statistical-society-certificate-2nd-grade)）
* 2017　天文宇宙検定 2 級（[受験記](/2017/12/13/astro-test-2nd-grade)）
* 2015　TOEIC 965
* 2010　ネットワークスペシャリスト
* 2010　情報セキュリティスペシャリスト
* 2008　ソフトウェア開発技術者
* 2007　基本情報処理技術者

# コンテスト

* ACM/ICPC Asia Regional Contest 2008 in Aizu / Participation
* ACM/ICPC Asia Regional Contest 2007 in Tokyo / The 17th place
