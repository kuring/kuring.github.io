---
title: linux中获取错误信息的方式
url: linux_get_error_of_function
tags: Linux
date: 2013-06-02 15:04:33
---

当linux中的函数内部出错时通常函数会返回-1，并且将错误码保存到全局变量errno中，用来表示错误代码。errno全局变量包含在头文件errno.h文件中。下面给出三种打印错误信息的方法。
# perror函数
应用举例如下：
```c
#include <stdio.h>

int main(void)
{
    FILE *fp ;
    fp = fopen( "/root/noexitfile", "r+" );
    if ( NULL == fp )
    {
        perror("error : ");
    }
    return 0;
}
```
输出如下：
Permission denied

# strerror函数
strerror函数原型为：char *strerror(int errnum);将参数errnum转换为对应的错误码。
应用举例如下：
```c
#include <stdio.h>

int main(void)
{
    FILE *fp ;
    fp = fopen( "/root/noexitfile", "r+" );
    if ( NULL == fp )
    {
        printf("%s\n", strerror(errno));
    }
    return 0;
}
```
输出如下：
Permission denied

# printf中的%m打印
应用举例如下：
```c
#include <stdio.h>

int main(void)
{
    FILE *fp ;
    fp = fopen( "/root/noexitfile", "r+" );
    if ( NULL == fp )
    {
        printf("%m\n");
    }
    return 0;
}
```
输出如下：
Permission denied
