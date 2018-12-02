---
layout: post
title: "Ruby の Marshal モジュールを読む (2) | marshal_dump()"
date: 2011-10-17 01:14:00 +09:00
tags: codereading
image: /images/profile.png
---

引き続き Ruby の Marshal モジュールを読んでいきます。

- [Ruby の Marshal モジュールを読む (1)](/2011/10/16/ruby-marshal-1 "Ruby の Marshal モジュールを読む (1)")

# marshal_dump()

今回はオブジェクトのシリアライズ処理に踏み込んでいきます。オブジェクトのシリアライズ、デシリアライズはそれぞれ `dump` メソッド、`load` メソッドによって行われますが、これは C 言語レベルの `marshal_dump()` と `marshal_load()` に対応しています。

というわけで、まずは `marshal_dump()` を見て行きましょう。

```c
// ruby-1.9.2-p180/marshal.c
static VALUE
marshal_dump(int argc, VALUE *argv)
{
  VALUE obj, port, a1, a2;
  int limit = -1;
  struct dump_arg *arg;
  volatile VALUE wrapper;

  port = Qnil;
  rb_scan_args(argc, argv, "12", &obj, &a1, &a2);
  if (argc == 3) {
    if (!NIL_P(a2)) limit = NUM2INT(a2);
    if (NIL_P(a1)) goto type_error;
    port = a1;
  }
  else if (argc == 2) {
    if (FIXNUM_P(a1)) limit = FIX2INT(a1);
    else if (NIL_P(a1)) goto type_error;
    else port = a1;
  }
  wrapper = TypedData_Make_Struct(rb_cData,
                                  struct dump_arg,
                                  &dump_arg_data, arg);
  arg->dest = 0;
  arg->symbols = st_init_numtable();
  arg->data    = st_init_numtable();
  arg->infection = 0;
  arg->compat_tbl = st_init_numtable();
  arg->encodings = 0;
  arg->str = rb_str_buf_new(0);
  if (!NIL_P(port)) {
    if (!rb_respond_to(port, s_write)) {
type_error:
      rb_raise(rb_eTypeError, "instance of IO needed");
    }
    arg->dest = port;
    if (rb_respond_to(port, s_binmode)) {
      rb_funcall2(port, s_binmode, 0, 0);
      check_dump_arg(arg, s_binmode);
    }
  }
  else {
    port = arg->str;
  }

  w_byte(MARSHAL_MAJOR, arg);
  w_byte(MARSHAL_MINOR, arg);

  w_object(obj, arg, limit);
  if (arg->dest) {
    rb_io_write(arg->dest, arg->str);
    rb_str_resize(arg->str, 0);
  }
  clear_dump_arg(arg);
  RB_GC_GUARD(wrapper);

  return port;
}
```

まずは変数の宣言から。

```c
  VALUE obj, port, a1, a2;
  int limit = -1;
  struct dump_arg *arg;
  volatile VALUE wrapper;
```

`limit` は dump 時に辿るオブジェクトの深さです。あるオブジェクトが別のオブジェクトをインスタンス変数に持ち、それが更に別のオブジェクトをインスタンス変数に持ち・・・というように、オブジェクトが入れ子状に存在する場合、延々と dump 処理が続いてしまう可能性があります。これに対し `limit` 値を指定することでその深さまでで処理を打ち切ることができます。デフォルト値は -1 で、この場合は無制限に辿ります。`limit` 内で辿り切れない場合は `ArgError` 例外を投げます。

`struct dump_arg *arg` は dump したデータなどを格納する構造体です。`wrapper` は `arg` をラップしたオブジェクトへのポインタを保持します。

```c
// ruby-1.9.2-p180/marshal.c
struct dump_arg {
  VALUE str;        # ダンプデータ
  VALUE dest;       # ダンプデータ出力先のポート
  st_table *symbols;
  st_table *data;
  st_table *compat_tbl;
  st_table *encodings;
  int infection;
};
```

`struct dump_arg` は上記のような構造になっています。コメントは私が加筆したものです。`str` は dump データを格納する `RString` オブジェクトへのポインタ、`dest` は dump データを出力するポートを保持しているようです。その他は現時点では分からないので、追々見ていくことにします。

