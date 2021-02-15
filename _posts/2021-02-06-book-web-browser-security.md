---
layout: post
title: "読書｜Web ブラウザセキュリティ"
date: 2021-02-06 00:00:00 +09:00
tags: book
image: /images/book-web-browser-security.webp
---

『Web ブラウザセキュリティ ― Web アプリケーションの安全性を支える仕組みを整理する』の読書メモです。

![表紙](/images/book-web-browser-security.webp)

# 感想

私は Chromium ブラウザの開発に携わっており、本書で紹介されている機能の一部の実装者の立場で読みました。

- ブラウザの持つセキュリティ機能が体系立てて紹介されており、機能間の関係性が分かりやすくまとめられている。特に最新のブラウザセキュリティ機能はブラウザの内部構成 (例えばプロセス分離) に強く依存しているので、ブラウザが何をどう守りたいのか抽象的なところから理解していかないと各機能の必要性が分かりにくいが、この本はその抽象的なところからしっかり整理してくれているのが良かった。
- プロセス分離に関して恐らく日本語で最も詳しく整理された資料だと思います。過去の研究実装から最新の仕様までしっかり言及されていて、ここまで調べてまとめあげるのは大変だったと思います。これからウェブブラウザ開発に参加する人に「これ読んで」と渡せて便利な一冊です。

著者やレビュアーによる書籍制作の話

