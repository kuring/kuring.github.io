---
title: docker bridge network
date: 2020-02-01 21:34:53
tags:
---

docker bridge是默认的网络模式

## 容器访问外网

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/docker_bridge_network.png)

默认情况下，容器即可访问外网。

启动一个容器：`docker run -d nginx`

容器中访问外网的请求如www.baidu.com，内核协议栈根据路由信息，会选择默认路由，将请求发送到容器中的eth0网卡，目的mac地址为网关172.17.0.1的mac地址。

```
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
```

eth0网卡接收到数据包后，会将数据包转发到veth pair的另外一端，即宿主机网络中的veth6b173fd设备。

veth6b173fd设备是挂在网桥上的，会将数据包转发到网桥br0，br0即为网关172.17.0.1。

br0接收到数据包后，会将数据包转发给内核协议栈。

宿主机上的/proc/sys/net/ipv4/ip_forward为1，表示转发功能开启，即目的ip不是本机的会根据路由规则进行转发，而不是丢弃。

仅在宿主机上开启了ip_forward后，包即使转发了，还是无法回来的，因为包中的源ip地址为172.17.0.1，是私有网段的ip地址。需要做一次SNAT才可以，docker会在iptabels的nat表中的postrouting链中增加SNAT规则，下面规则的意思是源地址为172.17.0.0/16的会做一次地址伪装，即SNAT。

```
# iptables -nL -t nat

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0
```

在eth0网卡上抓包，可以发现源ip已经是eth0的网卡ip地址。

## 端口映射

命令格式：`-p ${host_port}:${container_port}`

启动一个容器：`docker run -d -p 8080:80 nginx`

查看本地的iptables规则

```
$ iptables -nL -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0
MASQUERADE  tcp  --  172.17.0.2           172.17.0.2           tcp dpt:80

Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
## prerouting链引用，外面发往本机的8080端口的数据包，会dnat为172.17.0.2:80
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 to:172.17.0.2:80
```

## 丢包问题

SNAT在并发比较高的情况下，会存在少量的丢包现象，具体原因跟conntrack模式的实现有关。conntrack在SNAT端口的分配和插入conntrack表之间有个延时，如果在这中间存在冲突的话会导致插入失败，从而出现丢包的问题。

该问题没有根治的解决办法，能大大缓解的解决办法为使用iptabels的--random-fully选项，SNAT选择端口为随机，大大降低出现冲突的概率。
