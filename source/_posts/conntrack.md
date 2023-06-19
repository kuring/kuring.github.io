---
title: conntrack介绍
date: 2020-02-13 23:51:16
tags:
---

<img src="https://kuring.oss-cn-beijing.aliyuncs.com/common/Netfilter-packet-flow.svg">

conntrack是netfilter提供的连接跟踪机制，允许内核识别出数据包属于哪个连接。是iptables实现状态匹配(-m state)以及nat的基础，由单独的内核模块nf_conntrack实现。

conntrack在图中有两处，一处位于prerouting，一处位于output。主机自身进程产生的数据包会经过output链的conntrack，主机的网络设备接收到的数据包会通过prerouting链的conntrack。

每个通过conntrack的数据包，内核会判断是否为新的连接。如果是新的连接，则在连接跟踪表中插入一条记录。如果是已有连接，会更新连接跟踪表中的记录。

需要特别注意的是，conntrack并不会修改数据包，如dnat、snat，而仅仅是维护连接跟踪表。

## 连接跟踪表的内容

`/proc/net/nf_conntrack`可以看到连接跟踪表的所有内容，通过hash表来实现。

```
ipv4     2 tcp      6 117 TIME_WAIT src=10.45.4.124 dst=10.45.8.10 sport=36903 dport=8080 src=10.45.8.10 dst=10.45.4.124 sport=8080 dport=36903 [ASSURED] mark=0 zone=0 use=2
```

- 117: 该连接的生存时间，每个连接都有一个timeout值，可以通过内核参数进行设置，如果超过该时间还没有报文到达，该连接将会删除。如果有新的数据到达，该计数会被重置。
- TIME_WAIT: 当前该连接的最新状态

## 涉及到的内核参数

- `net.netfilter.nf_conntrack_buckets`: 用来设置hash表的大小
- `net.netfilter.nf_conntrack_max`: 用来设置连接跟踪表的数据条数上限

## iptables与conntrack的关系

iptables使用`-m state`模块来从连接跟踪表查找数据包的状态，上面例子中的`TIME_WAIT`即为连接跟踪表中的状态，但这些状态对应到iptable中就只有五种状态。特别需要注意的是，这五种状态是跟具体的协议是tcp、udp无关的。

| 状态 | 含义 |
| ---- | ---- | 
| NEW | 匹配连接的第一个包 |
| ESTABLISHED | NEW状态后，如果对端有回复包，此时连接状态为NEW |
| RELATED | 不是太好理解，当已经有一个状态为ESTABLISHED连接后，如果又产生了一个新的连接并且跟此时关联的，那么该连接就是RELATED状态的。对于ftp协议而言，有控制连接和数据连接，控制连接要先建立为ESTABLISHED，数据连接就变为控制连接的RELATED。那么conntrack怎么能够识别到两个连接是有关联的呢，即能够识别出协议相关的内容，这就需要通过扩展模块来完成了，比如ftp就需要nf_conntrack_ftp |
| INVALID | 无法识别的或者有状态的数据包 |
| UNTRACKED | 匹配带有NOTRACK标签的数据包，raw表可以将数据包标记为NOTRACK，这种数据包的连接状态为NOTRACK |

## conntrack-tools

```
# 查看连接跟踪表
conntrack -L
```

## reference

- [云计算底层技术-netfilter框架研究](https://opengers.github.io/openstack/openstack-base-netfilter-framework-overview/)

