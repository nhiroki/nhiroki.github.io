---
layout: post
title:  "pmap ソースコードリーディング"
date:   2013-12-05 00:00:00
categories: Linux Memory
---

昨日調べた pmap のソースコードを読んだので，メモしておきます．これから読む人の参考になればと思います．

pmap は指定したプロセスのメモリマッピングに関する情報を proc ファイルシステムから取得し，表示してくれるツールです．使い方はこちらの記事を参照してください．

pmap は procps と呼ばれるユーティリティ群の一つとして公開されています．ソースコードは procps の公式サイト からダウンロードできます．今回は procps 3.2.8 を読んでみました．

展開したファイルの中の pmap.c が本体です．372 行しかないので，気楽に読めます．

#ソースコードの詳細#

それでは，main 関数から見ていきます．

**main 関数**

長めですが，大半はオプションの解析です．360 行目辺りからが実際の処理を行なっている部分です．

{% highlight c %}
// pmap.c
359   discover_shm_minor();
360
361   pidlist[count] = 0;  // old libproc interface is zero-terminated
362   PT = openproc(PROC_FILLSTAT|PROC_FILLARG|PROC_PID, pidlist);
363   while(readproc(PT, &p)){
364     ret |= one_proc(&p);
365     if(p.cmdline) free((void*)*p.cmdline);
366     count--;
367   }
368   closeproc(PT);
{% endhighlight %}

 * discover_shm_minor 関数は共有メモリのメモリマッピング名を取るためのものです．詳細は後で見ます．
 * pidlist は引数で渡されたプロセス ID (pid) を詰め込んだリストです．
 * openproc/readproc/closeproc は PROCTAB (Process Table) というプロセスに関する情報を保持する構造体の初期化，読み込み，クローズを行う関数です．pmap ではここで取得する情報をほとんど使用しないので，今回は気にしないことにします．

readproc で読み込んだ情報を順次 one_proc 関数で処理します．

**one_proc 関数**

プロセスに関する情報を実際に読み取る関数です．

{% highlight c %}
// pmap.c
137   sprintf(buf,"/proc/%u/maps",p->tgid);
138   if(!freopen(buf, "r", stdin)) return 1;
{% endhighlight %}

tgid (Task Group ID) は PID を意味しています．”/proc/[pid]/maps” がプロセスのメモリマッピングに関する情報を提供しており，これを読み取るためにオープンしています．

