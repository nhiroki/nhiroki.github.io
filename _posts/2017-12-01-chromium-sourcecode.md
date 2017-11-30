---
layout: post
title: "Chromium のソースコードの歩き方"
date: 2017-12-01 00:00:00 +09:00
tags: codereading web
image: /images/chromium-sourcecode-codesearch.png
---

これは [Chromium Browser アドベントカレンダー](https://qiita.com/advent-calendar/2017/chromium)の一日目の記事です。初日ということで、本記事では Chromium のソースコードを読む上で役に立つであろう、プロジェクトのディレクトリ構成やファイル構成を紹介します。

# Chromium とは？

[Chromium](https://www.chromium.org/) とはオープンソースのウェブブラウザです。Chrome や Opera などは Chromium をベースにして作られています。主に C++ 言語で書かれています。

# ソースコードへのアクセス

Chromium のソースコードは Git で取得することができます (取得方法は[公式ドキュメント](https://www.chromium.org/developers/how-tos/get-the-code)を参照)。規模がとても大きいため、チェックアウトするだけでもかなりの時間がかかります。しかし、幸いにも Chromium には強力な[コードサーチのページ](https://cs.chromium.org)があるので、まずはこれを使って読むことをおすすめします。

コードサーチではクラスの定義や関数の呼び出し元、変数の参照元などを辿ることができます。また、Git のコミットログや blame を見たり、markdown ファイルを整形表示する機能などがあります。ちなみにコードサーチはコミットされた変更がわりとすぐに反映されますが、識別子の参照解析に時間がかかるため、時々参照元へのリンクが無効化されていることがあります。

![コードサーチ](/images/chromium-sourcecode-codesearch.png)

# ディレクトリ構成

Chromium のソースコードへのアクセス方法が分かったところで、次にプロジェクトのディレクトリ構成について説明します。Chromium のディレクトリはコンポーネントやプロセスの構成を元に構造化されています。この構造化のルールを知っておくと、気になる処理がどこで行われているのか探すのが早くなります。以下は Chromium の[トップディレクトリ src/](https://cs.chromium.org/chromium/src/) にある主要ディレクトリです。

```
src/
  - base/
  - chrome/
    - browser/
    - common/
    - renderer/
  - content/
    - browser/
    - common/
    - renderer/
  - third_party/
    - WebKit/
  - v8/
```

**コンポーネントによるディレクトリ分け (chrome / content / third_party / base)**

まずはコンポーネントによるディレクトリ分けです。ウェブページの表示枠（ページフレーム）を表示する最低限のコンポーネントは Content モジュールにまとめられています。Content モジュールは content/ 以下に配置されます。よく Blink/Chromium という表記方法をすることがありますが、開発者はこの content/ ディレクトリを指して Chromium と呼ぶことが多いです。

ページフレームをラップし、さらに Chrome 独自の機能 (ブラウザ拡張とかプロファイルの同期機能など) を実装しているのが Chrome モジュールです。Chrome モジュールは chrome/ ディレクトリにまとめられています。ブラウザ拡張の API 実装が見たい場合はこのディレクトリ以下を探すことになります。

Chromium が依存しているライブラリは third_party/ 以下に配置されます。このディレクトリには[動画処理ライブラリ FFmpeg](https://ja.wikipedia.org/wiki/FFmpeg) や[グラフィックスライブラリ Skia](https://ja.wikipedia.org/wiki/Skia)、[シリアライザの Protocol Buffers](https://ja.wikipedia.org/wiki/Protocol_Buffers) など、様々なライブラリが含まれています。その中でもとりわけ重要なのは Blink モジュールでしょう。Blink は Chromium のレンダリングエンジンです。レンダリングエンジンという名前ですが、実際にはレンダリング以外にも JavaScript を実行したり、JavaScript と C++ のバインディングを提供したりします。Blink は Chromium プロジェクトの一部で現在は同じリポジトリ内で開発されていますが、WebKit からフォークされた ([公式ブログ記事](https://blog.chromium.org/2013/04/blink-rendering-engine-for-chromium.html)) という歴史的な経緯により third_party/ ディレクトリに配置されていて、さらにディレクトリ名も WebKit のままになっています[^the-great-blink-mv]。

[^blink-fork]: [Blink: A rendering engine for the Chromium project - Chromium Blog](https://blog.chromium.org/2013/04/blink-rendering-engine-for-chromium.html)
[^the-great-blink-mv]: WebKit/ ディレクトリを blink/ にリネームする提案がされています / [The Great Blink mv](https://docs.google.com/document/d/1l3aPv1Wx__SpRkdOhvJz8ciEGigNT3wFKv78XiuW0Tw/edit?pli=1)

v8/ ディレクトリは JavaScript の実行エンジン V8 が配置されます。V8 は Blink のコアコンポーネントですが、開発は別のリポジトリで行われていて、サブモジュールとしてインポートされます。一見すると third_party/ に置かれるべきものですが、必須コンポーネントであるという理由からトップレベルディレクトリに置かれています。

base/ は chrome/ や content/ で使われる共通ライブラリで、OS の提供するシステムコールなどを抽象化した API を提供します。Chromium は Windows, Mac, Linux, Android, iOS など様々なプラットフォームをサポートしていますが、この base/ ライブラリのおかげで開発者は下位プラットフォームの差をあまり気にせずプログラミングすることができるようになっています。

**Chromium 内のプロセスによるディレクトリ分け (browser / renderer)**

次にプロセスによるディレクトリ分けです。Chromium では[マルチプロセスアーキテクチャ](https://www.chromium.org/developers/design-documents/multi-process-architecture)が採用されています。まず各タブに相当する Renderer プロセスというものがいます。Blink によるレンダリング処理や JavaScript の実行などは Renderer プロセスの中で行われます。一方、ネットワークやディスクへのアクセスといった特権を必要とする処理はメインプロセスである Browser プロセスで行われます。Browser プロセスと Renderer プロセスは 1:n の関係になっています。この他にも GPU プロセスや Plugin プロセスなどがあります。

[^multi-process-arch]: [Multi-process Architecture - The Chromium Projects](https://www.chromium.org/developers/design-documents/multi-process-architecture)

Browser プロセスと Renderer プロセスはそれぞれ browser/ ディレクトリと renderer/ ディレクトリに対応しています。プロセスモデルに従い、これらディレクトリはソースコードレベルでお互いに依存することはできず、IPC (Inter-Process Communication) 機構によってやりとりを行います。common/ は browser/ と renderer/ で共通して使われるヘッダーファイルなどが格納されていて、例えば IPC でやりとりされる enum や構造体、共通して呼び出されるヘルパー関数などが定義されています。

![プロセスモデル](/images/chromium-sourcecode-process-model.png)

**機能によるディレクトリ分け**

browser や renderer といったプロセスごとのディレクトリの下は、さらに機能毎のディレクトリに分かれています。大多数の機能は renderer プロセスと browser プロセスにまたがって実装されているため、両方のディレクトリに同名のディレクトリがあります。例えば [Service Worker](https://developers.google.com/web/fundamentals/primers/service-workers/?hl=ja) は両プロセスにまたがる典型的な機能ですが、content/renderer/service_worker/ と content/browser/service_worker/ の両方にディレクトリがあります。

**Blink 内のディレクトリ分け (core / modules)**

Blink はシングルプロセスで動くため、browser / renderer といったディレクトリはありません。しかし、別の観点によるディレクトリ分けが行われています。

```
WebKit/
  - LayoutTests/
  - Source/
    - bindings/
    - core/
    - modules/
    - platform/
  - public/
```

- bindings/ は JavaScript と C++ をつなげるバインディングコードが置かれています。
- core/ はウェブブラウザとしての中核的な機能 (DOM や CSS など) が実装されていて、modules/ はより応用的な機能が実装されています。modules/ は core/ に依存することができますが、その逆は禁止されています。core/ と modules/ の使い分け方はかなり曖昧で、Chromium 開発者の間でもたびたび議論の的になります。core/ と modules/ の下はさらに機能毎のディレクトリに分かれており、前述の Service Worker の場合は応用的な機能のため、modules/serviceworkers/ にディレクトリがあります。
- platform/ は core/ や modules/ で共通して使われる基本的な機能やプラットフォームを抽象化するライブラリを実装しています。特に platform/wtf/ は Blink のパフォーマンス特性に合わせた独自の String 実装やコンテナ実装を含んでいる重要なディレクトリです。platform/ 内のいくつかの機能は Chromium の base/ ライブラリと機能が被っており、現在 base/ ライブラリを直接使えるように移行作業が行われています。
- public/ は Content モジュールにエクスポートされる関数やクラスのヘッダーファイルが格納されています。Content モジュールはこのインタフェースを通して Blink の機能にアクセスします。
- LayoutTests/ には HTML 仕様との互換性を確認する end-to-end のテストが置かれています。これらテストは HTML や JavaScript で書かれており、API のサンプルコードとしても有用です。また、ブラウザ間の互換性を確認する [Web Platform Tests (WPT)](https://github.com/w3c/web-platform-tests) もこのディレクトリ以下 (LayoutTests/external/wpt/) にインポートされています。

**まとめ**

以上がコンポーネントやプロセスモデルによるディレクトリの分け方でした。これらを踏まえて主要なディレクトリの構成を眺めてみると、見事にコンポーネント/プロセス/機能という階層構造でディレクトリが切り分けられていることが分かると思います。

```
src/
  - chrome/
    - browser/
    - common/
    - renderer/
  - content/
    - browser/
    - common/
    - renderer/
  - third_party/
    - WebKit/
  - v8/
```

# 各ディレクトリ内のファイル構成

各ディレクトリには大まかに以下のようなファイルが含まれています。Blink と Chromium は元々別プロジェクトだったため、ファイルの命名規約などが微妙に異なっています[^the-great-blink-mv2]。

[^the-great-blink-mv2]: Blink と Chromium で命名規則を統一する作業が行われています / [The Great Blink mv](https://docs.google.com/document/d/1l3aPv1Wx__SpRkdOhvJz8ciEGigNT3wFKv78XiuW0Tw/edit?pli=1)

```
Chromium 内 (content/, chrome/ など)
- foo.{cc,h} : 実装
- foo_unittest.cc : ユニットテスト
- foo_browsertest.cc : インテグレーションテスト (プロセスをまたぐテスト)
- foo_messages.h : Legacy Chrome IPC メッセージの定義ファイル

Blink 内 (third_party/WebKit/Source/)
- Bar.{cpp,h} : 実装
- BarTest.cpp : ユニットテスト
- Bar.idl : JavaScript API のインタフェースの定義ファイル（WebIDL）

Chromium / Blink 共通
- BUILD.gn : ソースファイルの一覧やビルド方法を記述したファイル（Makefile みたいなもの）
- DEPS : ソースファイル間の依存関係の許可・不許可のルールを記述したファイル
- OWNERS : コードのオーナーシップが書かれたファイル
- foo.mojom : Mojo のインターフェースの定義ファイル
```

**JavaScript API のインタフェースの定義ファイル（WebIDL）**

ソースコードを読み進める上で特に重要なファイルは WebIDL（*.idl）ファイルです。これは JavaScript API のインタフェースの定義を行うファイルで、これを元にコンパイル時にコードジェネレーターが JavaScript と C++ のバインディングコードを生成します。特定の JavaScript API の実装を読み進める時は、まずこのファイルを探すと良いでしょう。

WebIDL は各 API の仕様で定義されています。例えば、Web Storage API の場合は、[Storage という WebIDL がその仕様に定義](https://html.spec.whatwg.org/multipage/webstorage.html#the-storage-interface)されていて、Blink 内ではこれを [WebKit/Source/modules/storage/Storage.idl](https://chromium.googlesource.com/chromium/src/+/45146fc8a1ca4c58272b2c7a5d945c35db35f14d/third_party/WebKit/Source/modules/storage/Storage.idl) というファイルでほぼそのまま定義しています。インタフェースの実装は同一ディレクトリ内の Storage.{cpp,h} で行われています。

```
// https://html.spec.whatwg.org/multipage/webstorage.html#the-storage-interface
interface Storage {
    [RaisesException=Getter] readonly attribute unsigned long length;
    [RaisesException] DOMString? key(unsigned long index);
    [LogActivity, RaisesException] getter DOMString? getItem(DOMString key);
    [LogActivity, RaisesException] setter void setItem(DOMString key, DOMString value);
    [LogActivity, RaisesException] deleter void removeItem(DOMString key);
    [LogActivity, RaisesException] void clear();
};
```

**OWNERS**

[OWNERS ファイル](https://chromium.googlesource.com/chromium/src/+/master/docs/code_reviews.md#owners-files)はコードのオーナーを記述したファイルです。例えば [Web Worker](https://developer.mozilla.org/ja/docs/Web/API/Web_Workers_API/Using_web_workers) の実装がある WebKit/Source/core/workers/ の OWNERS ファイルには次のように書かれています。

```
nhiroki@chromium.org

# TEAM: worker-dev@chromium.org
# COMPONENT: Blink>Workers
```

nhiroki がオーナーで、workers/ ディレクトリ内のソースコードの変更には nhiroki もしくは上位ディレクトリのオーナーのコードレビューが必要になります。コードに詳しいオーナーのレビューを強制することで、バグの混入を防いだり、コードのクオリティが維持されることが期待できます。TEAM はオーナーシップを持つチーム名、COMPONENT はバグトラッカーでバグを登録する際に付与すべきコンポーネント名を示しています。

**DEPS**

DEPS はソースコードレベルでの依存関係の許可・不許可のルールを定義するファイルです。例えば browser/ と renderer/ はお互い依存することはできませんが、そのルールがこのファイルに記述されています。ルールのマッチングはヘッダーファイルのインクルードに対して行われます。このルールに違反する場合、パッチをコードレビューに出したりコミットするときに警告が出ます。

**IPC の定義ファイル**

IPC (Inter-Process Communication) の定義ファイルは実装と同じディレクトリに置かれています。現在 Chromium では IPC の機構を新しいものに置き換えていて、従来のものを Legacy Chrome IPC、新しいものを [Mojo](https://chromium.googlesource.com/chromium/src/+/master/mojo/README.md#system-overview) と呼んでいます。Legacy Chrome IPC はメッセージパッシング型の IPC で、メッセージタイプとメッセージが格納するデータを *_messages.h というファイルに記述します。一方、Mojo は RPC (Remote Procedure Call) 型の IPC 機構で、受け付ける RPC の定義を *.mojom というファイルに記述します[^servicification]。IPC の定義ファイルを見ることで、Browser プロセスと Renderer プロセスがどのようなやり取りをしているのか知ることができます。

[^servicification]: ブラウザの各機能を「サービス」という単位に分割し、各サービスが提供する機能を Mojo で定義する Servicification というプロジェクトが進んでいます / [Servicification - The Chromium Projects](https://www.chromium.org/servicification)

# まとめ

本記事では Chromium プロジェクトのディレクトリ構成やファイル構成を紹介しました。この記事がウェブブラウザのソースコードを読み解くきっかけになれば幸いです。

# 注釈