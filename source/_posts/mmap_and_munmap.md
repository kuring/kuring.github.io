---
title: mmap和munmap函数的用法
url: mmap_and_munmap
Tags: Linux
date: 2013-06-03 09:33:16
---


在Linux的头文件sys/mman.h中提供了两个用来分配内存的函数：mmap和munmap，函数定义原型如下：
```c
void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void *start, size_t length);
```
# mmap说明
返回值：内存映射后返回虚拟内存的首地址。
参数：
    start为指定的映射的首地址，该地址应该没有映射过，如果为0则有系统指定位置。
    length为映射的空间大小，真正分配空间大小为(length/pagesize+1)。
    prot为映射的权限，分为四种未指定（PROT_NONE）、读（PROT_READ）、写（PROT_WRITE）、执行（PROT_EXEC）。如果为PROT_WRITE，则直接可以PROT_READ。
    flags：映射方式，分为内存映射和文件映射。内存映射：匿名映射。当值为文件映射是后面两个参数才有效。常用的值有：MAP_ANONYMOUS、MAP_SHARED、MAP_PRIVATE。
    fd：映射的文件描述符。
    offset为从文件的偏移位置开始映射。
    
# munmap说明
从start位置开始释放length个字节的内存。

# 应用举例
```c
#include <stdio.h>
#include <sys/mman.h>
#include <stdlib.h>

main()
{
    int *p = mmap(NULL,
                  getpagesize(),
                  PROT_READ | PROT_WRITE,
                  MAP_ANONYMOUS | MAP_SHARED,
                  0,
                  0);
    munmap(p, getpagesize());
}
```
其中getpagesize()函数的作用为获取一个页的大小，系统默认为4K。
