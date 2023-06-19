---
title: docker基础知识之network namespace
date: 2018-09-15 00:40:49
tags: docker
---

network namespace用来隔离Linux系统的网络设备、ip地址、端口号、路由表、防火墙等网络资源。用户可以随意将虚拟网络设备分配到自定义的networknamespace里，而连接真实硬件的物理设备则只能放在系统的根networknamesapce中。

一个物理的网络设备最多存在于一个network namespace，可以通过创建veth pair在不同的network namespace之间创建通道，来达到通讯的目的。

容器的bridge模式的实现思路为创建一个veth pair，一端放置在新的namespace，通常命名为eth0，另外一端放在原先的namespace中连接物理网络设备，以此实现网络通信。

docker daemon负责在宿主机上创建veth pair，把一端绑定到docker0网桥，另一端到新建的network namespace进程中。建立的过程中，docker daemon和dockerinit通过pipe进行通讯。

## 一、测试例子

测试network namespace的过程比较复杂。

docker默认采用的为bridge模式，在容器所在的宿主机上看到的网卡情况如下：

```
[root@localhost software]# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:6c:3e:95 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:a5:78:ca brd ff:ff:ff:ff:ff:ff
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:a3:75:00:16 brd ff:ff:ff:ff:ff:ff
18: veth71f2650@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether ca:05:f7:db:6f:4c brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

其中的enp0s3和enp0s8可以忽略，为虚拟机使用的网卡。docker0和veth71f2650@if17是需要关注的网卡。

```
[root@localhost software]# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.0242a3750016	no		veth71f2650
```

下面的操作为在已经运行docker的虚拟机上的，以便于跟docker进行比较。

以下命令根据coolshell中的步骤进行配置，并对执行命令的顺序进行了调整。

```
# 增加network namespace ns1
[root@localhost software]# ip netns add ns1
[root@localhost software]# ip netns
ns1

# 激活namespace ns1中的lo设备
[root@localhost software]# ip netns exec ns1   ip link set dev lo up

# 创建veth pair
[root@localhost software]# ip link add veth-ns1 type veth peer name lxcbr0.1
# 多出了lxcbr0.1@veth-ns1和veth-ns1@lxcbr0.1两个设备
# 后面的操作步骤中将lxcbr0.1位于主网络命名空间中，veth-ns1位于ns1命名空间中
[root@localhost software]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:6c:3e:95 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:a5:78:ca brd ff:ff:ff:ff:ff:ff
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:a3:75:00:16 brd ff:ff:ff:ff:ff:ff
18: veth71f2650@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether ca:05:f7:db:6f:4c brd ff:ff:ff:ff:ff:ff link-netnsid 0
19: lxcbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether c6:b7:4d:7f:f8:90 brd ff:ff:ff:ff:ff:ff
20: lxcbr0.1@veth-ns1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether c6:8a:26:3d:ba:de brd ff:ff:ff:ff:ff:ff
21: veth-ns1@lxcbr0.1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether f2:03:22:93:d6:f4 brd ff:ff:ff:ff:ff:ff

# 将设备veth-ns1放入到ns1命名空间中
[root@localhost software]# ip link set veth-ns1 netns ns1
# 可以看到veth-ns1设备在当前命名空间消失了
[root@localhost software]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:6c:3e:95 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:a5:78:ca brd ff:ff:ff:ff:ff:ff
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:a3:75:00:16 brd ff:ff:ff:ff:ff:ff
18: veth71f2650@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default
    link/ether ca:05:f7:db:6f:4c brd ff:ff:ff:ff:ff:ff link-netnsid 0
19: lxcbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether c6:b7:4d:7f:f8:90 brd ff:ff:ff:ff:ff:ff
20: lxcbr0.1@if21: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether c6:8a:26:3d:ba:de brd ff:ff:ff:ff:ff:ff link-netnsid 1
# 同时在命名空间ns1中看到了设备veth-ns1，同时可以看到veth-ns1设备的状态为DOWN
[root@localhost software]# ip netns exec ns1  ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
21: veth-ns1@if20: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether f2:03:22:93:d6:f4 brd ff:ff:ff:ff:ff:ff link-netnsid 0

# 将ns1中的veth-ns1设备更名为eth0
[root@localhost software]# ip netns exec ns1  ip link set dev veth-ns1 name eth0
[root@localhost software]# ip netns exec ns1  ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
21: eth0@if20: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether f2:03:22:93:d6:f4 brd ff:ff:ff:ff:ff:ff link-netnsid 0

# 为容器中的网卡分配一个IP地址，并激活它
[root@localhost software]# ip netns exec ns1 ifconfig eth0 192.168.10.11/24 up
# 可以看到eth0网卡上有ip地址
[root@localhost software]# ip netns exec ns1  ifconfig
eth0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.10.11  netmask 255.255.255.0  broadcast 192.168.10.255
        ether f2:03:22:93:d6:f4  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# 添加一个网桥lxcbr0，类似于docker中的docker0
[root@localhost software]# brctl addbr lxcbr0
[root@localhost software]# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.0242a3750016	no		veth71f2650
lxcbr0		8000.000000000000	no

# 关闭生成树协议，默认该协议为关闭状态
[root@localhost software]# brctl stp lxcbr0 off
[root@localhost software]# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.0242a3750016	no		veth71f2650
lxcbr0		8000.000000000000	no

