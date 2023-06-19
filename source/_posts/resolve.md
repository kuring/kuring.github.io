---
title: resolve.conf配置文件解析
date: 2018-04-24 20:55:32
tags:
---

一直以来对/etc/resolv.conf配置文件中的search和domain字段的含义不是很理解，这里重新学习记录一下。

# 实验一

修改/resolv.conf配置文件的内容如下：

```
nameserver 8.8.8.8
```

可以看到该nameserver是生效的，但是访问map域名是不生效的，因为没有map这个域名.

```
[vagrant@localhost ~]$ ping map.baidu.com
PING map.n.shifen.com (119.75.222.71) 56(84) bytes of data.
64 bytes from 119.75.222.71: icmp_seq=1 ttl=63 time=4.22 ms

[vagrant@localhost ~]$ ping map
ping: unknown host map
```

# 实验二

修改文件内容如下：

```
nameserver 8.8.8.8

search baidu.com google.com
```
此时可以ping通map域名，解析到了跟map.baidu.com相同的域名.如果map.baidu.com的域名没有解析到，会继续解析map.google.com的域名。

```
[vagrant@localhost ~]$ ping map
PING map.n.shifen.com (112.80.248.48) 56(84) bytes of data.
64 bytes from 112.80.248.48: icmp_seq=1 ttl=63 time=28.6 ms
```

domain的作用跟search类似，作为search的默认值，因为search可以使用多个域.
