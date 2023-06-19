---
title: 在linux程序中获取和设置环境变量
url: linux_get_and_set_env_variable
tags: Linux
date: 2013-06-04 21:59:37
---

# shell中的环境变量
### 查看环境变量
1. 通过env命令可以查看所有的环境变量
2. 通过echo $环境变量名方式来查看单个环境变量
### 设置环境变量
export命令来设置环境变量

在程序中该如何获取和设置环境变量呢？

# 通过main函数的第三个参数
通常大家接触比较多的是两个参数的main函数，实际上还有一个包含三个参数的main函数，第三个参数为包含了系统的环境变量的二级指针。
用法如下：
```c
#include <stdio.h>

int main ( int argc, char *argv[], char *arge[])
{
    while (*arge)
    {
        printf("%s\n", *arge);
        arge++;
    }
    return 0;
}
```
实例将会输出该用户的所有环境变量。

# 通过外部环境变量environ
该变量定义在“unistd.h”头文件中，定义为：extern char **environ;
用法如下：
```c
#include <stdio.h>

extern char **environ;

int main ( int argc, char *argv[] )
{
    char **env = environ;
    while (*env)
    {
        printf("%s\n", *env);
        env++;
    }
    return 0;
} 
```
实例将会输出该用户的所有环境变量。

# 通过系统函数
在C语言的头文件“stdlib.h”中定义了三个和环境变量相关的函数：
```c
char *getenv(const char *name);
int setenv(const char *name, const char *value, int overwrite);
int unsetenv(const char *name);
```
比较简单，不再举例。