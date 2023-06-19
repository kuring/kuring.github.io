---
title: Linux常用网络诊断工具整理
Status: public
url: linux_net_tools
date: 2014-02-08
---

本文对Linux常用的网络命令进行整理和总结。

# 连通性测试

## ping命令
最常用的网络诊断命令。
```
[kuring@localhost ~]$ ping www.baidu.com
PING www.a.shifen.com (61.135.169.125) 56(84) bytes of data.
64 bytes from 61.135.169.125: icmp_seq=1 ttl=128 time=25.5 ms
64 bytes from 61.135.169.125: icmp_seq=2 ttl=128 time=20.3 ms
64 bytes from 61.135.169.125: icmp_seq=3 ttl=128 time=25.0 ms
64 bytes from 61.135.169.125: icmp_seq=4 ttl=128 time=21.7 ms
64 bytes from 61.135.169.125: icmp_seq=5 ttl=128 time=23.4 ms
64 bytes from 61.135.169.125: icmp_seq=6 ttl=128 time=21.9 ms
```
通过上述输出可以看出，ping命令可得到DNS对应的IP信息、ping的数据包大小、网络延迟信息。

另外，可以通过-s参数指定ping的数据包大小。例如：
```
kuring@ubuntu:~$ ping www.baidu.com -s 1024
PING www.a.shifen.com (61.135.169.125) 1024(1052) bytes of data.
1032 bytes from 61.135.169.125: icmp_req=1 ttl=55 time=22.6 ms
1032 bytes from 61.135.169.125: icmp_req=2 ttl=55 time=22.9 ms
1032 bytes from 61.135.169.125: icmp_req=3 ttl=55 time=53.0 ms
1032 bytes from 61.135.169.125: icmp_req=4 ttl=55 time=28.0 ms
1032 bytes from 61.135.169.125: icmp_req=5 ttl=55 time=54.7 ms
1032 bytes from 61.135.169.125: icmp_req=6 ttl=55 time=93.1 ms
1032 bytes from 61.135.169.125: icmp_req=7 ttl=55 time=26.9 ms
1032 bytes from 61.135.169.125: icmp_req=8 ttl=55 time=25.2 ms
1032 bytes from 61.135.169.125: icmp_req=9 ttl=55 time=25.4 ms
^C1032 bytes from 61.135.169.125: icmp_req=10 ttl=55 time=21.2 ms

--- www.a.shifen.com ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 45436ms
rtt min/avg/max/mdev = 21.225/37.350/93.149/21.966 ms
```
会发现ping命令的响应时间变长了，这正是由于ping发送数据包变大了。

## traceroute
可以显示路由信息。

## mtr
traceroute命令的升级版，可以动态刷新路由信息，可以显示路由上每个节点的丢包率和时间等信息，信息比较全面和直观。

# arp相关

## arping
可以通过该命令查看IP地址对应的mac地址。`arping IP地址`会立即发送一个arp广播，可以根据收到的arp回应的多少看局域网内是否中arp病毒、IP地址冲突等情况。

## arp
跟arp协议相关，可以设置arp表、读取arp表等。

# 端口相关

## telnet
可以利用该命令来测试某个端口是否打开。例如执行`telnet localhost 881`其中881为本机的未打开端口，会产生如下输出：

```
Trying 127.0.0.1...
telnet: connect to address 127.0.0.1: Connection refused
```

执行`telnet localhost 22`，其中22端口为本机的ssh服务端口且已经打开，会产生如下输出：

```
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
SSH-2.0-OpenSSH_5.3
```

则表示本机的22端口已经打开。

## netstat
查看本机网络端口命令，常用`netstat -aunp`。

#DNS相关

## host
```
kuring@ubuntu:~$ host www.google.com.hk
www.google.com.hk is an alias for www-wide.l.google.com.
www-wide.l.google.com has address 74.125.128.199
www-wide.l.google.com has IPv6 address 2404:6800:4005:c00::c7
```

## nslookup
```
kuring@ubuntu:~$ nslookup www.google.com.hk
Server:		127.0.0.1
Address:	127.0.0.1#53

Non-authoritative answer:
www.google.com.hk	canonical name = www-wide.l.google.com.
Name:	www-wide.l.google.com
Address: 74.125.128.199
```

## dig
可以代替nslookup的命令，显示的域名信息更为详细。

# 其他

# ab
Linux下的压力测试工具，可以模拟多个客户端发送多个请求。

```
kuring@ubuntu:~$ ab -c 100 -n 100 http://kuring.me/
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking kuring.me (be patient).....done


Server Software:        Tengine/2.0.0
Server Hostname:        kuring.me
Server Port:            80

Document Path:          /
Document Length:        707 bytes

Concurrency Level:      100
Time taken for tests:   1.547 seconds
Complete requests:      100
Failed requests:        10
   (Connect: 0, Receive: 0, Length: 10, Exceptions: 0)
Write errors:           0
Non-2xx responses:      90
Total transferred:      145330 bytes
HTML transferred:       126570 bytes
Requests per second:    64.64 [#/sec] (mean)
Time per request:       1546.990 [ms] (mean)
Time per request:       15.470 [ms] (mean, across all concurrent requests)
Transfer rate:          91.74 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:      173  223  43.8    200     289
Processing:   167  340 301.5    236    1368
Waiting:      166  339 300.5    235    1364
Total:        345  563 295.4    438    1546

Percentage of the requests served within a certain time (ms)
  50%    438
  66%    555
  75%    562
  80%    597
  90%   1108
  95%   1474
  98%   1484
  99%   1546
 100%   1546 (longest request)
```

# 参考

* [解决Linux服务器访问比较慢的问题-网络测试命令讲解](http://edu.51cto.com/lesson/id-12697.html)
* 《鸟哥的Linux私房菜-服务器架设篇》