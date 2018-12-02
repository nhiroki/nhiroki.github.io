---
layout: post
title: "Marshal dump した値には改行文字が含まれることがある"
date: 2011-08-02 13:39:00 +09:00
tags: web
---

Ruby で次のようなコードを書いたらエラーが出た。

```ruby
pin, pout = IO.pipe
Process.fork do
  pin.close
  pout.puts Marshal.dump(265)
  pout.puts Marshal.dump(266)
  pout.puts Marshal.dump(267)
  pout.close
end

pout.close
puts Marshal.load(pin.gets)  # OK
puts Marshal.load(pin.gets)  # `load': marshal data too short (ArgumentError)
puts Marshal.load(pin.gets)  # OK
pin.close
```

これは、266 を Marshal dump した値に改行文字が含まれているのが原因。

```bash
$ irb
ruby-1.9.2-p290 :002 > Marshal.dump(266)
 => "\x04\bi\x02\n\x01"
```

正しくはこうする。

```ruby
pin, pout = IO.pipe

Process.fork do
  pin.close
  pout.write Marshal.dump(265)
  pout.write Marshal.dump(266)
  pout.write Marshal.dump(267)
  pout.close
end

pout.close
puts Marshal.load(pin)  # OK
puts Marshal.load(pin)  # OK
puts Marshal.load(pin)  # OK
pin.close
```
