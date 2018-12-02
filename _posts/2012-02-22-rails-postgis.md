---
layout: post
title: "PostGIS を使った Rails アプリケーションのテスト"
date: 2012-02-22 14:34:00 +09:00
tags: web
image: /images/profile.png
---

PostGIS の位置情報カラムを使った Rails アプリケーションのテストを実行する時、次のようなエラーを表示して失敗してしまう (Rails 3.1.1, PostgreSQL 9.1.1, PostGIS 1.5.3)。

```bash
$ bundle exec rake test:functional
rake aborted!
PGError: ERROR:  関数addgeometrycolumn(unknown, unknown, integer, unknown,
integer)は存在しません
LINE 1: ...d_at" timestamp, "updated_at" timestamp) ; SELECT AddGeometr...
```

この原因は test 用の DB を構築する際に PostGIS 関連の関数がロードされないかららしい。そこで、db/schema.rb から test 用 DB を構築するのではなく、development 用 DB のスキーマをダンプした SQL から構築するようにする。

SQL で test 用 DB を構築するには config/environments/test.rb を次のように修正する (コメントを読むと PostGIS に限らず DB 固有のカラム型を使う場合は SQL を使えと書いてある)。

```bash
$ vim config/environments/test.rb
   # Use SQL instead of Active Record's schema dumper when creating the test
database.
   # This is necessary if your schema can't be completely dumped by the schema
dumper,
   # like if you have constraints or database-specific column types
   # config.active_record.schema_format = :sql
   config.active_record.schema_format = :sql      (<= コメントを外す)
```

あとはテストを実行すれば OK。

```bash
$ bundle exec rake test:functionals RAILS_ENV=test
```

テスト実行中に development 用 DB のスキーマを複製した db/test_structure.sql が生成され、これをもとに test 用 DB が生成される。

# 追記 (2012/02/22)

db/test_structure.sql が生成されない場合は、

```bash
$ bundle exec rake db:test:clone_structure
$ mv development_structure.sql test_structure.sql
$ bundle exec rake test:functionals RAILS_ENV=test
```

とすれば大丈夫そう。