- [きまぐれログ #3: 書籍製作の振り返り](https://diary.shift-js.info/kimagure-03/)
- [書籍「Webブラウザセキュリティ」発刊記念 著者＆レビュアー対談(前編)](https://techblog.securesky-tech.com/entry/2021/01/08/)
- [書籍「Webブラウザセキュリティ」発刊記念 著者＆レビュアー対談(後編)](https://techblog.securesky-tech.com/entry/2021/02/15/)

## 類書との比較

下記のようなツイートをした手前、実際読んでみてどうだったか書いてみました。あくまで私の感覚的な比較であり、またどっちが良いという話でもないです (情報の鮮度という意味では間違いなく本書を読んだ方が良いです)

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="ja" dir="ltr">ウェブブラウザセキュリティといえば &quot;The Tangled Web&quot; (翻訳版：めんどうくさい Web セキュリティ) という素晴らしい本があるけど、原著が 2011 年出版で内容がかなり古くなっているので、それに取って代わるような本だと良いなぁと期待しつつ読み進める・・・ <a href="https://twitter.com/hashtag/nhbk?src=hash&amp;ref_src=twsrc%5Etfw">#nhbk</a> <a href="https://t.co/Dk3xI3whOo">pic.twitter.com/Dk3xI3whOo</a></p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1353244517839773698?ref_src=twsrc%5Etfw">January 24, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- 『めんどうくさい Web セキュリティ』はブラウザの提供する各機能について基礎的なところから細かい挙動やセキュリティ上の注意点までを順番に解説しており、いわゆるボトムアップ的なアプローチを取っている。機能毎に紹介していくためとっつきやすい反面、ブラウザ全体で何をどう守るべきなのか俯瞰的な視点が得にくいかもしれない。Flash やブラウザプラグインなど扱う話題が古いながらも多岐に渡っている。
- 『Web ブラウザセキュリティ』は最初にウェブシステムの分類やブラウザアーキテクチャといったシステムの全体像を整理し、どこにある何を守るべきなのか、守るために必要な機能は何なのかをトップダウン的に紹介している。具体的な攻撃手法を研究論文を大量に引用しながら紹介し、それらに対抗すべく導入されたブラウザセキュリティ機構を最新のものまで詳しく説明している。一般のウェブアプリ開発者にももちろん有用だが、それ以上にウェブセキュリティ研究者の「知の高速道路」として大いに役立ちそう[^highway]。

[^highway]: 序文とあとがきにて筆者自身が Web ブラウザセキュリティにおける「知の高速道路」を目指していると述べている。

# 読書メモ

[Twitter でのメモ書き](https://twitter.com/nhiroki_/status/1353010223334584321)

## 第 1 章　Web と Web セキュリティ

- Web の構成要素 (URL, HTTP, HTML)、Web システムの分類 (クライアントサイド・サーバサイド) とそれぞれの構成要素・接続、クライアントにおけるセキュリティの関心事 (リソースの論理・プロセス隔離、Cookie、読み込むリソースの信頼性)、など。
- これとても気にしてます

> プロセスレベルの隔離について考えるときは、Web ブラウザベンダは実装にあたってパフォーマンスも気にしなくてはならないのだ、ということを頭の片隅に置いておきましょう (中略) セキュリティ向上とパフォーマンスに関する定量的評価に一定のトレードオフがある ― p.19

## 第 2 章　Origin を境界とした基本的な機構

- 制限すべき操作の種類、URL からどのように境界を決めるか、Same-Origin Policy、CORS、CORS 以外の SOP 緩和方法、XSS、CSP、Trusted Types など。
- これらセキュリティ機能が筋道を立てて紹介されていて分かりやすかった。例えば、SOP は強力だけど XSS は防ぎきれないので CSP があり、さらに CSP でも防げないものがあってそれは・・・みたいな。
- Mozilla が CSP の論文を出してたの知らなかった。他にも各機能の下敷きになった論文が紹介されていて勉強になった。
- CSP やその他ポリシー周りはディレクティブが多すぎてテスト (web platform tests) を書くのも走らせるのもめちゃ大変。以前もツイートしたけど、同僚とインターン氏が generator を書いてかなり良くなった。

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="ja" dir="ltr">[蛇足の蛇足] Referrer Policy はポリシーの種類やコンテキスト間での継承ルールが多岐に渡ることから generator を作って WPT のテストケースを自動生成してるんですが、組み合わせが多すぎてテストを走らせるのが困難だったりします。最近、同僚とインターン氏が頑張ってくれてだいぶ良くなりました</p>&mdash; nhiroki (@nhiroki_) <a href="https://twitter.com/nhiroki_/status/1266042945368514560?ref_src=twsrc%5Etfw">May 28, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- CSP や sandbox に関連して [Feature Policy (Permission Policy)](https://w3c.github.io/webappsec-permissions-policy/) の話があっても良いかもと少し思った。
- 実行コンテキスト間の policy inheritance のルールが feature 毎に違っててカオスという話もある。例えば dedicated worker は親の CSP を継承するけど、親の referrer policy は継承しない、みたいな。この辺は [W3C TAG で議論](https://github.com/w3ctag/design-principles/issues/111) になってて、HTML 仕様で [Policy Container というものにまとめるのはどうかという提案](https://github.com/whatwg/html/issues/4926)が出てたりする。
- アンカータグが [ping 属性](https://html.spec.whatwg.org/multipage/links.html#ping)を持ってるの知らなかった・・・

## 第 3 章　Web ブラウザのプロセス分離によるセキュリティ

- シングルプロセスアーキテクチャの問題点、プロセス分割モデル (Process-per-BrowsingInstance, Process-per-SiteInstance, Process-per-Site)、memory disclosure attacker の仕組み、Origin Isolation (CORS, CORP, COOP/COEP)、FetchMetadata など。
- プロセス分離に関して恐らく日本語で最も詳しく整理された資料だと思います。過去の研究実装から最新の仕様までしっかり言及されていて、ここまで調べてまとめあげるのは大変だったと思います。欲を言うと、プロセス分離やメモリ共有の仕様レベルのコンセプトである [Agent や AgentCluster](https://nhiroki.jp/2017/12/10/javascript-parallel-processing#4-%E5%85%B1%E6%9C%89%E3%83%A1%E3%83%A2%E3%83%AA%E3%83%A2%E3%83%87%E3%83%AB) の話も書いてあると良かった。ちょうど最近 [Origin-Agent-Cluster ヘッダ](https://web.dev/origin-agent-cluster/)の話も出ましたね (記事にある通り厳密にはセキュリティのための機能ではないけれども)
- 本書はセキュリティの本なので Origin Isolation に焦点を当ててアーキテクチャを解説してるんだけど、これを読んで Performance Isolation に焦点を当てたアーキテクチャのまとめ記事があると面白そうだなぁ、と思った。
- 図 3.4 のマルチプロセスアーキテクチャの図ですが、2007 年の論文の図からの引用なので少し現状と違っていて、今の Chromium ではネットワーク通信もブラウザカーネルとは別プロセス (Network Service) になっているし、ストレージも別プロセス (Storage Service) へ移行している最中です。

## 第 4 章　Cookie に関連した機構

- Cookie 導入の動機、Cookie の持つ属性値、Cookie の性質とその問題 (CSRF 脆弱性・プライバシー・SOP とのセキュリティ境界の相違)、Cookie に関連した最新の動き、など。
- [_Host- や __Secure- みたいな特別な prefix](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-03#section-4.1.3) があるの知らなかった

## 第 5 章　リソースの完全性と機密性に関連する機構

- リソースの完全性と機密性の意味とそれらが損なわれたときに起こりうる問題、HTTPS と HSTS、Mixed Content、SubResoucre Integrity (SRI)、Secure Context、など。
- [TOFU (Trust On First Use)](https://en.wikipedia.org/wiki/Trust_on_first_use) という言葉知らなかった。
- SRI が opaque filtered response には使えないという話、言われてみると当然なんだけど、こういうのって意外と気づかない。似た話で App Cache や Cache Storage にそれを突っ込んで [Quota API でサイズ変化を見るみたいな攻撃](https://chromestatus.com/feature/5400170344742912)もあった (第 6 章で言及されてた)

## 第 6 章　攻撃手法の発展

- CSP をバイパスする攻撃手法、scripting を使わない攻撃手法、サイドチャネル攻撃、など。
- 攻撃手法が多岐に渡りすぎてて、セキュリティの専門家がいないとしっかり対応するのは難しそう。

# 注釈