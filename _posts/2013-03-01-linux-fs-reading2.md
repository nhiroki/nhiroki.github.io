---
layout: post
title: "Linux のファイルシステムを読む (2)"
date: 2013-03-01 01:30:00 +09:00
tags: linux codereading
image: /images/profile.png
---

"ソースコードを読むときは中心的なデータ構造から眺めるべし" と昔何かで教わったので、ファイルシステムの中心的なデータである inode 情報を眺めてみる。

```c
// linux/fs/ext2/ext2.h
/*
 * Structure of an inode on the disk
 */
struct ext2_inode {
    __le16  i_mode;     /* File mode */
    __le16  i_uid;      /* Low 16 bits of Owner Uid */
    __le32  i_size;     /* Size in bytes */
    __le32  i_atime;    /* Access time */
    __le32  i_ctime;    /* Creation time */
    __le32  i_mtime;    /* Modification time */
    __le32  i_dtime;    /* Deletion Time */
    __le16  i_gid;      /* Low 16 bits of Group Id */
    __le16  i_links_count;  /* Links count */
    __le32  i_blocks;   /* Blocks count */
    __le32  i_flags;    /* File flags */
...
}

/*
 * second extended file system inode data in memory
 */
struct ext2_inode_info {
    __le32  i_data[15];
    __u32   i_flags;
    __u32   i_faddr;
    __u8    i_frag_no;
    __u8    i_frag_size;
    __u16   i_state;
    __u32   i_file_acl;
    __u32   i_dir_acl;
    __u32   i_dtime;
...
}
```

メンバの詳細は追々見ていくとして、どうやら inode はディスク上の形式とメモリ上の形式の二種類があるらしく、両者の構造はだいぶ違う。頻繁に読み書きするものはメモリに置いておいて、あとはディスクに書き出しておくということなんだろう。

ちなみに `__u32` は "unsigned 32bit"、`__le16` は "little endian 16bit" を意味し、次のように定義されている。

```c
// linux/include/uapi/linux/types.h
typedef __u16 __bitwise __le16;
typedef __u16 __bitwise __be16;
typedef __u32 __bitwise __le32;
typedef __u32 __bitwise __be32;
typedef __u64 __bitwise __le64;
typedef __u64 __bitwise __be64;
```

`__bitwise` は sparse と呼ばれる静的コード解析ツールのためのアノテーションで、endian をごちゃ混ぜにしないように指定するものらしい ([Sparse (Wikipedia)](http://en.wikipedia.org/wiki/Sparse "Sparse (Wikipedia)"))。
