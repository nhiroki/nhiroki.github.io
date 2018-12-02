---
layout: post
title: "WebSocket のミニマムな client/server サンプル"
date: 2013-08-17 21:35:00 +09:00
tags: web
image: /images/profile.png
---

WebSocket のミニマムな client/server サンプルをメモ。

# サーバサイド

サーバサイドは [EM-WebSocket](https://github.com/igrigorik/em-websocket "EM-WebSocket") を使った Ruby による実装。送られてきたメッセージを接続済みの clients に送信する。

```ruby
require 'em-websocket'

conns = Array.new

EM::WebSocket.start(:host => "0.0.0.0", :port => 8080) do |ws|
  ws.onopen do
    puts "Connected"
    ws.send "Connected"
    conns.push(ws) unless conns.index(ws)
  end

  ws.onmessage do |msg|
    puts "Received #{msg}"
    ws.send msg
    conns.each do |conn|
      conn.send(msg) unless conn == ws
    end
  end

  ws.onclose do
    puts "Closed"
    conns.delete(ws)
  end
end
```

'em-websocket' の gem をインストール後、`$ ruby server.rb` でサーバを起動。

# クライアントサイド

適当な html ファイルで次のスクリプトをロードする。

```js
var ws = new WebSocket("ws://localhost:8080");

ws.onclose = function() {
  log("Closed");
};

ws.onopen = function() {
  log('Connected');
  ws.send("Hello");
};

ws.onmessage = function(e) {
  log(e.data);
};

function log(msg) {
  console.log("[log] " + msg);
};
```
