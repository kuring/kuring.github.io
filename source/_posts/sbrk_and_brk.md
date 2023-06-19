---
title: sbrk和brk函数
url: sbrk_and_brk
Tags: Linux
date: 2013-06-02 13:39:54
---

Linux系统中提供了两个在堆中分配空间的底层函数，函数原型如下：


void *sbrk(intptr_t increment);
int brk(void *end_data_segment);

两个函数的作用均为从堆中分配空间，并且在内部维护一个指针，指针的值默认为NULL。如果内部指针为NULL，则得到一页的空闲地址，系统默认为4K字节。指针向后移动即为分配空间，指针向前移动为释放空间。当内部指针的位置移动到一个页的开始位置时，整个页会被操作系统回收。brk为绝对改变位置，sbrk为相对改变位置。

# sbrk函数
在sbrk函数中，参数increment为要增加的字节数，increment可以为负数。当increment为负数时表示释放空间。当increment==0时，内部指针位置不动。函数调用成功返回内部指针改变前的值，失败返回(void *)-1。
函数使用举例如下：
```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    int *p0 = sbrk(0);
    // 打印堆地址
    printf("this site of p0 is : %d\n", p0);    

    int *p1 = sbrk(1000);
    // 这里仍然打印的是第一次的堆地址
    printf("this site of p1 is : %d\n", p1);    
    
    int *p2 = sbrk(1);
    // 打印第一次堆地址+1000后的地址
    printf("this site of p2 is : %d\n", p2);    
    
    // 回到初始堆地址，释放空间
    sbrk(-1001);
    
    int *p3 = sbrk(0);
    // 检查是否回到初始地址
    printf("this site of p3 is : %d\n", p3);
}
```
输出如下内容：
this site of p0 is : 264622080
this site of p1 is : 264622080
this site of p2 is : 264623080
this site of p3 is : 264622080


# brk函数
在brk函数中，函数作用为将当前的内部指针移动到end_data_segment位置。成功返回0，失败返回-1。
函数使用举例如下：
```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    int *p0 = sbrk(0);
    printf("this site of p0 is : %d\n", p0);
    
    brk(p0 + 1000);
    printf("this site is : %d\n", sbrk(0));
    
    brk(p0 + 1001);
    printf("this site is : %d\n", sbrk(0));
    
    brk(p0);
    printf("this site is : %d\n", sbrk(0));
}
```
函数输出如下：
this site of p0 is : 206979072
this site is : 206983072
this site is : 206983076
this site is : 206979072


# 适用场景：
内存的管理方式比malloc和free更加灵活，适合申请不确定的内存空间的情况，特别适合同类型的大块数据。如果用malloc则可能存在申请内存空间过多浪费的情况，过少时需要重新调用realloc来重新申请内存的情况。速度比malloc快。