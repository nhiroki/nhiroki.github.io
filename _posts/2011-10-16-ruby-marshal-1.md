---
layout: post
title: "Ruby の Marshal モジュールを読む (1)"
date: 2011-10-16 00:19:00 +09:00
tags: codereading
image: /images/profile.png
---

Ruby の Marshal モジュールの実装を読んでいるので、それについて (モチベーションが続く範囲で) まとめていきたいと思います。

なるべく平易にまとめるつもりですが、Ruby の拡張ライブラリが読み書きできる程度の知識がないと辛いかも知れないです。ちなみに、Ruby 1.9.2-p180 に付属の Marshal モジュールを読んでいます。

# Marshal とは？

Marshal は Ruby オブジェクトをシリアライズするためのモジュールです。シリアライズとは構造を持った Ruby オブジェクトをバイト列 (以後 "Marshal データ" と表記) に変換し、ストリーム型の通信路やファイルシステムに格納できるようにすることです。もちろん、バイト列を Ruby オブジェクトへと復元することもできます。

Marshal モジュールは Ruby ソースディレクトリ内の marshal.c にその実装があります。`Init_marshal()` という関数が Ruby 処理系起動時に実行され、Marshal モジュールのメソッドを定義します。この関数のコメントは Marshal モジュールの概要を説明したものとなっているので、今回はこれを見ていきます。

# Marshal のバージョン

Marshal したデータには、Marshal モジュールのバージョン情報 (Major, Minor) が付加されます。

> Marshaled data has major and minor version numbers stored along with the object
> information. In normal use, marshaling can only load data written with the same
> major version number and an equal or lower minor version number.

同じ、もしくは互換性を持ったバージョン間でしかデータの dump/load を行うことができません。Marshal モジュールのバージョンアップに伴って、何度かバイナリフォーマットが変更されているようですね。

> You can extract the version by reading the first two bytes of marshaled data.

```ruby
str = Marshal.dump("thing")
RUBY_VERSION   #=> "1.9.0"
str[0].ord     #=> 4
str[1].ord     #=> 8
```

バージョン情報は Marshal データの 1byte 目と 2byte 目に保存されるようです。Marshal のバイナリフォーマットは後々読み進めるのに必要となるので、覚えておきます。

# Marshal dump 不可能なオブジェクト

Marshal には dump できないオブジェクトが存在します。

> Some objects cannot be dumped: if the objects to be dumped include bindings,
> procedure or method objects, instances of class IO, or singleton objects, a
> TypeError will be raised.

ここでは、binding オブジェクト、proc オブジェクト、method オブジェクトを持つオブジェクトは dump できないと紹介されています。また、IO クラスのインスタンスや特異オブジェクトも駄目です。これらのオブジェクトを marshal しようとすると、`TypeError` 例外を投げます。

# Marshal dump/load の拡張

Marshal は独自の dump/load 関数を使うための拡張ポイントを提供しています。

> If your class has special serialization needs (for example, if you want to
> serialize in some specific format), or if it contains objects that would
> otherwise not be serializable, you can implement your own serialization
> strategy.
>
> There are two methods of doing this, your object can define either marshal_dump
> and marshal_load or _dump and _load.  marshal_dump will take precedence over
> _dump if both are defined.  marshal_dump may result in smaller Marshal strings.

`Object#marshal_dump`/`Object#marshal_load`, `Kernel.#_dump`/`Kernel.#_load` がそれです。両者の違いを理解するのに少し時間がかかりました。

```ruby
   class MyObj
     def initialize name, version, data
       @name    = name
       @version = version
       @data    = data
     end

     def marshal_dump
       [@name, @version]
     end

     def marshal_load array
       @name, @version = array
     end
   end
```

`marshal_dump`/`marshal_load` は実際に dump/load するオブジェクトを指定するのに使うようです。`marshal_dump` の返り値が dump されます。dump する必要がないオブジェクトがある場合、このメソッドでシリアライズ対象から外してあげることで、marshal データのサイズを減らすことができそうです。

```ruby
   class MyObj
     def initialize name, version, data
       @name    = name
       @version = version
       @data    = data
     end

     def _dump level
       [@name, @version].join ':'
     end

     def self._load args
       new(*args.split(':'))
     end
   end
```

一方、`_dump`/`_load` は dump/load 処理自体をユーザ自身で記述するために使います。Marshal モジュールのインタフェースで独自のシリアライズフォーマットを使うことができる、といった感じでしょうか。どういう場合に使うんでしょう？

# まとめ

今回は `Init_marshal()` のコメントを読むことで、Marshal モジュールの概要を掴みました。Marshal では dump したデータ内にバージョン情報を埋め込むことでデータの互換性を確認していること、Marshal 不可能なオブジェクトがあること、そしていくつかの拡張ポイントが用意されていることが分かりました。

次回は dump/load 処理の具体的な実装について見ていこうと思います。
