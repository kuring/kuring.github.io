---
title: Linux TCP backlog
date: 2018-12-01 21:24:07
tags:
---

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/tcp_connect.webp)

一个正常的tcp server在处理请求时会经过如下的系统调用：socket() bind() listen() accept() read() write() close()。一个请求在被应用程序读取之前，可能处于SYN_RCVD和ESTABLISHED两种状态。

第一个队列：SYN_RCVD状态是server端接收到了client端的SYN包，server端会将该连接放到半连接队列中，并向客户端发送SYN+ACK包，此时连接处于半连接状态。通常该队列被称为半连接队列。

第二个队列：ESTABLISHED状态为已经完成了三次握手，但是server端的应用程序还未调用accept系统调用的情况。通常该队列被称为全连接队列。

这两种情况下都需要操作系统提供相应队列来保存连接状态。

backlog用来设置这两个队列的最大值，但在不同的操作系统中有不同的含义，下面的说明以linux操作系统为准。

其中第一个维护SYN_RCVD状态的队列使用内核参数`net.ipv4.tcp_max_syn_backlog`来控制，如果队列超过这一阈值，连接会被拒绝。该值默认为1000.

第二个维护ESTABLISHED状态的队列，该队列的长度由应用程序调用listen系统调用时指定。

# 内核参数

1. net.ipv4.tcp_max_syn_backlog

```
tcp_max_syn_backlog (integer; default: see below; since Linux 2.2)
              The  maximum number of queued connection requests which have still not received an acknowledgement from the connecting client.  If this number is exceeded, the kernel will begin dropping requests.  The default value of 256 is increased to 1024 when the memory present in the system is adequate or greater (>= 128Mb), and  reduced  to 128  for  those systems with very low memory (<= 32Mb).  It is recommended that if this needs to be increased above 1024, TCP_SYNQ_HSIZE in include/net/tcp.h be modified to keep TCP_SYNQ_HSIZE*16<=tcp_max_syn_backlog, and the kernel be recompiled.
```

用来设置 syn 队列的大小，通常也会称为半连接队列。该参数的默认值一般为 1024。如果 syn 队列满，此时 syn 报文会被丢弃，无法回复 syn + ack 报文。可以通过 `netstat -s` 命令看到 "XX SYNs to LISTEN sockets dropped". 的报错信息。

2. net.ipv4.tcp_syncookies

```
tcp_syncookies (Boolean; since Linux 2.2) Enable TCP syncookies. The kernel must be compiled with CONFIG_SYN_COOKIES.  Send out syncookies when the syn backlog queue of a socket overflows.  The syncookies feature attempts to protect a socket from a SYN flood attack.  This should be used as a last resort, if at all.  This is a violation of the  TCP  protocol,  and  con‐ flicts  with other areas of TCP such as TCP extensions.  It can cause problems for clients and relays.  It is not recommended as a tuning mechanism for heavily loaded servers to help with overloaded or misconfigured conditions.  For recommended alternatives see tcp_max_syn_backlog, tcp_synack_retries, and tcp_abort_on_overflow.
```

因为 syn 队列的存在，当客户端一直在发送 syn 包，但是不回 ack 报文时，一旦服务端的队列超过 `net.ipv4.tcp_max_syn_backlog` 设置的大小就会存在队列溢出的问题，从而导致服务端无法响应客户端的请求，这就是 syn flood 攻击。

为了防止 syn flood 攻击，引入了 syn cookies 机制，该机制并非 tcp 协议的一部分。原理参见：[深入浅出TCP中的SYN-Cookies](https://segmentfault.com/a/1190000019292140)

一旦开启了 syn cookies 机制后，即使 syn 队列满，仍可以对新建的连接回复 syn + ack 报文，但是不需要进入队列。

因为 syn cookies 存在部分缺陷，只有当 syn 队列满时该特性才会生效。

3. net.ipv4.tcp_abort_on_overflow

在三次握手完成后，该连接会进入到 ESTABLISHED 状态，并将该连接放入到用户程序队列中。若该队列已满，默认会将该连接重新设置为 SYN_ACK 状态，相当于是服务端没有接收到客户端的 syn + ack 报文，后续可以利用客户端的重传机制重新接收报文。

一旦开启了 `net.ipv4.tcp_abort_on_overflow` 选项后，会直接发送 RST 报文给到客户端，客户端会终止该连接，并报错 `104 Connection reset by peer`。

4. net.core.somaxconn

全连接队列的最大值，该配置为全局默认配置。单个 socket 的全连接队列的长度选择为 Min(backlog, somaxconn)。

# 内核源码

队列位于内核代码的 include/net/inet_connection_sock.h 中的如下位置:

```c
struct inet_connection_sock {
	/* inet_sock has to be the first member! */
	struct inet_sock	  icsk_inet;
	struct request_sock_queue icsk_accept_queue; // 队列
	struct inet_bind_bucket	  *icsk_bind_hash;
}
```

# 如何参看

查看 tcp 状态为 SYN_RECV 的链接即为半连接状态的请求：`netstat -napt | grep SYN_RECV`。也可以通过 `netstat -s | grep 'SYNs to LISTEN'` 查看。

全连接队列可以使用 `ss -nlt | grep 8080` 的方式查看 Recv-Q 的值。
也可通过如下命令来查看全连接队列的溢出情况。

```shell
$netstat -s | grep overflow
    3255 times the listen queue of a socket overflowed
```
