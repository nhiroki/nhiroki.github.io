---
layout: post
title: "VMware Player で ThinkPad のトラックポイントスクロールを使う"
date: 2011-11-01 02:58
tags: linux
image: /images/profile.png
---

VM 上の Linux で ThinkPad のトラックポイントスクロールを動かせるようにする方法。

今までネットに書かれているやり方を色々試しても動かず。半ば諦めていたのですが、次の方法を試したらあっさりと動きました。それをメモしておきます。自己責任でお願いします。

私の環境は次の通りです。

- マシン: ThinkPad X200
- ホスト OS: Windows Vista
- ゲスト OS: Fedora 15
- VMM: VMware Player 4.0.0

# やり方

まず、C:\Program Files\Lenovo\TrackPoint\tp4table.dat の Pass 0 rules の末尾に下記を追加します (ファイルパスは環境によって異なるので、適宜読み替えてください)。

```
# tp4table.dat
; VMware Player
*,*,vmware-vmx.exe,*,smooth
```

タスクマネージャから tp4serv.exe を終了させ、C:\Program Files\Lenovo\TrackPoint\tp4serv.exe を起動します。

これでゲスト OS 上でトラックポイントスクロールが使えるようになります。

# 私の環境で動かなかったやり方

解決策を検索してよく見かけるのは Pass 0 rules に

```
# tp4table.dat
; VMware Player
*,*,vmplayer.exe,*,*,*,WheelStd,0,9
```

を追加するというものですが、私の環境では動きませんでした。同様の症状で諦めかけていた方は是非試してみてください :-)
