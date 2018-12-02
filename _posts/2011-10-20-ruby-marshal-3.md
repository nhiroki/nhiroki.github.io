---
layout: post
title: "Ruby の Marshal モジュールを読む (3) | w_object()"
date: 2011-10-20 11:13:00 +09:00
tags: codereading
image: /images/profile.png
---

引き続き Ruby の Marshal モジュールを読んでいきます。

- [Ruby の Marshal モジュールを読む (1)](/2011/10/16/ruby-marshal-1)
- [Ruby の Marshal モジュールを読む (2) \| marshal_dump()](/2011/10/17/ruby-marshal-2)

# バージョン情報の dump と w_byte()

今回は dump 処理を行う `w_` 系関数について見ていきます。前回確認した `marshal_dump()` には、バージョン情報を書き込む

```c
  w_byte(MARSHAL_MAJOR, arg);
  w_byte(MARSHAL_MINOR, arg);
```

というコードがありました。`w_byte()` は第一引数で渡した文字 1byte を dump する関数です。この `w_byte()` について見ていきます。

```c
static void
w_byte(char c, struct dump_arg *arg)
{
  w_nbyte(&c, 1, arg);
}
```

`w_byte()` は `w_nbyte()` に処理を委譲しています。`w_nbyte()` は文字通り指定した byte 数分だけデータを dump する関数です。

```c
static void
w_nbyte(const char *s, long n, struct dump_arg *arg)
{
  VALUE buf = arg->str;
  rb_str_buf_cat(buf, s, n);
  RBASIC(buf)->flags |= arg->infection;
  if (arg->dest && RSTRING_LEN(buf) >= BUFSIZ) {
    rb_io_write(arg->dest, buf);
    rb_str_resize(buf, 0);
  }
}
```

`w_nbyte()` は上の通りです。dump データ `buf` に対して、`rb_str_buf_cat()` によって文字列 `s` を n byte 分追記します。6 行目では `infection` の論理和を取ることでフラグの設定をしているようですが、具体的にどんなフラグを立てているのかが分からないので、ここでは飛ばします。7 行目から 10 行目は出力先ポートが指定されていて、かつバッファサイズよりも dump データ長が大きい場合は、そのポートへと結果を出力してバッファを切り詰める作業を行います。

このように、`w_nbyte()` が dump データを実際に書き込む処理を行なっていることが分かりました。

さて、先ほどのバージョン情報を書き込む部分に話を戻します。

```c
  w_byte(MARSHAL_MAJOR, arg);
  w_byte(MARSHAL_MINOR, arg);
```

`MARSHAL_MAJOR` と `MARSHAL_MINOR` は次のように定義されています。

```c
#define MARSHAL_MAJOR   4
#define MARSHAL_MINOR   8
```

よって、dump データに '4' と '8' を書き出す処理を行なっていることになります。ここが初めて dump データを書き出す場所なので、Marshal フォーマットでは dump データの先頭 2byte はバージョン情報であることが分かりました。

```c
static VALUE
marshal_load(int argc, VALUE *argv)
{
  // ～省略～
  major = r_byte(arg);
  minor = r_byte(arg);
  if (major != MARSHAL_MAJOR || minor > MARSHAL_MINOR) {
    clear_load_arg(arg);
    rb_raise(rb_eTypeError, "incompatible marshal file format (can't be read)\n\
        \tformat version %d.%d required; %d.%d given",
        MARSHAL_MAJOR, MARSHAL_MINOR, major, minor);
  }
  if (RTEST(ruby_verbose) && minor != MARSHAL_MINOR) {
    rb_warn("incompatible marshal file format (can be read)\n\
        \tformat version %d.%d required; %d.%d given",
        MARSHAL_MAJOR, MARSHAL_MINOR, major, minor);
  }
  // ～省略～
}
```

言うまでもないことですが `marshal_load()` では先頭 2byte でバージョンチェックを行なっていて、互換性のないバージョンの場合は例外もしくは警告を発します。

念のため本当に先頭 2byte がバージョン情報なのか irb を使って確認してみます。

```bash
$ irb
ruby :001 > data = Marshal.dump("a")
 => "\x04\bI\"\x06a\x06:\x06ET"
ruby :002 > data[0].ord
 => 4
ruby :003 > data[1].ord
 => 8
```

ちゃんとバージョン情報になっていますね。ちなみに String#ord
は文字列の最初の文字のコードポイントを返すメソッドです。

# Marshal フォーマット

Marshal フォーマットの話が出てきたので、ここらでまとめておきます。

Marshal フォーマットとは dump データの何 byte 目がどういう意味を表すのか定義したものです。クラス毎に固有のフォーマットが定義されています。クラスが違えばオブジェクト構成も変わるので、それに伴ってバイナリ列が変わるのは当然ですね。Marshal フォーマットは次のページにまとまっています。

