---
title: Linux和Windows平台下的网络通信问题
Status: public
url: linux_windows_network
tags: C++
date: 2013-09-15 10:41:25
---


大小端问题跟CPU的架构直接相关，我们常见的80x86系列CPU采用小端字节序模式。Windows平台就采用的80x86系列CPU，因此为小端字节序。
而主机之间进行网络通信时往往采用大端字节序，因此小端字节序机器在发送数据前需要进行字节序转换，在接收到数据处理处理数据之前要将网络字节序转换成本地字节序。

在Linux平台下提供了四个函数用来字节序转换：
```
#include <arpa/inet.h>
uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

Windows平台下也提供了相关的自己序转换函数：
```
#include <WinSock2.h>
unsigned __int64 __inline htond(
  double value
);

unsigned __int32 __inline htonf(
  float value
);

u_long WSAAPI htonl(
  _In_  u_long hostlong
);

unsigned __int64 __inline htonll(
  unsigned __int64 value
);

u_short WSAAPI htons(
  _In_  u_short hostshort
);


double __inline ntohd(
  unsigned __int64 value
);

float __inline ntohf(
  unsigned __int32 value
);

u_long WSAAPI ntohl(
  _In_  u_long netlong
);

u_long __inline ntohll(
  unsigned __int64 value
);

u_short WSAAPI ntohs(
  _In_  u_short netshort
);

```

这里有个技巧需要说明以下，比如要发送如下的结构体：
```
struct foo
{
    int a;
    long b;
};
```
为了避免每个成员都调用字节序转换函数，可以在结构体的内部定义两个方法用于转换字节序，添加字节序后的foo如下：
```
struct foo
{
    int a;
    long b;
    void ntoh()
    {
         a = ntohl(a);
         b = ntohl(b);
    }
    void hton()
    {
         a = htonl(a);
         b = htonl(b);
    }
};
```

需要特别注意的是，在发送结构体类型的数据时要注意字节对齐的问题，这里不再展开讨论，不同的平台有不同的解决办法。大体分为Winodws平台、AIX平台和GNU类平台。
