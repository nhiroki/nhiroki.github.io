---
layout: post
title: "Asset (css, js, image) ファイルの置き場所 (CodeIgniter)"
date: 2011-09-16 01:35:00 +09:00
tags: web
image: /images/profile.png
---

CodeIgniter で asset ファイル (css, js, image など) を何処に置けば良いか分からず悩んだ．Rails だと public フォルダに置いて helper メソッドで位置を指定するんだけど，CI にはそれらしきフォルダが見つからず．

ググッて見たら下のような記事が見つかった．

- [CodeIgniter: How do I include a .js file in view?](http://stackoverflow.com/questions/1543407/codeigniter-how-do-i-include-a-js-file-in-view "CodeIgniter: How do I include a .js file in view?")

これを眺めていると，`base_url()` を使って base タグにアプリケーションルートを設定し，そこに asset を置く方法が紹介されていた．詳しくは次のページを参照．

- [Asset handling in CodeIgniter with the BASE tag](http://philsturgeon.co.uk/news/2009/09/Asset-handling-in-CodeIgniter-with-the-BASE-tag "Asset handling in CodeIgniter with the BASE tag")

# やり方

やり方は簡単．アプリケーションのルート (application/config/config.php の `$config['base_url']` で指定した場所) に css や js のディレクトリを作り，その中に asset ファイルを配置．controller の head タグ内で次のように設定する．

```html
<base href="<?php echo base_url(); ?>">
```

これで asset ファイルをアプリケーションルートからの相対パスで指定できるので，次のように記述することができる．

```html
<link rel="stylesheet" type="text/css" href="css/main.css" />
<script type="text/javascript" src="js/prototype.js"></script>
```

これがデファクトなやり方かは分かりません :-p


# まとめ

- CodeIgniter で Asset (css, js, image, など) ファイルを置く場所は base タグと `base_url()` を使って指定する
- ただし，これが普通のやり方かは分からない (CI に詳しい方，是非教えて下さい)．

# 追記 (2011.09.16)

html helper に `link_tag()` と `img()` があることに気付きました．

[URL ヘルパー : CodeIgniter ユーザガイド 日本語版 Version 2.0.3](http://codeigniter.jp/user_guide_ja/helpers/url_helper.html "URL ヘルパー : CodeIgniter ユーザガイド 日本語版 Version 2.0.3")

前述の「やり方」に書いたとおり，アプリケーションのルートに css と image ディレクトリを作り，次のように使います (base タグの設定は不要です)．

```php
<?php echo link_tag("css/main.css"); ?>
<?php echo img("image/icon.png"); ?>
```

これが次のように展開されます．

```html
<link href="http://example.com/app/css/main.css" rel="stylesheet" type="text/css" />
<img src="http://example.com/app/image/logo.png" />
```

`script_tag()` はないのかな？