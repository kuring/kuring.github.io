---
title: Linux网络接口特性
date: 2020-08-04 00:48:45
tags:
---

## MTU

MTU是指一个以太网帧能够携带的最大数据部分的大小，并不包含以太网的头部部分。一般情况下MTU的值为1500字节。

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/mtu1.png)

当指定的数据包大小超过MTU值时，ip层会根据当前的mtu值对超过数据包进行分片，并会设置ip层头部的More Fragments标志位，并会设置Fragment offset属性，即分片的第二个以及后续的数据包会增加offset，第一个数据包的offset值为0。接收方会根据ip头部的More Fragment标志位和Fragment offset属性来进行切片的重组。

如果手工将发送方的MTU值设置为较大值，比如9000（巨型帧），如果发送方设置了不分片（ip头部的Don't fragment），此时如果发送的链路上有地方不支持该MTU，报文就会被丢弃。

## offload特性

执行 `ethtool -k ${device}` 可以看到很多跟网络接口相关的特性，这些特性的目的是为了提升网络的收发性能。TSO、UFO和GSO是对应网络发送，LRO、GRO对应网络接收。

执行`ethtool -K ${device} gro off/on` 来开启或者关闭相关的特性。

### LRO(Large Receive Offload)

通过将接收的多个tcp segment聚合为一个大的tcp包，然后传送给网络协议栈处理，以减少上层网络协议栈的处理开销。

但由于tcp segment并不是在同一时刻到达网卡，因此组装起来就会变得比较困难。

由于LRO的一些局限性，在最新的网络上，该功能已经删除。

### GRO(Generic Receive Offload)

GRO是LRO的升级版，正在逐渐取代LRO。运行与内核态，不再依赖于硬件。

## RSS hash 特性

网卡可以根据数据包放到不同的网卡队列来处理，并可以根据不同的数据协议来设置不同的值。

注意：该特性并非所有的网卡都支持

下面命令为查询 udp 协议的设置，可以看到 hash 的策略为根据源 ip 地址和目的 ip 地址。

```
$ ethtool -n eth0 rx-flow-hash udp4
UDP over IPV4 flows use these fields for computing Hash flow key:
IP SA
IP DA
```

可以使用 `ethtool -N eth0 rx-flow-hash udp4 sdfn`来修改hash 策略，`sdfn`对应的含义如下：

```
m   Hash on the Layer 2 destination address of the rx packet.
v   Hash on the VLAN tag of the rx packet.
t   Hash on the Layer 3 protocol field of the rx packet.
s   Hash on the IP source address of the rx packet.
d   Hash on the IP destination address of the rx packet.
f   Hash on bytes 0 and 1 of the Layer 4 header of the rx packet.
n   Hash on bytes 2 and 3 of the Layer 4 header of the rx packet.
r   Discard all packets of this flow type. When  this  option  is
    set, all other options are ignored.
```

修改完成后再查看网卡的 hash 策略如下：

```
$ ethtool -n eth0 rx-flow-hash udp4
UDP over IPV4 flows use these fields for computing Hash flow key:
IP SA
IP DA
L4 bytes 0 & 1 [TCP/UDP src port]
L4 bytes 2 & 3 [TCP/UDP dst port]
```



## 参考文章

- [关于MTU，这里也许有你不知道的地方](https://segmentfault.com/a/1190000019206098)
- [常见网络加速技术浅谈](https://zhuanlan.zhihu.com/p/44683790)
