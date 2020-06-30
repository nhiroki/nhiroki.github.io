---
layout: post
title: "読書｜Real World HTTP ― 歴史とコードに学ぶインターネットとウェブ技術 [第 2 版]"
date: 2020-06-30 00:00:00 +09:00
tags: book
image: /images/book-real-world-http-2nd-edition.jpg
---

『Real World HTTP ― 歴史とコードに学ぶインターネットとウェブ技術』の第 2 版の読書メモです。

第 1 版が素晴らしい内容だった ([当時の読書メモ](/2017/08/10/book-real-world-http)) ので、改訂版が出たらまた読もうと思っていました。第 2 版も素晴らしかったです。ウェブ関係でおすすめの本を聞かれたら真っ先にこれを紹介しています。第 3 版が出たらまた読みます。

![表紙](/images/book-real-world-http-2nd-edition.jpg)

# 読書メモ

[Twitter でのメモ書き](https://twitter.com/nhiroki_/status/1262576631031316480)

まえがきに次のように書かれています。

> 現在策定中、構想中の未来の技術などを網羅するのは不可能です（中略）基本的にはすでに実現されていて、使われているものを中心に扱っています（中略）まだ特定ブラウザのみで実装されている機能も取り上げていません。

私はウェブブラウザの開発者なので、あえてウェブブラウザ側の事情を付け加えながら読んでみました。

## 第 1 章：HTTP/1.0 の世界：基本となる 4 つの要素

HTTP の歴史 (0.9 から 1.0)、HTTP の先祖 (電子メール、ニュースグループ) から受け継いだプロトコル、リダイレクト、URL など。URLSearchParams が URL エンコーディングしてくれるの知らなかった。

**[1.4.4]** Content Type Sniffing の話。新しく入るブラウザ機能はデフォルトで sniffing をしないようになっています。例えば module script は JavaScript MIME type が明示的に指定されていないと弾くように HTML 仕様で定義されています (下記参照)。昔からある classic script は sniffing します。

> For historical reasons, fetching a classic script does not include MIME type checking. In contrast, module scripts will fail to load if they are not of a correct MIME type.
>
> [https://html.spec.whatwg.org/multipage/webappapis.html#fetch-a-single-module-script](https://html.spec.whatwg.org/multipage/webappapis.html#fetch-a-single-module-script)

一方、昔からある機能でも sniffing しないように徐々に変更が加えられています。例えば Chromium では [classic worker script で sniffing をやめる取り組み](https://www.chromestatus.com/feature/6037497138118656)が行われています。

**[1.7]** URL はユーザに提供される数少ないセキュリティ情報となるため、それをどのように提示すべきかブラウザ開発者は色々検討していて、Chromium ではそれを[ガイドライン](https://source.chromium.org/chromium/chromium/src/+/master:docs/security/url_display_guidelines/url_display_guidelines.md)にまとめています。ブラウザ以外のアプリで URL を扱う場合の参考になるかもしれません。

## 第 2 章：HTTP/1.0 のセマンティクス：ブラウザの基本機能の裏側

フォームの送信（ファイル、リダイレクトなど）、コンテントネゴシエーション、クッキー、認証とセッション、プロキシ、キャッシュ、リファラ、検索エンジン向けのアクセス制御 (robots.txt) など。

**[2.4.4]** ブラウザが通信に使っている圧縮アルゴリズム (deflate, gzip) は [Compression Streams / Decompression Streams](https://wicg.github.io/compression/) という API を通して JavaScript で使えるようになりました。[Chrome ではバージョン 80 から使えます](https://www.chromestatus.com/feature/5855937971617792)。

**[2.5]** Chrome で[新しい Cookie Store API](https://wicg.github.io/cookie-store/explainer.html) の experiment をしています。従来の document.cookie は同期的な API であり、同期的な API を禁止している Service Worker などで使用することができませんでしたが、新しい Cookie Store は非同期な API になっていて Service Worker で使えたり変更をイベントで取れたりします。

**[2.5.4]** same origin が「同一サイト」と表記されてますが、[HTML 仕様上は same site は違う意味を持つ](https://html.spec.whatwg.org/multipage/origin.html#same-site)ので注意が必要です。本書内の例だと www.example.com と example.com は「別のサイト扱い」となっていますが、これは same site になります。

**[2.5.5]** cookie における same-site は [schemeless](https://html.spec.whatwg.org/multipage/origin.html#schemelessly-same-site) で http と https が same-site として扱われるんですが、これを [schemeful](https://www.chromestatus.com/feature/5096179480133632) にして CSRF を防ごうという取り組みがあります。

**[2.9]** Referrer Policy のデフォルト値が [no-referrer-when-downgrade から strict-origin-when-crossorigin に変わりそう](https://www.chromestatus.com/feature/6251880185331712)です。ちなみに Referrer Policy はポリシーの種類やコンテキスト間での継承ルールが多岐に渡ることから generator を作って Web Platform Tests のテストケースを自動生成しているんですが、組み合わせが多すぎてテストを走らせるのが困難だったりします。最近、同僚とインターンの方が頑張ってくれてだいぶ良くなりました。

**[2.11]** [ユーザーエージェント文字列は凍結・非推奨](https://groups.google.com/a/chromium.org/forum/#!msg/blink-dev/-2JIRNMWJ7s/yHe4tQNLCgAJ)になり、代わりに [UA Client Hints](https://groups.google.com/a/chromium.org/forum/#!msg/blink-dev/A4wxFpvqUfA/g7iccl9ICgAJ) を使うように変更されていく流れです。ウェブにとって大きな転換になりそうです。ちょうど web.dev で[詳しい解説記事](https://web.dev/user-agent-client-hints/)が公開されました。

## 第 3 章：Go 言語による HTTP/1.0 クライアントの実装

Go 言語を使った GET や POST リクエストなどの送受信方法、マルチパートフォームデータやクッキー、プロキシの使い方、タイムアウトの設定、など。

## 第 4 章：HTTP/1.1 のシンタックス：高速化と安全性を求めた拡張

Keep-Alive、パイプライニング、TLS、PUT/DELETE メソッドと CRUD、プロトコルアップグレード、バーチャルホスト、chunked transfer encoding、data URL など。

## 第 5 章：HTTP/1.1 のセマンティクス：広がる HTTP の用途

ローカル保存、ダウンロードの中断再開、XHR、Comet、ジオロケーション、Server ヘッダ、RPC、WebDAV、各種認証と認可の仕組み、など。

## 第 6 章：Go 言語による HTTP1.1 クライアントの実装

Go による各機能の実現方法。TLS、証明書の作り方、プロトコルアップグレード、チャンク、RPC など。

## 第 7 章：HTTP/2、HTTP/3 のシンタックス：プロトコルの再定義

HTTP/2 プロトコルの概説、HTTP/2 Server Push、リソースプリロード、HTTP/3 プロトコルの概説、プロトコルネゴシエーション、JS API (Fetch API, Server-Sent Events, WebSocket, WebRTC, WebPush) など。

**[7.2.7]** Chrome における prerender はその名に反してリソースのプリフェッチをするだけでプリレンダリングはしていません。メモリ消費量や機能上の懸念によりそのような実装になっています。詳しくは NoState Prefetch について書かれた[こちらの記事](https://developers.google.com/web/updates/2018/07/nostate-prefetch)を参照。

**[7.2.7]** 次回ナビゲーションのための先読みには preload ではなく prefetch を使う方が適切で、これは preload の仕様に明記されています。prefetch なら 3 秒以内にロードしなくても警告が出ません。

> prefetch is an optional and often low-priority fetch for a resource that might be used by a subsequent navigation; preload is a mandatory fetch for a resource that is necessary for the current navigation.
>
> [https://w3c.github.io/preload/#x2.link-type-preload](https://w3c.github.io/preload/#x2.link-type-preload)

**[7.5.5]** 以前 Audio Worker が試験実装されていましたがもうなくなりました。代わりに [Audio Worklet](https://developers.google.com/web/updates/2017/12/audio-worklet) の仕様が定義され、現在 Chromium-based browsers と Firefox で使えます。

## 第 8 章：HTTP/2 のセマンティクス：新しいユースケース

レスポンシブデザイン、セマンティックウェブ、オープングラフプロトコル、QR コード、AMP、DeepLink、動画ストリーミング (HLS, MPEG-DASH, CMAF)、など。

## 第 9 章：Go 言語による HTTP/2、HTML 5 のプロトコルの実装

HTTP/2 サーバプッシュ、Server-Sent Events、WebSocket の Go 言語によるサンプル実装の紹介。

## 第 10 章：クライアント視点で見る RESTful API

RESTful と REST-ish、 RESTful API におけるメソッド・ステータスコード・ボディー、Web API とトランザクション、REST API の実例、タイムアウトやアクセス数制限、など。

## 第 11 章：JavaScript によるブラウザからの動的な HTTP リクエスト

JavaScript から HTTP リクエストを発行する方法として、XHR、Fetch API、location.href や form submit によるリロード、anchor タグの download プロパティ、Server-Sent Events、WebSocket の使い方やオプションの紹介、など。

その他の networking API について。本章でちょこっと紹介されている Service Worker はブラウザ内プロキシとして HTTP リクエストを動的に処理したり、オフライン対応をするための基盤となる機能です。CacheStorage API は IndexedDB のようなストレージ API ですが、それ自体で HTTP リクエストを発行してストレージに格納することができます。その他本章で挙げられていないものでパッと思いつくものは、Fetch API の keepalive オプションと beacon API、Background Fetch API、Web Transport、dynamic import があります。

import についてもう少し。import で [HTML modules](https://www.chromestatus.com/feature/4854408103854080), [CSS modules](https://www.chromestatus.com/feature/5948572598009856), [JSON modules](https://www.chromestatus.com/feature/5749863620804608) を取得できるようにするという機能提案があって Chromium で途中まで実装されているんですが、[仕様面で色々あって](https://github.com/w3c/webcomponents/issues/839)最近動きがなさそうです・・・と思ったら JSON modules の方は最近動きがあるみたい。[HTML 仕様に JSON modules を再度追加する PR](https://github.com/whatwg/html/pull/5658) が議論されています。

## 第 12 章：ウェブアプリケーションの基礎

リクエストのライフサイクル、セッション、アプリ構成 (SSR, Ajax, SPA, SPA+SSR)、インフラ構成 (開発・本番環境、言語・フレームワーク毎の構成、PaaS, サーバレス、マイクロサービス)、ウェブ API の設計、CGI やリッチインターネットアプリケーション、など。

第 2 版で追加された章。ウェブアプリの構成方法をざっくり概観できて良かったです。特にウェブ API 設計において何にどんなデータを入れるかの話は、今まで経験的にやっていたことが明文化されていて整理できました。

今どきの人はこういうウェブアプリ構成に関する全般的な知識をどうやって学んでるんですかね？自分は学生の頃に WEB+DB PRESS plus の『サーバ/インフラを支える技術』とか『大規模サービス技術入門』とかを読んで、後は適当なフレームワークでアプリ作って実践しながら手探りで学んだ感じでした。最近もこういった書籍があるのかな？ウェブアプリ構成法みたいな本があっても良さそう。

## 第 13 章：クラウド時代の HTTP：ウェブを強くするさまざまな技術

DNS の仕組みと各種役割 (キャッシュ・ロードバランス・CDN への誘導・サービスディスカバリー)、リバースプロキシ、CDN、API ゲートウェイ、死活監視、VPC (Virtual Private Cloud)、分散トレーシング、など。

前章ではブラウザ・ウェブサーバ・アプリケーションサーバという最小構成に関する話題だったのに対し、本章ではクラウドサービス上でのアプリケーション構成、負荷分散や分散トレーシングといったより大規模な構成に関する話題を扱っている。

以下雑感。DNS の SRV レコード知らなかった。リセマラによる CDN 破産の話が面白い。Trace Context という仕様が W3C で議論されているの知らなかった。クラウドサービス上でのアプリケーション構成法は知らないことばかりで不勉強が身に染みる・・・。

## 第 14 章：セキュリティ：ブラウザを守る HTTP の機能

攻撃の種類 (XSS 、中間者攻撃 etc) とそれらを防ぐブラウザやフレームワークの仕組み、ウェブ広告やアクセス解析の仕組み、など。

**[14.3.2]** CSP のところで紹介されている reflected-xss ディレクティブの存在を知らなくてびっくりしたんだけど、どうやら [Chrome ではバージョン 56 で消されていて](https://www.chromestatus.com/feature/5769374145183744)、仕様にももうないみたい。