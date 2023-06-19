---
title: Linux中的veth pair设备
date: 2020-01-29 21:21:06
tags:
---

veth pair是一对虚拟的网络设备，两个网络设备彼此连接。常用于两个network namespace之间的连接，如果在同一个命名空间下有很多的限制。

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│                                                                              │
│                               network protocol                               │
│                                                                              │
│                                                                              │
└────────────────────▲─────────────────────────▲──────────────────────▲────────┘
                     │                         │                      │
                     │                         │                      │
                     │                         │                      │
                     │                         │                      │
                     │                         │                      │
               ┌─────▼────┐              ┌─────▼────┐           ┌─────▼────┐
               │          │              │          │           │          │
               │   eth0   │              │  veth0   ◀───────────▶  veth1   │
               │          │              │          │           │          │
               └─────▲────┘              └──────────┘           └──────────┘
                     │
                     │
                     │
                     │
                     │
                     ▼

             physical network
```

## 实战

### veth设备的ping测试

### 1. 只给一个veth设备配置ip的情况测试

给veth0配置ip 192.168.100.10，可以看到主机的路由表中增加了目的地为192.168.100.0的记录

```
[root@localhost vagrant]# ip link add veth0 type veth peer name veth1
[root@localhost vagrant]# ip addr add 192.168.100.10/24 dev veth0
[root@localhost vagrant]# ip addr add 192.168.100.11/24 dev veth1
## 因为veth创建完后默认不启用，此时还没有路由
[root@localhost vagrant]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    100    0        0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.33.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1

## 启用veth0后增加路由
[root@localhost vagrant]# ip link set veth0 up
[root@localhost vagrant]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    100    0        0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.33.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 veth0

## 启用veth1后居然又增加了一条路由信息
[root@localhost vagrant]# ip link set veth1 up
[root@localhost vagrant]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:26:10:60 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 86214sec preferred_lft 86214sec
    inet6 fe80::5054:ff:fe26:1060/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:98:06:20 brd ff:ff:ff:ff:ff:ff
    inet 192.168.33.11/24 brd 192.168.33.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe98:620/64 scope link
       valid_lft forever preferred_lft forever
4: veth1@veth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether e2:15:95:0a:1f:da brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.11/24 scope global veth1
       valid_lft forever preferred_lft forever
    inet6 fe80::e015:95ff:fe0a:1fda/64 scope link
       valid_lft forever preferred_lft forever
5: veth0@veth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether b2:2c:f6:e4:74:c5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.10/24 scope global veth0
       valid_lft forever preferred_lft forever
    inet6 fe80::b02c:f6ff:fee4:74c5/64 scope link
       valid_lft forever preferred_lft forever
[root@localhost vagrant]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    100    0        0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.33.0    0.0.0.0         255.255.255.0   U     101    0        0 eth1
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 veth0
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 veth1
```

默认情况下arp表如下：

```
# arp
Address                  HWtype  HWaddress           Flags Mask            Iface
localhost.localdomain            (incomplete)                              veth0
192.168.33.1             ether   0a:00:27:00:00:00   C                     eth1
gateway                  ether   52:54:00:12:35:02   C                     eth0
10.0.2.3                 ether   52:54:00:12:35:03   C                     eth0
```

使用ping命令`ping -I veth0 192.168.100.11 -c 2`，默认情况下veth1和veth0会接收到arp报文，但并没有arp的响应报文。这是因为默认情况下有些arp内核参数的限制。执行如下命令解决arp的限制。

```
echo 1 > /proc/sys/net/ipv4/conf/veth1/accept_local
echo 1 > /proc/sys/net/ipv4/conf/veth0/accept_local
echo 0 > /proc/sys/net/ipv4/conf/all/rp_filter
echo 0 > /proc/sys/net/ipv4/conf/veth0/rp_filter
echo 0 > /proc/sys/net/ipv4/conf/veth1/rp_filter
```

## veth pair设备的删除

```
# 删除veth0后会自动删除veth1
$ ip link delete veth0
```

## container与host veth pair的关系

veth pair的其中一个设备位于container中备位于container中，另外一个设备位于host network namespace中，如何知道container中的eth0和host network namesapce中的veth设备的对应关系呢？

原理为veth pair设备都有一个ifindex和iflink值，，容器中的eth0设备的ifindex值跟host network namespace中的对应veth pair设备的iflink值相等，反之亦然。

### 在容器中找到eth0的iflink

方法一

获取iflink值：`cat /sys/class/net/eth0/iflink` 

也可用此方法获取ifindex值：`cat /sys/class/net/eth0/ifindex` 

方法二

```
$ ip link show eth0
3: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 96:5f:80:a3:a3:01 brd ff:ff:ff:ff:ff:ff
```

其中的3为eth0的ifindex。18为eth0的iflink，即对应的veth pair的另外一个设备的ifindex。

### host network namespace中找到对应ifindex值的veth pair设备

```
$ ip addr 
18: veth0e09999e@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br0 state UP group default
    link/ether de:b0:74:89:e8:3e brd ff:ff:ff:ff:ff:ff link-netnsid 4
    inet6 fe80::dcb0:74ff:fe89:e83e/64 scope link
       valid_lft forever preferred_lft forever
```

其中的18为ifindex，3为对应的veth pair的ifindex。

## reference

- [Linux 虚拟网络设备 veth-pair 详解，看这一篇就够了](https://www.cnblogs.com/bakari/p/10613710.html)
- [Linux虚拟网络设备之veth](https://segmentfault.com/a/1190000009251098)

