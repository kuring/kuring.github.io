title: /etc/resolv.conf文件
date: 2022-06-16 21:57:43
tags:
author:
---

/etc/resolv.conf文件为Linux主机的DNS配置文件，在 Linux 主机上可以执行 `man resolv.conf` 查看帮助信息。

## 配置文件说明

整个配置文件分为了4个部分：nameserver、search、sortlist和options

### nameserver

用来配置DNS服务器地址。支持ipv4和ipv6地址。如果要配置多个DNS服务器，可以增加多条配置，域名按照DNS服务器配置的顺序来解析。配置多条的例子如下：

```
nameserver 100.100.2.136
nameserver 100.100.2.138
```

### search

在默认情况下，如果要查找的域名中不包含domain信息（即不包含`.`字符），此时会以当前hostname的domain信息来进行查找。可以通过search字段来修改该行为，即可以通过在域名的后面增加配置的后缀来进行查找。

如果存在多条search记录，则以最后一条search记录为准。

比如k8s的pod中的search域格式如下，如果要查找defaultnamespace下的service的svc1，此时系统会自动追加 svc1.default.svc.cluster.local来进行查找：

```
search default.svc.cluster.local svc.cluster.local cluster.local
```

### sortlist

该字段极少场景下会用到。在做域名解析时，如果返回的A记录为多条，可以对结果进行排序，排序的依据为当前的字段配置。

```
sortlist 130.155.160.0/255.255.240.0 130.155.0.0
```

#### options

该配置必须加载一行中，格式如下，其中option部分可以为多个，多个之间用空格分割：

```
options option ...
```

- inet6: 应用程序在执行系统调用 gethostbyname 时，会优先执行 AAAA 记录的查询。如果查询不到 ipv6 地址，再去查询 ipv4 地址。

## 实验

### 实验一

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

### 实验二

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

domain的作用跟search类似，作为search的默认值，因为search可以使用多个域。

## 参考资料

- [resolv.conf(5) - Linux manual page (man7.org)](https://man7.org/linux/man-pages/man5/resolv.conf.5.html)