{% highlight c %}
// pmap.c
154   while(fgets(mapbuf,sizeof mapbuf,stdin)){
155     char flags[32];
156     char *tmp; // to clean up unprintables
157     unsigned KLONG start, end, diff;
158     unsigned long long file_offset, inode;
159     unsigned dev_major, dev_minor;
160     sscanf(mapbuf,"%"KLF"x-%"KLF"x %31s %Lx %x:%x %Lu", &start, &end, flags, &file_offset, &dev_major, &dev_minor, &inode);
{% endhighlight %}

154 行目の while 文で，メモリマッピングを行単位で読み取っています．160 行目の sscanf では，


> 7fb242da5000-7fb242da6000 rw-p 00026000 fc:01 8519822 /lib/x86_64-linux-gnu/libtinfo.so.5.9

のようなメモリマップ情報を，

> start: 7fb242da5000
> end: 7fb242da6000
> flags: rw-p
> file_offset: 00026000
> dev_major: fc
> dev_minor: 01
> inode: 8519822

ようにパースします．あとはオプションに応じて必要な情報を集め，それらを整形して出力しています．

**mapping_name 関数**

{% highlight c %}
// pmap.c
189       const char *cp = mapping_name(p, start, diff, mapbuf, 0, dev_major, dev_minor, inode);
{% endhighlight %}

189 行目などで呼び出している mapping_name 関数は，その名の通りマッピング名 (ライブラリ名や [ stack ], [ anon ] など) を取得するための関数です．

{% highlight c %}
// pmap.c
 99 static const char *mapping_name(proc_t *p, unsigned KLONG addr, unsigned KLONG len, const char *mapbuf, unsigned showpath, unsigned dev_major, unsigned dev_minor, unsigned long long inode){
100   const char *cp;
101
102   if(!dev_major && dev_minor==shm_minor && strstr(mapbuf,"/SYSV")){
103     static char shmbuf[64];
104     snprintf(shmbuf, sizeof shmbuf, "  [ shmid=0x%Lx ]", inode);
105     return shmbuf;
106   }
107
108   cp = strrchr(mapbuf,'/');
109   if(cp){
110     if(showpath) return strchr(mapbuf,'/');
111     return cp[1] ? cp+1 : cp;
112   }
113
114   cp = strchr(mapbuf,'/');
115   if(cp){
116     if(showpath) return cp;
117     return strrchr(cp,'/') + 1;  // it WILL succeed
118   }
119
120   cp = "  [ anon ]";
121   if( (p->start_stack >= addr) && (p->start_stack <= addr+len) )  cp = "  [ stack ]";
122   return cp;
123 }
{% endhighlight %}

102 行目から 106 行目は共有メモリに関するマッピング名を取得する部分です．これは先程スキップした discover_shm_minor 関数が関わってくるので，詳しくは次の項を見てください．

108 行目から 118 行目はメモリマップの情報から ‘/’ を探すことで，マッピング名に当たる部分 (e.g. “/lib/x86_64-linux-gnu/libc-2.15.so”) を見つけようとしています．

121 行目ではマッピングのアドレスがメインスタックの範囲に含まれているか判定し，含まれる場合は “[ stack ]” とします．

これらの処理でマッピング名が分からなかった場合は “[ anon ]” とします．

**discover_shm_minor 関数**

pmap ではマッピング名を取得する際に，デバイスのマイナー番号 (dev_minor) が特定の番号のものを SYSV 共有メモリのマッピングとして判定するようです．その特定の番号を取得するのがこの関数です．

デバイス番号を取得するために，一旦 SYSV 共有メモリを自身にアタッチし，そのマッピング情報を調べるということをしています．

{% highlight c %}
// pmap.c
53 static void discover_shm_minor(void){
54   void *addr;
55   int shmid;
56   char mapbuf[256];
57
58   if(!freopen("/proc/self/maps", "r", stdin)) return;
59
{% endhighlight %}

58 行目で “/proc/self/maps” をオープンしていますが，これは読み込みを行うプロセス自身のメモリマッピングに関する情報を保持しています．この場合だと pmap のメモリマッピングに関する情報が取得されます．

{% highlight c %}
// pmap.c
60   // create
61   shmid = shmget(IPC_PRIVATE, 42, IPC_CREAT | 0666);
62   if(shmid==-1) return; // failed; oh well
63   // attach
64   addr = shmat(shmid, NULL, SHM_RDONLY);
65   if(addr==(void*)-1) goto out_destroy;
{% endhighlight %}

60 行目から 65 行目で，共有メモリの割り当てを行なっています．SYSV 共有メモリの使い方は下記を参照してください．

 * [Man page of SHMGET](http://linuxjm.sourceforge.jp/html/LDP_man-pages/man2/shmget.2.html)

{% highlight c %}
// pmap.c
67   while(fgets(mapbuf, sizeof mapbuf, stdin)){
68     char flags[32];
69     char *tmp; // to clean up unprintables
70     unsigned KLONG start, end;
71     unsigned long long file_offset, inode;
72     unsigned dev_major, dev_minor;
73     sscanf(mapbuf,"%"KLF"x-%"KLF"x %31s %Lx %x:%x %Lu", &start, &end, flags, &file_offset, &dev_major, &dev_minor, &inode);
74     tmp = strchr(mapbuf,'\n');
75     if(tmp) *tmp='\0';
76     tmp = mapbuf;
77     while(*tmp){
78       if(!isprint(*tmp)) *tmp='?';
79       tmp++;
80     }
81     if(start > (unsigned long)addr) continue;
82     if(dev_major) continue;
83     if(flags[3] != 's') continue;
84     if(strstr(mapbuf,"/SYSV")){
85       shm_minor = dev_minor;
86       break;
87     }
88   }
{% endhighlight %}

67 行目以降では，自身のメモリマップから先ほどアタッチした共有メモリを探しています．start アドレスがアタッチした共有メモリよりも後ろにあり，デバイスのメジャー番号が 0，フラグが “s”hared で，かつマッピング情報に ”/SYSV” を含んでいるものを探し，そのデバイスマイナー番号を採用しています．

以上が共有メモリのデバイスマイナー番号を取得する流れです．この番号をもとに，mapping_name 関数で共有メモリかどうかの判定を行います．

#まとめ#

プロセスのメモリマップについて表示してくれる pmap というツールのソースコードを読みました．メモリマップの情報は “/proc/[pid]/maps” の情報をパースしており，各マッピングの名前を取得する方法なども分かりました．

procps ユーティリティには top や vmstat といったコマンドも含まれており，こちらもそのうち読みたいと思っています．

#参考#

 * [procps – Home Page]([http://procps.sourceforge.net/)
 * [Man page of PMAP](http://linuxjm.sourceforge.jp/html/procps/man1/pmap.1.html)
 * [Man page of PROC](http://linuxjm.sourceforge.jp/html/LDP_man-pages/man5/proc.5.html)
 * [pmap でプロセスのメモリマッピングについて調べる – nhiroki’s weblog](/2013/12/03/pmap/)
