title: tcp
date: 2022-06-24 11:02:48
tags:
author:
---
## tcp状态


![](https://kuring.oss-cn-beijing.aliyuncs.com/common/tcp_state.png)

SYN_RCVD: 服务端状态，服务端已经接收到了客户端的SYN报文，但还未返回给客户端SYN报文。通常会特别短暂。

FIN_WAIT2: 主动关闭连接的一方，在接收到对端的ack回包后，但还未返回包给对端第二个FIN包。通常该状态比较短，如果看到大量的FIN_WAIT2状态，通常为性能问题。在下面例子中，可以看到主动关闭方为172.16.128.40。

```
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 172.16.128.40:9876      172.16.128.2:36878      FIN_WAIT2   -
```

