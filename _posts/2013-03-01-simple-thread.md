---
layout: post
title: "pp::SimpleThread でお手軽バックグラウンド処理"
date: 2013-03-01 21:43:00 +09:00
categories: web
image: /images/profile.png
---

Native Client の話。Pepper API の version は 25 です。

Pepper API には pp::SimpleThread というユーティリティーがあって、これがちょろっとバックグラウンドスレッドを使いたい時に便利なので紹介。

pp::SimpleThread[^1] は pthread/Windows thread をラップし、その上で pp::MessageLoop[^2] を走らせることができるユーティリティーです。pp::SimpleThread のインスタンスを作って、pp::MessageLoop に PostWork() でタスクを投げれば、それを別スレッドで実行してくれます。

使い方は簡単。

```cpp
pp::SimpleThread thread = new pp::SimpleThread(instance);
thread->message_loop()->PostWork(task);
```

これで task がバックグラウンドスレッドで実行されます。

NaCl module のメインスレッドでは blocking 処理を行えないので、Pepper API を叩くときは (1) asynchronous callback を渡して非同期に処理するか (callback 関数がずらずら・・・)、もしくは (2) 別スレッドで blocking callback で待つ (スレッド作るのめんどくさい・・・) 必要がありますが、pp::SimpleThread を使えば (2) の処理が手軽に書けます。

[^1]: ppapi/utility/threading/simple_thread.h
[^2]: [pp::MessageLoop Class Reference](https://developers.google.com/native-client/pepper25/peppercpp/classpp_1_1_message_loop "pp::MessageLoop Class Reference")
