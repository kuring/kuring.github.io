title: TCP TIME_WAIT
date: 2018-12-17 23:02:09
tags:
---
## time_wait状态

客户端在收到服务器端发送的FIN报文后发送ACK报文，并进入TIME_WAIT状态，等待2MSL（最大报文生存时间）后才断开连接，MSL在Linux中值为30s。

之所以设计time_wait主要用来解决以下异常场景：

1. 确保对端处于关闭状态。主动断开连接一段发送最后一个ack报文，如果丢失，被动断开连接一端会重新发送fin报文。如果主动断开连接一方直接关闭，被动方会一直处于last-ack状态。
2. 防止上一个连接中的包影响新的连接，上一个连接中的包在2MSL中一定可以到达对端。

过多的危害：在客户端占用过多的端口号

## time_wait过多的解决思路

1. 将`net.ipv4.tcp_max_tw_buckets`值调小，当TIME_WAIT的数量到达该值后，TIME_WAIT状态会被清除，相当于没有遵守tcp协议
2. 修改TCP_TIMEWAIT_LEN的值，但需要重新编译内核，非常不建议修改
3. 打开tcp_tw_recycle和tcp_timestamps
4. 打开tcp_tw_reuse和tcp_timestamps
5. 采用长连接

tcp有个tcp时间戳选项，第一个是发送方的当前时钟时间戳（4个字节），第二个4字节为从远程主机接收到的最新时间戳

## 相关内核参数

### tcp_timestamp

用来控制tcp option字段，发送方在发送报文时会将当前时钟的时间值放入到时间戳字段。

### tcp_tw_reuse

tcp_tw_reuse意思为主动关闭连接的一方可以复用之前的time_wait状态的连接。

复用连接后，这条连接的时间更改为当前时间，延迟数据到达时，延迟数据时间小于新连接时间。

需要连接双方都打开timestamp选项。

该选项适用的范围为作为客户端主动断开连接，复用客户端的time_wait的状态，对服务端无影响。

### tcp_tw_recycle

内核会在一个RTO的时间内快速销毁掉time_wait状态，RTO时间为数据包重传的超时时间，该时间通过RTT动态计算，远小于2MSL。

需要连接双方都打开timestamp选项。

适用场景为服务端主动断开连接，time_wait状态位于服务端，服务端适用该选项快速回收time_wait状态的连接。

*弊端*：如果客户端在NAT网络中，如果配置了tcp_tw_recycle，可能会出现在一个RTO的时间内，只有一个客户端和自己连接成功的情况。

4.10之后，Linux内核修改了时间戳生成机制，该选项已经抛弃。


## In Action

解决time_wait状态过多的比较好的思路为采用http的keepalive功能。

### nginx

nginx对于upstream，默认是使用http1.0协议的，要想启用keepalive，需要在location中增加

```
proxy_http_version 1.1;
proxy_set_header Connection "";
```

在upstream中增加keepalive参数，这里的参数含义为每个nginx worker连接所有后端的最大连接数。

```
keepalive 200;
```

如果keepalive连接过少，此时由于使用的是http1.1的协议，upstream端不会主动断开连接，nginx会主动断开连接，此时nginx端的time_wait就会过多，会占用端口号，导致nginx端没有端口号可以使用。

## 引用

* [关于 Nginx upstream keepalive 的说明
](https://www.nosa.me/2014/12/18/%E5%85%B3%E4%BA%8E-nginx-upstream-keepalive-%E7%9A%84%E8%AF%B4%E6%98%8E/)
* [HAProxy and HTTP errors 408 in Chrome](http://www.haproxy.com/blog/haproxy-and-http-errors-408-in-chrome/)
* [被抛弃的tcp_recycle](https://mp.weixin.qq.com/s?__biz=MzUxMDQxMDMyNg==&mid=2247484841&idx=1&sn=7e0923ea9204e126003e263dc8414261&chksm=f9022e90ce75a7865a985ddfb02016949b0c36a02f8fb805c957699ba64144682b9b40d58a1c&mpshare=1&scene=1&srcid=1212u1NkZyHItSo6HpPq7eer%23rd)