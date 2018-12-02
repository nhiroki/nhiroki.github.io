---
layout: post
title: "Fedora 16 に PostgreSQL をインストールする方法"
date: 2011-11-10 03:09:00 +09:00
tags: database
image: /images/profile.png
---

Fedora 16 で PostgreSQL のインストールに苦労したので，その手順をメモ．

```bash
$ sudo yum install postgresql postgresql-libs postgresql-server postgresql-devel
postgresql-contrib
$ sudo postgresql-setup initdb
```

postgres ユーザが自動で作成されている．データベースを起動するには postgres ユーザになる必要がある．

```bash
$ sudo su - postgres
postgres$ pg_ctl -w start
サーバの起動完了を待っています.........完了
サーバ起動完了
postgres% pg_ctl status
pg_ctl: サーバが動作中です
/usr/bin/postgres
```

自動起動の設定をする．

```bash
$ sudo chkconfig --add pgsql
$ sudo chkconfig --list pgsql
pgsql           0:off   1:off   2:on    3:on    4:on    5:on    6:off
```
