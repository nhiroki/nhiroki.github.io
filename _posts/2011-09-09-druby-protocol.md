---
layout: post
title: "dRuby に独自プロトコルを追加する方法"
date: 2011-09-09 10:54:00 +09:00
tags: web
image: /images/profile.png
---

Ruby には分散オブジェクトを扱うライブラリである dRuby が標準で含まれている。dRuby は標準ではトランスポート層に TCP ソケットを用いるが、ユーザが自由に新しいプロトコルを追加することもできる。これを調べてみた。

なお、プロトコルの追加方法については Ruby ソースコード内の lib/drb/drb.rb に詳細なコメントが記述されているので、そちらを参照してください。dRuby の基本的な使い方は適当にググッてください。

# プロトコルの追加方法

`DRbProtocol` モジュールに対して、独自定義したプロトコルのクラスを追加する。プロトコルの追加には、`DRb::DRbProtocol#add_protocol` メソッドを用いる。

```ruby
class OrigProtocol
# ...
end
DRb::DRbProtocol.add_protocol OrigProtocol
```

# 独自プロトコルの使い方

通信に使用するプロトコルは URI のスキーマ部分で指定する。例えば、`OrigProtocol` 内で `orig` というスキーマを使用するように実装した場合、次のように指定する。

```ruby
drb = DRb.start_service 'orig://host:port', obj
```

# プロトコルの実装

プロトコルを追加するには Protocol, Connection, Listener を表すクラスが必要となり、それぞれのクラスには dRuby が要求するメソッドを定義しておく必要がある。

(実際にはそれぞれ個別のクラスに分ける必要はなく、一つのクラスにまとめてしまっても大丈夫である。説明の便宜上、三つに分けている。)

話は逸れるが、あるクラスにあるメソッド M が実装されていることを要求する場合、Java などではインタフェース I を定義してそれを実装することになるだろう。一方、Ruby ではインタフェース I を宣言的に実装しなくても、メソッド M が実装されているだけでそのインタフェース I が実装されていると見なす。つまり、メソッド M を持つクラスは全てインタフェース I を実装していることになる。このような考え方を**ダックタイピング**と呼ぶ。

## (1) Protocol クラス

Protocol クラスのインスタンスを `add_protocol` メソッドに与えることで、そのプロトコルを追加することができる。Protocol クラスに必要なメソッドは次の通りである。

- `open(uri, config)` : uri が示すサーバに対してコネクションを張る。コネクションを表すクラス (Connection) のインスタンスを返す。</dd>
- `open_server(uri, config)` : uri に対して listen する。リスナーを表すクラス (Listener) のインスタンスを返す。
- `uri_option(uri, config)` : uri が持つパラメータ部分 ("?param=val") を解釈し、`[uri, option]` の組で返す。

## (2) Connection クラス

Connection クラスは Protocol クラスのコネクションを表す。`send_request`, `recv_reply` がクライアント側で使うメソッド、`recv_request`, `send_reply` がサーバ側で使うメソッドである。Connection クラスに必要なメソッドは次の通りである。

- `send_request(ref, msg_id, arg, b)` : オブジェクト ref に対してメッセージを送る。
- `recv_reply` : サーバから受信したリプライを `[success-boolean, reply-value]` のペアとして返す。
- `recv_request` : クライアントからリクエストを受信し、`[object, message, args, block]` の組で返す。
- `send_reply` : リプライをクライアントへ送信する。
- `alive?` : コネクションが生きているか返す。
- `close` : コネクションを閉じる。

## (3) Listener クラス

`Listener` クラスはサーバ側でコネクションの受け入れを行うためのクラスである。`Listener` クラスに必要なメソッドは次の通りである。

- `accept` : 新しいコネクションを受け入れる。コネクションを表すクラス (Connection) のインスタンスを返す。
- `close` : コネクションを閉じる
- `uri` : サーバの URI を取得する。

プロトコルの実装に関しては以上である。実装サンプルとして、lib/drb/drb.rb に TCP ソケットを用いた標準プロトコル (DRbTCPSocket)、lib/drb/unix.rb に Unix ソケットを用いたプロトコル (DRbUNIXSocket) の実装があるので、そちらを参照すると良い。

# まとめ

分散オブジェクト機構 dRuby に独自のプロトコルを追加する方法を調べた。dRuby ではモジューラブルにプロトコルを追加する仕組みがあり、必要なメソッドを実装したクラスを用意するだけで簡単に追加できることが分かった。
