---
title: Linux Buffer与Cache的含义
date: 2019-01-29 00:27:45
tags:
---

Linux中的Buffer与Cache的含义通常非常容易混淆，两者翻译成中文都可以叫做缓存，都是数据在内存中的临时存储，而且网络上很多文章都是错误的。

```
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           125G         12G        347M        9.3M        113G        113G
Swap:            0B          0B          0B
```

free命令直接将buff和cache写到了一块，说明两者有很多共同点。

```
$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 7  1      0 364076  18664 118624552    0    0   214 11198  106  118  6  4 89  1  0
13  1      0 349096  18664 118638192    0    0     0 1012404 171031 270124 20 13 66  2  0
```

而通过vmstat命令可以分别看到buffer和cache的大小，单位为KB。

使用`man free`命令看到的解释如下：

```
buffers: Memory used by kernel buffers (Buffers in /proc/meminfo)

cache: Memory used by the page cache and slabs (Cached and Slab in /proc/meminfo)
```

查看proc的man手册结果如下：

```
Buffers %lu
    Relatively temporary storage for raw disk blocks that shouldn't get tremendously large (20MB or so).

Cached %lu
    In-memory cache for files read from the disk (the pagecache).  Doesn't include SwapCached.

SReclaimable %lu (since Linux 2.6.19)
    Part of Slab, that might be reclaimed, such as caches.

SUnreclaim %lu (since Linux 2.6.19)
    Part of Slab, that cannot be reclaimed on memory pressure.
```

上述信息，文档写的并不是非常明确。

可以看出buffers是磁盘数据的缓存，通常不会特别大，缓存的数据包括磁盘的写请求和读请求。内核用于将分散的写磁盘操作集中起来，批量写入磁盘。

Cached是文件数据的缓存，同样可以缓存读请求和写请求。

Slab包括了SReclaimalbe和Sunreclaim两部分信息，其中SReclaimable是可回收部分，SUnreclaim是不可回收部分。

关于文件和磁盘的区别如下：

磁盘是一个块设备，可以划分为多个分区，每个分区上可以构建不同的文件系统，文件系统挂载到目录上后，就可以对该文件系统进行读写文件操作了。

读写普通文件系统中的文件时，会经过文件系统，由文件系统跟磁盘进行交互，而文件系统的缓存为cache。读写磁盘或者分区时，会跳过文件系统，直接对磁盘进行操作，而操作系统对磁盘的缓存称之为buffer。

## ref

- [Linux Programmer's Manual PROC[5]](http://man7.org/linux/man-pages/man5/proc.5.html)
