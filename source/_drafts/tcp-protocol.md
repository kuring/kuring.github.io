title: tcp协议
date: 2022-03-26 23:34:21
tags:
author:
---
## tcp会话

### 如何查看

conntrack -L

### 会话超时回收



## 超时重传机制

发送端一旦确认未收到对端的ack请求，则需要重传尚未ack确认的数据。

### 内核参数

跟超时重传相关的内核参数包括：
- net.ipv4.tcp_retries1
- net.ipv4.tcp_retries2

### 如何确认发生了超时重传

一般伴随着发送队列的堆积，可以查看Send-Q队列是否有数据。

https://ata.alibaba-inc.com/articles/232868?spm=ata.25287382.0.0.482135625roN2C
待补充：https://zhuanlan.zhihu.com/p/101702312

## tcp keepalive机制

### 内核参数

- net.ipv4.tcp_keepalive_intvl
- net.ipv4.tcp_keepalive_probes
- net.ipv4.tcp_keepalive_time

