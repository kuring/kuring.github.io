title: tcp协议 - timestamp机制
date: 2022-03-28 19:53:30
tags:
author:
---
tcp协议中的timestamp选项，引入该选项主要用来解决如下的问题：

1. tcp协议为了提高效率，接收端在接收到N个报文后，向发送端回复ack报文。但发送端在发送出报文后，到底需要多长时间能够接收到ack报文是不确定的，因为一旦报文丢失后，无法预估出timeout时间。因此tcp协议的往返时间测量非常关键，但该值却非常难测量准确。
2. tcp报文中的seq号的字段长短为4个字节，这就限制了发送端在没发送2^32个长度的数据后就必须等待对端的ack报文。这对于高速的网络连接会影响通讯性能。


在引入timestamp选项后，