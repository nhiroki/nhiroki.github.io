---
layout: post
title: "MongoDB を PHP から操作する"
date: 2010-11-18 15:56:00 +09:00
tags: web
image: /images/profile.png
---

前回の記事で MongoDB をインストールできたので、PHP から MongoDB にアクセスできるように設定をした。

# PHP の拡張モジュールのインストール

PHP から MongoDB にアクセスするために必要な拡張モジュールをインストールする。

```bash
$ sudo yum install php-pear
$ sudo pecl install mongo
```

mongo.ini ファイルを新しく作る。

```bash
$ sudo vim /etc/php.d/mongo.ini
; Enable mongo extension module
extension=mongo.so
```

HTTP で MongoDB を操作したい場合は、apache を再起動する。

```bash
$ sudo /etc/init.d/httpd restart
```

# PHP から MongoDB にアクセスする基本操作

まず、PHP からデータの挿入を行ってみる。

```bash
$ vim test1.php
```

```php
<?php
$mongo = new Mongo();
$db = $mongo->selectDB("test");
$col = $db->createCollection("test_col");
$col->insert(array("key" => "value"));
```

実行してみて、MongoDB に格納されているか確認。

```bash
$ php test1.php
$ mongo
> db.test_col.find()
{ "_id" : ObjectId("4ce4cb9b762fc80935000000"), "key" : "value" }
```

挿入されているのが確認できた。次に、PHP からデータの取得を行ってみる。

```bash
$ vim test2.php
```

```php
<?php
$mongo = new Mongo();
$db = $mongo->selectDB("test");
$col = $db->selectCollection("test_col");
$cur = $col->findOne();

var_dump($cur);
```

実行してみて、MongoDB からデータが取得できているか確認する。

```bash
$ php test2.php
array(2) {
  ["_id"]=>
  object(MongoId)#6 (1) {
    ["$id"]=>
    string(24) "4ce4cb9b762fc80935000000"
  }
  ["key"]=>
  string(5) "value"
}
```

取得できた。以上が基本的な使い方。

# 参考

あとは、PHP の Mongo のマニュアルを参照すれば何でもできそう。

[PHP: Mongo - Manual](http://www.php.net/manual/ja/book.mongo.php "PHP: Mongo - Manual")
