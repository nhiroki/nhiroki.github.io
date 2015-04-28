---
layout: post
title:  "Octpress の codeblock でオプションが効かない"
date:   2013-11-11 02:00:00
tags: blog
---

TL;DR Octpress の codeblock の start, mark, linenos オプションは現状 site-2.1 ブランチでしか使えない．

Octpress の plugin の一つにシンタックスハイライトしたコードを表示する codeblock というものがあります．

公式のドキュメント [octpress.org](http://octopress.org/docs/plugins/codeblock/) を見ると，codeblock のオプションとして start, mark, linenos というものを指定すれば，特定の行をさらにハイライトすることができるとなっていますが，この機能は今日現在 (2013/11/11)，master や 2.5 ブランチ，3.0 ブランチでは取り込まれておらず，site-2.1 ブランチでしか動かないようです (そもそもこのドキュメントは 2.0 用のもので，2011 年からアップデートされてない)．

対応していないブランチを使ってる人は自分で書くか cherry-pick する必要があるのでご注意を．関連する issue は次の通りです．

 * [pygments.rb options not working (Issue #1382)](https://github.com/imathis/octopress/issues/1382)
 * [codeblock plugin: highlight a line? (Issue #584)](https://github.com/imathis/octopress/issues/584)
