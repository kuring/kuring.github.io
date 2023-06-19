---
title: ovs入门
date: 2020-02-10 23:52:22
tags:
---

控制器向上提供接口，用来供应用程序调用，此接口成为北向接口；控制器向下调用接口，控制网络设备，此接口成为南向接口。

OpenFlow是控制器和网络设备之间互通的南向协议，OpenvSwitch 用于创建软件的虚拟交换机。

## 原理

![https://static001.geekbang.org/resource/image/d8/14/d870e5bfcad8ec45d146c3226cdccb14.jpg](https://kuring.oss-cn-beijing.aliyuncs.com/common/ovs.jpg)

用户态进程

- ovsdb：本地数据库，存储ovs的配置信息
- vswitchd：ovs-ofctl用来跟该命令通讯，下发流表规则

## install OpenvSwitch on CentOS 7

```
# 安装依赖
yum install openssl-devel  python-sphinx gcc make python-devel openssl-devel kernel-devel graphviz kernel-debug-devel autoconf automake rpm-build redhat-rpm-config libtool python-twisted-core python-zope-interface PyQt4 desktop-file-utils libcap-ng-devel groff checkpolicy selinux-policy-devel gcc-c++ python-six unbound unbound-devel -y

mkdir -p ~/rpmbuild/SOURCES && cd ~/rpmbuild/SOURCES
# 下载ovs源码
wget https://www.openvswitch.org/releases/openvswitch-2.12.0.tar.gz

tar zvxf openvswitch-2.12.0.tar.gz
# 构建rpm包
rpmbuild -bb --nocheck openvswitch-2.12.0/rhel/openvswitch-fedora.spec

# 安装rpm包
yum localinstall /root/rpmbuild/RPMS/x86_64/openvswitch-2.12.0-1.el7.x86_64.rpm

systemctl start openvswitch.service
```

在编译的时候有如下报错：

```
  File "/usr/lib64/python2.7/site-packages/jinja2/sandbox.py", line 22, in <module>
    from markupsafe import EscapeFormatter
ImportError: cannot import name EscapeFormatter
```

是因为markupsafe的版本不对导致的，解决方法为安装合适的版本：

```
pip uninstall markupsafe
pip install markupsafe==0.23
```

## vlan实验

```
# 创建虚拟交换机ovs_br
ovs-vsctl add-br ovs_br
ovs-vsctl add-port ovs_br first_port
```

## flow table试验 1

```
# 创建namespace
ip netns add ns1
ip netns add ns2

# 创建veth pair设备
ip link add veth1 type veth peer name veth1_br
ip link add veth2 type veth peer name veth2_br

# 设置veth pair设备的namespace
ip link set veth1 netns ns1
ip link set veth2 netns ns2

# 创建OVS网桥
ovs-vsctl add-br ovs1

# 将veth pair设备另一端绑到网桥ovs1
ovs-vsctl add-port ovs1 veth1_br
ovs-vsctl add-port ovs1 veth2_br

# 启动veth pair
ip netns exec ns1 ip link set veth1 up
ip netns exec ns2 ip link set veth2 up
ip link set veth1_br up
ip link set veth2_br up

# 设置veth1和veth2的ip地址
ip netns exec ns1 ip addr add 192.168.1.100 dev veth1
ip netns exec ns2 ip addr add 192.168.1.200 dev veth2

# 配置路由
ip netns exec ns1 route add -net 192.168.1.0 netmask 255.255.255.0 dev veth1
ip netns exec ns2 route add -net 192.168.1.0 netmask 255.255.255.0 dev veth2
```

可以使用如下命令来查看刚才的操作：

```
# 可以看到刚才创建的网桥
$ ovs-vsctl list-br
ovs1

# 查看网桥的端口
$ ovs-vsctl list-ports ovs1
veth1_br
veth2_br

# 查看网桥的状态
$ ovs-vsctl show
4ec35070-a763-4748-878a-c3784b5938a4
    Bridge "ovs1"
        Port "veth1_br"
            Interface "veth1_br"
        Port "ovs1"
            Interface "ovs1"
                type: internal
        Port "veth2_br"
            Interface "veth2_br"
    ovs_version: "2.12.0"

# 查看interface的状态，跟port是一一对应的
$ ovs-vsctl list interface veth1_br
...
mac                 : []
mac_in_use          : "32:13:6b:99:91:2b"
mtu                 : 1500
mtu_request         : []
name                : "veth1_br"
ofport              : 1 # ovs port编号
...
```

接下来测试一下网络的连通性是没问题的。

```
ip netns exec ns1 ping 192.168.1.200
```

查看当前流表，可以看到有一条默认的规则，该条规则用来实现交换机的基本动作。

```
$ ovs-ofctl dump-flows ovs1
 cookie=0x0, duration=1428.955s, table=0, n_packets=22, n_bytes=1676, priority=0 actions=NORMAL
```

将上述规则删除，再执行ping命令发现已经不通。说明该默认规则会将流量在端口之间进行转发。

```
ovs-ofctl del-flows ovs1
```

新增加如下两条规则，用来表示将port 1的流量转发到port 3，将port 3的流量转发到port 1。其中的1和3分别为port编号，使用`ovs-vsctl list interface veth1_br`命令中的ofport可以看到。

```
ovs-ofctl add-flow ovs1 "priority=1,in_port=1,actions=output:3"
ovs-ofctl add-flow ovs1 "priority=2,in_port=3,actions=output:1"

$ ovs-ofctl dump-flows ovs1
 cookie=0x0, duration=69.378s, table=0, n_packets=4, n_bytes=280, priority=1,in_port="veth1_br" actions=output:"veth2_br"
 cookie=0x0, duration=69.063s, table=0, n_packets=4, n_bytes=280, priority=2,in_port="veth2_br" actions=output:"veth1_br"
```

再执行ping命令，发现可以ping通了。

重新增加一条优先级更高的规则，将port 1的数据drop掉。此时再ping发现已经不通了。

```
ovs-ofctl add-flow ovs1 "priority=3,in_port=1,actions=drop"
```

### 多table

接下来清理掉规则，并将规则重新写入到table1中，默认规则是写入到table0中的

```
ovs-ofctl del-flows ovs1
ovs-ofctl add-flow ovs1 "table=1,priority=1,in_port=1,actions=output:3"
ovs-ofctl add-flow ovs1 "table=1,priority=2,in_port=3,actions=output:1"
```

此时再执行ping命令，发现网络是不通的。因为table0中没有匹配成功，包被drop掉了。

再增加如下规则，即将table 0的规则发送到table 1处理，此时可以ping通。

```
ovs-ofctl add-flow ovs1 "table=0,actions=goto_table=1"
```

### group table

执行`ovs-ofctl del-flows ovs1`重新清理掉规则，执行下面命令查看group table内容，可以看到内容为空。

```
# ovs-ofctl -O OpenFlow13 dump-groups ovs1
OFPST_GROUP_DESC reply (OF1.3) (xid=0x2):
```

执行如下命令，完成数据包从table0 -> group table -> table1的过程，真正数据处理在table1中。

```
# 创建一个group table，其作用为将数据包发送到table 1
ovs-ofctl add-group ovs1 "group_id=1,type=select,bucket=resubmit(,1)"

# 将port 1和3 的数据发往group table 1
ovs-ofctl add-flow ovs1 "table=0,in_port=1,actions=group:1"
ovs-ofctl add-flow ovs1 "table=0,in_port=3,actions=group:1"

# table 1为真正要处理数据的逻辑
ovs-ofctl add-flow ovs1 "table=1,priority=1,in_port=1,actions=output:3"
ovs-ofctl add-flow ovs1 "table=1,priority=2,in_port=3,actions=output:1"
```

此时再执行ping命令，发现是可以ping通的。

### 清理操作

```
# 删除网桥
ovs-vsctl del-br ovs1
ip link delete veth1_br
ip link delete veth2_br
ip netns del ns1
ip netns del ns2
```

## 常用操作

- `ovs-appctl fdb/show ovs1`: 查看mac地址表
- `ovs-ofctl show ovs1`: 可以查看网桥的端口号
- `ovs-vsctl set bridge ovs1 stp_enable=false`: 开启网桥的生成树协议
- `ovs-appctl ofproto/trace ovs1 in_port=1,dl_dst=7a:42:0a:ca:04:65`: 可用来验证一个包到达网桥后的处理流程

## reference

- [CentOS 7 安装 Open vSwitch](https://zhuanlan.zhihu.com/p/63114462)
- [OpenvSwitch初探 - FLOW篇](https://zpzhou.com/archives/ovs_flow.html)
- [Open vSwitch (OVS) commands for troubleshooting](https://kb.juniper.net/InfoCenter/index?page=content&id=KB32283&actp=METADATA)
