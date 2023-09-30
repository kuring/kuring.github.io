title: TCP Fast Open
date: 2023-09-30 23:46:40
tags:
author:
---
![](https://kuring.oss-cn-beijing.aliyuncs.com/common/tcp_fast_open.webp)

TCP Fast Open（TFO）是一种TCP协议的扩展，旨在加快建立TCP连接的速度和降低延迟。传统的TCP连接需要进行三次握手（SYN-SYN/ACK-ACK）才能建立连接，而TFO允许在第一个数据包中携带连接建立的请求。

TFO的工作原理如下：

1. 客户端在首次建立TCP连接时，在发送的SYN包中插入一个加密的Cookie。这个Cookie由服务器生成并发送给客户端。
2. 当客户端发送带有TFO Cookie的SYN包到服务器时，服务器会验证Cookie的有效性。
3. 如果Cookie有效，服务器会立即发送带有SYN+ACK标志的数据包，这样客户端就可以立即发送数据而无需等待ACK响应。
4. 客户端收到带有SYN+ACK标志的数据包后，发送带有ACK标志的数据包，建立完整的TCP连接。


# 相关内核参数

## `net.ipv4.tcp_fastopen`

支持如下值：
- 0：关闭
- 1: 作为客户端可以使用 TFO 功能
- 2: 作为服务端可以使用 TFO 功能
- 3: 作为客户端和服务端均可使用 TFO 

## `net.ipv4.tcp_fastopen_key`

用来产生 TFO 的 Cookie。