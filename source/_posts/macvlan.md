---
title: Linux macvlan network
date: 2018-08-13 23:29:58
tags: docker macvlan
---

macvlan的原理是在宿主机物理网卡上虚拟出多个子接口，每个子接口有独立的mac地址，通过不同的MAC地址在数据链路层（Data Link Layer）进行网络数据转发的。达到的效果类似，一块物理网卡上有多个IP地址，多个IP地址有自己的mac地址。

它是比较新的网络虚拟化技术，需要较新的内核支持（Linux kernel v3.9–3.19 and 4.0+）。

macvlan设备跟物理设备之间并不直接互通。

macvlan不以交换机端口来划分vlan，一个交换机端口可接收来自多个mac地址的数据。

一个交换机端口要处理多个vlan的数据，需要开启trunk模式。

## 四种模式

以下四种模式为每个macvlan设备单独配置，而不是一个物理设备就只有一个配置。

### VEPA

所有发送出去的报文都经过交换机，交换机再发送到对应的目标地址。默认模式。物理网卡接收到macvlan设备的数据后，总是将数据发送出去，即使是发往本设备上其他macvlan设备的数据包。这样在交换机设备上可以看到所有网络的流量。如果是本机的macvlan设备流量仍然是发往本机的macvlan设备，可能会被交换机的生成树协议阻止。需要交换机开启hairpin模式或者reflective relay模式，该模式在目前的交换机上未广泛支持，vepa模式的应用较少。

linux的网桥支持hairpin模式。

### bridge

最常用，同一个物理设备上的不同macvlan设备间的通讯可以直接转发，不再需要经过外部的交换机。转发非常快速，macvlan设备相对固定，不需要生成树协议。

### Private

本质上是VEPA模式，但同一个物理设备上的macvlan设备之间无法直接通讯，不常用。

### Passthru

后来增加的模式，比较少用

vepa和passthru都会将不同macvlan接口之间的数据发送到交换机，然后发回，对性能的影响比较明显。

物理网卡收到包后，根据包的mac地址来判断这个包交给哪个虚拟接口。

![image](https://kuring.oss-cn-beijing.aliyuncs.com/images/macvlan-workmode-1.png)

## 手工创建实践

以下实验为在virturalbox虚拟机下

```
# 创建两个network namespace net1和net2
ip netns add net1
ip netns add net2

# 创建macvlan接口
# enp0s8相当于物理网卡eth0
ip link add link enp0s8 mac1 type macvlan

# 可以看到创建了接口mac1@enp0s8
[root@localhost vagrant]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
    link/ether 08:00:27:6c:3e:95 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
    link/ether 08:00:27:cf:b0:b4 brd ff:ff:ff:ff:ff:ff
4: docker_gwbridge: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT
    link/ether 02:42:8e:cf:a9:da brd ff:ff:ff:ff:ff:ff
5: br-b3e83aa45886: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT
    link/ether 02:42:bf:b4:e5:36 brd ff:ff:ff:ff:ff:ff
6: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT
    link/ether 02:42:25:e7:40:32 brd ff:ff:ff:ff:ff:ff
19: br-ec6c4e77321d: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT
    link/ether 02:42:e0:d8:38:6d brd ff:ff:ff:ff:ff:ff
24: mac1@enp0s8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT
    link/ether 3e:b8:e9:1d:3b:c8 brd ff:ff:ff:ff:ff:ff
```

创建macvlan接口的格式为：`ip link add link <PARENT> <NAME> type macvlan`， <PARENT>是macvlan接口的父接口名称，name是新创建的macvlan接口名称。

```
# 将mac1放入到net1 namespace中
[root@localhost vagrant]# ip link set mac1 netns net1
[root@localhost vagrant]# ip netns exec net1 ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
24: mac1@if3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT
    link/ether 3e:b8:e9:1d:3b:c8 brd ff:ff:ff:ff:ff:ff link-netnsid 0

# 在net1中将mac1接口命名为eth0
[root@localhost vagrant]# ip netns exec net1 ip link set mac1 name eth0
[root@localhost vagrant]# ip netns exec net1 ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
24: eth0@if3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT
    link/ether 3e:b8:e9:1d:3b:c8 brd ff:ff:ff:ff:ff:ff link-netnsid 0

# 在net1中分配eth0网卡的ip地址为192.168.8.120
[root@localhost vagrant]# ip netns exec net1 ip addr add 192.168.8.120/24 dev eth0
[root@localhost vagrant]# ip netns exec net1 ip link set eth0 up
[root@localhost vagrant]# ip netns exec net1 ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
24: eth0@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT
    link/ether 3e:b8:e9:1d:3b:c8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

## 在docker上的实践

```
[root@localhost vagrant]# docker network create -d macvlan --subnet=10.0.2.100/24 --gateway=10.0.2.2 -o parent=enp0s3 mcv
5c637798d559471bd8d1036cdd947d3949e1973f724568da066d9c60c00fb5e6

[root@localhost vagrant]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
b6a128f1730e        bridge              bridge              local
a65957cd6c3f        docker_gwbridge     bridge              local
540bb390028a        host                host                local
b3e83aa45886        isolated_nw         bridge              local
ec6c4e77321d        local_alias         bridge              local
5c637798d559        mcv                 macvlan             local
3d247d0414d0        none                null                local
```

## reference

[Some notes on macvlan/macvtap](https://backreference.org/2014/03/20/some-notes-on-macvlanmacvtap/)
[](http://backreference.org/2014/03/20/some-notes-on-macvlanmacvtap/)
