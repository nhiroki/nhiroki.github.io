---
layout: post
title: "MongoDB のインストール"
date: 2010-11-18 13:57:00 +09:00
tags: web
image: /images/profile.png
---

Fedora 12 に MongoDB をインストールしたときのメモ。

```bash
$ sudo yum install mongodb mongodb-devel mongodb-server
```

サーバの起動

```bash
$ sudo /etc/init.d/mongod start
```

動作確認

```bash
$ mongo
MongoDB shell version: 1.6.3
connecting to: test
> db.foo.save({a:1})
> db.foo.find()
{ "_id" : ObjectId("4ce4b0ef2fc8b1cc66f4104a"), "a" : 1 }
> exit
bye
```
