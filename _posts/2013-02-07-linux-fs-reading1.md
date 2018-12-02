---
layout: post
title: "Linux のファイルシステムを読む (1)"
date: 2013-02-07 00:06:00 +09:00
tags: linux codereading
image: /images/profile.png
---

ずっと気になっていた Linux のファイルシステムのコードを読み始めることにした。カーネルのコードなんてまともに読んだことがないので、ちょっとずつ頑張る。

Linux のコードベースでファイルシステム関係のものが入っているのは多分 linux/fs。このディレクトリにファイルシステム共通のコードがいて、その下位ディレクトリに各ファイルシステム特有のコードが入っているのかな？

```bash
$ tree linux/fs/ext2
linux/fs/ext2/
├── Kconfig
├── Makefile
├── acl.c
├── acl.h
├── balloc.c
├── dir.c
├── ext2.h
├── file.c
├── ialloc.c
├── inode.c
├── ioctl.c
├── namei.c
├── super.c
├── symlink.c
├── xattr.c
├── xattr.h
├── xattr_security.c
├── xattr_trusted.c
├── xattr_user.c
├── xip.c
└── xip.h
```