[Marshalフォーマット](http://www.ruby-lang.org/ja/old-man/html/Marshal_A5D5A5A9A1BCA5DEA5C3A5C8.html "Marshalフォーマット")

URL が old-man になっていますが、私が見ている Marshal Format version 4.8 もサポートしているようなので問題ないでしょう。むしろ最新の文書はどこにあるんだろう・・・？以後、たびたび参照するのでお気に入りに入れておくといいかもしれません。

オブジェクトの dump データは大雑把に言うと「データの種類を表す文字」、「データ固有の管理データなど」、「データ部」という並びになっています。詳細は追々見ていきます。ここでは **<span style="color: red;">各オブジェクトの dump データの先頭はそのデータの種類を表す文字</span>** だということを覚えておいてください。

# w_object()

いよいよ、オブジェクトの dump 処理を見ていきます。`marshal_dump()` の

```c
w_object(obj, arg, limit);
```

という部分がその処理に当たります。

```c
static void
w_object(VALUE obj, struct dump_arg *arg, int limit)
{
  struct dump_call_arg c_arg;
  st_table *ivtbl = 0;
  st_data_t num;
  int hasiv = 0;
#define has_ivars(obj, ivtbl) ((ivtbl = rb_generic_ivar_table(obj)) != 0 || \
    (!SPECIAL_CONST_P(obj) && !ENCODING_IS_ASCII8BIT(obj)))

  if (limit == 0) {
    rb_raise(rb_eArgError, "exceed depth limit");
  }

  limit--;
  c_arg.limit = limit;
  c_arg.arg = arg;
```

`w_object()` は長いので分割して見ていきます。まず初期化処理が続きます。`struct dump_call_arg` は名前の通り、dump 関数コール時の引数として渡す構造体のようです。次のように定義されています。

```c
struct dump_call_arg {
  VALUE obj;             // dump 対象のオブジェクト
  struct dump_arg *arg;  // dump データを格納する構造体 (前回記事参照)
  int limit;             // 再帰的に dump を行う深さの制限値
};
```

簡単ですね。次に `st_table` ですが、これは Ruby 処理系内で用いられているテーブルの実装です。`*ivtbl` は Instance Variable Table の略で、インスタンス変数を管理するテーブルのようです。また、`st_data_t num` は `ivtbl` に関する何らかの数値を、`hasiv` は `obj` がインスタンス変数を持っているか管理するフラグのようですね。

8 行目の define 文は、インスタンス変数を持つかどうかを判定するマクロを定義しています。持つ場合は先程の `ivtbl` にテーブルへのポインタをセットします。

11 行目から 13 行目は `limit` を越えた場合に marshal を打ち切る処理です。

```c
  if (st_lookup(arg->data, obj, &num)) {
    w_byte(TYPE_LINK, arg);
    w_long((long)num, arg);
    return;
  }
```

だんだん書くのが疲れてきましたが、もう少しだけ進みます・・・。ここでは `st_lookup()` を使って `arg->data` というテーブルから `obj` を検索しています。これだけじゃ何をしているのか分からないので、if 文の中身を見ます。どうやら `TYPE_LINK` という値を dump しているらしいことが分かります。ちなみに `TYPE_LINK` が持つ値は '@' です。

```c
#define TYPE_LINK   '@'
```

ここで、dump データの先頭はそのデータの種類を表す文字が来る、ということを思い出しましょう。「`TYPE_LINK` なんてクラスあったっけ？」。そういうときは先程のフォーマット一覧の出番です！

> | '@' | オブジェクトの実態を指す番号(Fixnum形式 |
> 対応するオブジェクトが既に dump/load されている場合に使用される。番号は内部管理のもの。(dump/load 時に オブジェクト管理用にハッシュテーブルが作られる。そのレコード位置)
> <div style="text-align: right;">「Marshalフォーマット」より</div>

どうやら dump 済みのデータを再度 dump しないようにするための仕組みのようです。ということは `arg->data` は dump 済みデータの情報を管理しているテーブルで、`st_lookup()` で `obj` が dump 済みかどうか調べていることが分かります。検索によって dump 済みオブジェクトが見つかった場合はそのレコード位置が `num` にセットされます。検索結果が `true` の場合は、dump 済みデータを表す `TYPE_LINK ('@')` を書き込み、さらにレコード位置を `w_long()` で追記します。`w_long()` の中身はそのうち見ることにしましょう。ご察しの通り `long` 値を dump する関数です。

さて、処理の流れが分かったところで dump 済みオブジェクトを管理する必要が何故あるのか考えてみました。既に dump しているオブジェクトを再度 dump するのは二度手間になるから、つまり効率を上げるため、というのが真っ先に思い浮かびそうな話です。ただ理由はそれだけではなく、恐らくオブジェクトの参照の循環を避ける狙いもあるのでしょう。コンテナ型のオブジェクト、例えば Hash などは自分への参照を持つオブジェクトを要素として持つことができます。このようなオブジェクトに対して愚直に再帰的なシリアライズを行うと、無限ループになっていつまでも dump 処理が終わりません。これを避けるため、既に dump 済みのオブジェクトはそれへのポインタを追記するので済ませるのだと思います。


# まとめ

今回もあまり dump 処理について踏み込めませんでしたが、疲れたのでこの辺で終わります。dump 処理の基本は今まで書いたとおりなので、Ruby オブジェクトについて知ってれば後はサクサク読めると思います。

- `wbyte()`, `wnbyte()` はバイト単位でオブジェクト dump する関数
- dump データの先頭 2byte は Marshal のバージョン情報
- Marshal フォーマットは [ここ](http://www.ruby-lang.org/ja/old-man/html/Marshal_A5D5A5A9A1BCA5DEA5C3A5C8.html "Marshalフォーマット") を参照する
- 各オブジェクトの dump データの先頭 1byte はデータ種類を表す文字
- 巡回参照しているオブジェクトをシリアライズする時は、無限ループを避けるために同じオブジェクトの dump 処理を行わない (多分)

次回は `w_object()` の続きを見ていきます。