# 为网桥配置ip地址
ifconfig lxcbr0 192.168.10.1/24 up
[root@localhost software]# ifconfig lxcbr0
lxcbr0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.1  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::c4b7:4dff:fe7f:f890  prefixlen 64  scopeid 0x20<link>
        ether c6:b7:4d:7f:f8:90  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8  bytes 648 (648.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

# 将veth设备中的其中一个lxcbr0.1添加到网桥lxcbr0上
[root@localhost software]# brctl addif lxcbr0 lxcbr0.1
# 可以看到网桥lxcbr0中已经包含了设备lxcbr0.1
[root@localhost software]# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.0242a3750016	no		veth71f2650
lxcbr0		8000.c68a263dbade	no		lxcbr0.1

# 为网络空间ns1增加默认路由规则，出口为网桥ip地址
[root@localhost software]# ip netns exec ns1     ip route add default via 192.168.10.1
[root@localhost software]# ip netns exec ns1 ip route
default via 192.168.10.1 dev eth0
192.168.10.0/24 dev eth0 proto kernel scope link src 192.168.10.11

# 为ns1增加resolv.conf
[root@localhost software]# mkdir -p /etc/netns/ns1
[root@localhost software]# echo "nameserver 8.8.8.8" > /etc/netns/ns1/resolv.conf
```

## 二、常用命令

### 1. 列出当前的network namespace

#### 1.1 使用`lsns`命令

lsns命令通过读取/proc/${pid}/ns目录下进程所属的命名空间来实现，如果是通过`ip netns add`场景的命名空间，但是没有使用该命名空间的进程，该命令是看不到的。

```
# lsns -t net
        NS TYPE NPROCS   PID USER  COMMAND
4026531956 net     383     1 root  /usr/lib/systemd/systemd --switched-root --system --deserialize 21
4026532490 net       1  1026 rtkit /usr/libexec/rtkit-daemon
4026532762 net       2 24872 root  /pause
4026532866 net      20 25817 root  /pause
4026532965 net       3 30763 root  /pause
4026533059 net       3  2794 root  /bin/sh -c python /usr/src/app/clean.py "${endpoints}" "${expire}"
4026533163 net       2  1122 102   /docker-java-home/jre/bin/java -Xms2g -Xmx2g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupan
4026533266 net       4 13920 root  /pause
4026533371 net       2  1844 root  /pause
4026533559 net       3  1067 root  sleep 4
```

#### 1.2 通过`ip netns`命令

该命令仅会列出有名字的namespace，对于未命名的不能显示。

- `ip netns identify ${pid}` 可以找到进程所属的网络命名空间
- `ip netns list`: 显示所有有名字的namespace

### 2. 通过pid进入具体的network namespace

#### 2.1 通过nsenter命令

`nsenter --target $PID --net`可以进入到对应的命名空间

#### 2.2 docker `--net`参数

docker提供了`--net`参数用于加入另一个容器的网络命名空间`docker run -it --net container:7835490487c1 busybox ifconfig`。

#### 2.3 setns系统调用

一个进程可以通过setns()系统调用来进入到另外一个namespace中。

编写setns.c程序，该程序会进入到进程id所在的网络命令空间，并使用`gcc setns.c -o setns`进行编译，编译完成后执行`./setns /proc/4913/ns/net ifconfig`可以看到网卡的信息为容器中的网卡信息。

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>

int main(int argc, char *argv[]) {
  int fd = open(argv[1], O_RDONLY);
  if (setns(fd, 0) == -1) {
    perror("setns");
    exit(-1);
  }
  execvp(argv[2], &argv[2]);
  printf("execvp exit\n");
}
```

如果执行`./setns /proc/4913/ns/net /bin/bash`，在宿主机上查看docker进程和/bin/bash进程的网络命名空间/proc/${pid}/ns/net，会发现都指向`lrwxrwxrwx 1 root root 0 Sep 14 14:42 net -> net:[4026532133]`同一个位置。

### 3. pid的获取方式

最简单的方式上文第1点中的PID列

#### 3.1 `/proc/[pid]/ns`

可以使用如下命令查看当前容器在宿主机上的进程id。

```
docker inspect --format '{{.State.Pid}}' a1bf0119d891
```

每个进程在/proc/${pid}/ns/目录下都会创建其对应的虚拟文件，并链接到一个真实的namespace文件上，如果两个进程下的链接文件链接到同一个地方，说明两个进程同属于一个namespace。

```
[root@localhost runc]# ls -l /proc/4913/ns/
total 0
lrwxrwxrwx 1 root root 0 Sep 11 00:21 ipc -> ipc:[4026532130]
lrwxrwxrwx 1 root root 0 Sep 11 00:21 mnt -> mnt:[4026532128]
lrwxrwxrwx 1 root root 0 Sep 11 00:18 net -> net:[4026532133]
lrwxrwxrwx 1 root root 0 Sep 11 00:21 pid -> pid:[4026532131]
lrwxrwxrwx 1 root root 0 Sep 11 00:21 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0 Sep 11 00:21 uts -> uts:[4026532129]
```

## reference

* [DOCKER基础技术：LINUX NAMESPACE（下）](https://coolshell.cn/articles/17029.html)
* 极客时间-深入剖析Kubernetes
* [一文搞懂 Linux network namespace](https://www.cnblogs.com/bakari/p/10443484.html)
