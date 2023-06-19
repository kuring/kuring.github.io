---
title: getaddrinfo函数调用问题
Status: public
url: question_getaddrinfo
tags: C++
date: 2015-07-17
toc: yes
---

最近在开发程序的过程中遇到了一个getaddrinfo函数的问题，令我感到非常奇怪。

程序中调用了librdkafka库，当程序选择用-static方式链接所有库时程序会在librdkafka库中某个函数core dump，但是选择动态链接系统库（包括libpthread、libdl、libz、libm、libc等）时程序却能正常运行。

每次程序都回core dump在getaddrinfo函数中，经过搜索发现有人跟我遇到同样的[问题](https://sourceware.org/bugzilla/show_bug.cgi?id=10652)，但是却没有解决方案。

我这里实验了文中提到了例子，在静态链接的时候确实会报错，动态链接却非常正常，编译选项为`g++ -o test_getaddrinfo test_getaddrinfo.cpp -lpthread -Wall -static`。

```c++
include <stdio.h>
#include <netdb.h>
#include <pthread.h>
#include <unistd.h>

void *test(void *)
{
    struct addrinfo *res = NULL;
    fprintf(stderr, "x=");
    int ret = getaddrinfo("localhost", NULL, NULL, &res);
    fprintf(stderr, "%d ", ret);
    return NULL;
}

int main()
{
    for (int i = 0; i < 512; i++)
    {
        pthread_t thr;
        pthread_create(&thr, NULL, test, NULL);
    }
    sleep(5);
    return 0;
}
```

发现程序在链接的时候会提示如下警告：

```
/tmp/cc0WILtn.o: In function `test(void*)':
test_getaddrinfo.cpp:(.text+0x49): warning: Using 'getaddrinfo' in statically linked applications requires at runtime the shared libraries from the glibc version used for linking
/usr/lib/gcc/x86_64-redhat-linux/4.8.3/../../../../lib64/libpthread.a(libpthread.o): In function `sem_open':
(.text+0x685b): warning: the use of `mktemp' is dangerous, better use `mkstemp'
```

从网上查看有该警告的人还是非常多的，都是在-static方式链接glibc库时遇到的，但是没有发现很好的解决方案。该问题的原因
产生估计是glibc在静态链接时调用libnss库存在问题，因此不提倡静态链接方式。

我看到了两种解决方案：

方案一：用newlib或uClibc来代替glibc来静态链接，这种方案我没有去尝试是否可行。

方案二：用`--enable-static-nss`重新编译glibc。我试了一下问题仍然存在。

我之所以采用静态链接的方式，是因为开发机器和运行机器的glibc版本不一致造成的。我尝试将libc.so相关文件复制运行机器上，并让程序链接我复制过去的文件，ldd查看可执行文件没有错误，但是当运行程序时会报如下错误：

```
./xxx: relocation error: /home/kuring/lib/libc.so.6: symbol _dl_starting_up, version GLIBC_PRIVATE not defined in file ld-linux-x86-64.so.2 with link time reference
```

最终，我放弃了静态链接的方式，采用了动态链接方式来暂时解决了问题。如果你知道解决方案，请告诉我。

# 参考文章

* [getaddrinfo causes segfault if multithreaded and linked statically](https://sourceware.org/bugzilla/show_bug.cgi?id=10652)

* [Even statically linked programs need some shared libraries which is not acceptable for me.](https://sourceware.org/glibc/wiki/FAQ#Even_statically_linked_programs_need_some_shared_libraries_which_is_not_acceptable_for_me.__What_can_I_do.3F)

* [Create statically-linked binary that uses getaddrinfo?](http://stackoverflow.com/questions/2725255/create-statically-linked-binary-that-uses-getaddrinfo)

* [Static Linking Considered Harmful](http://www.akkadia.org/drepper/no_static_linking.html)