```c
  port = Qnil;
  rb_scan_args(argc, argv, "12", &obj, &a1, &a2);
  if (argc == 3) {
    if (!NIL_P(a2)) limit = NUM2INT(a2);
    if (NIL_P(a1)) goto type_error;
    port = a1;
  }
  else if (argc == 2) {
    if (FIXNUM_P(a1)) limit = FIX2INT(a1);
    else if (NIL_P(a1)) goto type_error;
    else port = a1;
  }
```

9行目から20行目は `marshal_dump()` の引数を解析していますが、本筋から逸れるので省略。

```c
    wrapper = TypedData_Make_Struct(rb_cData, struct dump_arg, &dump_arg_data, arg);
    arg->dest = 0;
    arg->symbols = st_init_numtable();
    arg->data    = st_init_numtable();
    arg->infection = 0;
    arg->compat_tbl = st_init_numtable();
    arg->encodings = 0;
    arg->str = rb_str_buf_new(0);
```

先程見た `dump_arg` 構造体をラップし、構造体の初期化を行なっています。

```c
  if (!NIL_P(port)) {
    if (!rb_respond_to(port, s_write)) {
type_error:
      rb_raise(rb_eTypeError, "instance of IO needed");
    }
    arg->dest = port;
    if (rb_respond_to(port, s_binmode)) {
      rb_funcall2(port, s_binmode, 0, 0);
      check_dump_arg(arg, s_binmode);
    }
  }
  else {
    port = arg->str;
  }
```

出力先ポートが指定されている場合、その設定を行います。指定されていない場合は dump データ `arg->str` を戻り値 `port` に設定します。

```c
  w_byte(MARSHAL_MAJOR, arg);  # Major バージョンの dump
  w_byte(MARSHAL_MINOR, arg);  # Minor バージョンの dump

  w_object(obj, arg, limit);   # エントリポイント
```

`w_byte()` は dump データ `arg->str` に 1byte データを書き込む関数です。ここでは、dump データの先頭にバージョン情報を付加します。バージョン情報については [前回の記事](/blog/2011/10/16/ruby-marshal-1 "Ruby の Marshal モジュールを読む (1)") を参照してください。

**<span style="color: red;">このように接頭辞 `w_` が付く関数は dump データにオブジェクトを書きこむ関数です。逆に dump データからオブジェクトを読み込む関数は接頭辞として `r_` が付きます。</span>**

よって、`w_object()` は dump データに `object` を書き込む処理となります。この呼び出しを起点に、オブジェクトを次々と dump していくことになります。dunp データは引数に渡している `arg` の `str` メンバに追記されます。

```c
  if (arg->dest) {
    rb_io_write(arg->dest, arg->str);
    rb_str_resize(arg->str, 0);
  }
```

`arg->dest` が 0 でない場合、つまり出力先のポートが指定されている場合は、その出力先に結果を出力します。

```c
  clear_dump_arg(arg);
  RB_GC_GUARD(wrapper);

  return port;
```

`clear_dump_arg()` は dump に使ったデータ (arg) のクリーンアップを行います。

~~RB_GC_GUARD は wrapper が指すオブジェクトが GC されるのを防ぎます。この時点では wrapper オブジェクトはどのオブジェクトからも参照されていないため、GC で回収される可能性があります。このままだと、wrapper が保持している dump データが関数リターン後に参照できなくなってしまうため、明示的に GC からガードします。ちなみに、wrapper 生成から RB_GC_GUARD を呼び出すまでの間で GC されないのか気になるところですが、拡張ライブラリ実行中は明示的に実行しない限り GC が走らないため問題になりません (rb_gc() 関数で明示的に GC を走らせることができます)。~~

`RB_GC_GUARD` ですが、字面を見て勘違いしていました。これは `volatile` を付けた変数に代入することで、コンパイラの最適化による `wrapper` の消失を避けるためのコードのようです (2011/10/17 追記)。

# まとめ

今回は `marshal_dump()` の中身を見ました。

- `dump_arg` 構造体に dump データが保存されていく
- 引数 port で dump データの出力先を指定できる
- 引数 limit で dump を試みる深さを制限できる
- 接頭辞 `w_` が付くのが dump 用関数、`r_` が付くのが load 用関数
- ~~GC を意識する必要がある~~

次回は `w_` 系の dump 関数や marshal フォーマットについて見ていきます。
