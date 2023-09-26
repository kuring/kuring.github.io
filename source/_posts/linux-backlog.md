---
title: Linux TCP backlog
date: 2018-12-01 21:24:07
tags:
---

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/tcp_connect.webp)

一个正常的tcp server在处理请求时会经过如下的系统调用：socket() bind() listen() accept() read() write() close()。一个请求在被应用程序读取之前，可能处于SYN_RCVD和ESTABLISHED两种状态。

SYN_RCVD状态是server端接收到了client端的SYN包，server端会将该连接放到半连接队列中，并向客户端发送SYN+ACK包，此时连接处于半连接状态。

ESTABLISHED状态为已经完成了三次握手，但是server端的应用程序还未调用accept系统调用的情况。

这两种情况下都需要操作系统提供相应队列来保存连接状态。

backlog用来设置这两个队列的最大值，但在不同的操作系统中有不同的含义，下面的说明以linux操作系统为准。

其中第一个维护SYN_RCVD状态的队列使用内核参数`net.ipv4.tcp_max_syn_backlog`来控制，如果队列超过这一阈值，连接会被拒绝。该值默认为1000.

第二个维护ESTABLISHED状态的队列，该队列的长度由应用程序调用listen系统调用时指定。
