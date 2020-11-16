---
layout: post
title: "Chromium の HttpStreamParser によるヘッダ処理"
date: 2020-11-16 00:00:00 +09:00
tags: web
image: /images/profile.png
---

Chromium の net スタックにある HttpStreamParser というクラスの挙動についてのメモです。Chromium のネットワーキング周りの開発に関わっていない人には全く役に立たない知識です・・・

Chromium の実装に関するメモをなるべく外出ししていきたいと思っていて、本記事はその一環です。自分用メモなので細かいことは説明しません。ブログ記事が最近読書メモばかりなので、こういうニッチなメモを小出ししていくことでソフトウェア関係の記事の比重を高めていきたい・・・

# はじめに

最近 Chromium の net スタック周りのコード、特に HTTP の header parsing 周りのコードを眺めている。それで今回は HTTP/1.1 で使われている HttpStreamParser というクラスの挙動を調べた。net スタックのコードは Chromium リポジトリの net/ にあり、HttpStreamParser の実装は net/http/http_stream_parser.{cc,h} にある。読んだのは [master@{#827690} (Nov 16, 2020)](https://chromium.googlesource.com/chromium/src/+/dad2da69fd81138bc907fb5ed115598de5759730) 時点での HttpStreamParser ([cc](https://chromium.googlesource.com/chromium/src/+/dad2da69fd81138bc907fb5ed115598de5759730/net/http/http_stream_parser.cc), [h](https://chromium.googlesource.com/chromium/src/+/dad2da69fd81138bc907fb5ed115598de5759730/net/http/http_stream_parser.h)) です。

# 実装と挙動

HttpStreamParser の中でも HandleReadHeaderResult() という関数が response header の parse 処理を駆動する。parse 結果 (HTTP ステータスコードやヘッダーライン) は HttpResponseHeaders クラスに詰め込まれ、parse イベントに関するタイムスタンプを記録する。このタイムスタンプは [Resource Timing](https://w3c.github.io/resource-timing/) と呼ばれる JavaScript API へと expose される。

```c++
int HttpStreamParser::HandleReadHeaderResult(int result) {
  // ... snip ...
  int end_of_header_offset = FindAndParseResponseHeaders(result);
  // ... snip ...
}
```

通常は 1 request に対して 1 回しかこの関数は呼ばれないが、以下の場合において複数回呼ばれうる。

**(1) Informatinal response (1xx) が送信された場合**

HTTP リクエストと HTTP レスポンスは常に 1:1 対応すると思っている人もいるかもしれないがそんなことはなく、サーバは一つのリクエストに対して複数のレスポンスを返すことがある。多分一番分かりやすい例だと HTTP/2 の Server Push だけど、今回は Server Push ではなく [informational response](https://tools.ietf.org/html/rfc7231#section-6.2) と呼ばれる HTTP ステータスコード 1xx 系のレスポンスが問題となる (そもそも今回のコードは HTTP/1.1 向けなので HTTP/2 Server Push は関係しない)。

Informational responses は実際の response (non-informational response と呼ぶことにする) に先立って送られてくる。例えば、1xx --> 1xx --> ... --> 1xx --> 200 みたくなる。Non-informational response は最後に一回だけ送られてくる。

Chromium の実装では 1 request に対するこれら informational responses と non-informational response は同じ HttpStreamParser インスタンスで処理され、その分だけ HandleReadHeaderResult() が呼ばれる。つまり N 個の informational responses と 1 個の informational response が送られてきた場合、HandleReadHeaderResult() の呼び出し回数は (N + 1) 回になる。

現在処理している response が informational かどうかはステータスコードを見れば分かる。

```c++
if (response_->headers->response_code() / 100 == 1) {
  // This is an informational response!
}
```

**(2) Header が分割されて読み込まれた場合**

一つの header が複数のフラグメントに分割されて読み込まれる場合があり、このときはフラグメント毎に HandleReadHeaderResult() が呼ばれる。二回目以降は offset に non-zero が指定されるのでそれで分かる。

```c++
// Record our best estimate of the 'response time' as the time when we read
// the first bytes of the response headers.
if (read_buf_->offset() == 0)
  response_->response_time = base::Time::Now();
```

N 個の non-informational response headers と 1 個の informational response header で構成されたレスポンスで、さらにそれぞれが Q 個と R 個に分割されて送信されてきた場合、HandleReadHeaderResult() の呼び出し回数は (N * Q + R) 回になる。

全フラグメントが読み込まれるまで header parsing は行われないのでそれまでステータスコードは取れない。

## 問題

これまで HandleReadHeaderResult() が複数回呼ばれる場合について見てきた。これらの呼び出しはすべて同じ HttpStreamParser インスタンスに対して行われるので、HttpStreamParser に状態を持たせる場合は注意が必要になる。例えば、HandleReadHeaderResult() に response 毎に一回きりの処理を足す場合は、前述の複数回呼ばれる場合を考慮しないとバグる (そして私がこのバグを踏んだのでこの記事を書いている)。分かりやすい例だと、前述した parse タイミングに関するタイムスタンプを誤って上書きしてしまったりする。

また、HttpStreamParser は HttpResponseInfo* をメンバ変数に持つがこれもちょっと分かりにくい。このポインタの値は HandleReadHeaderResult() が何度呼ばれても変わらないが、参照先のオブジェクトの中身は現在処理している response に応じて書き換わるので注意が必要。

# まとめ

HttpStreamReader の header parsing 周りは複雑な状態を持つので注意が必要。特に HandleReadHeaderResult() が複数回呼ばれることがあるので、一回きりの処理を書く場合は気をつける。