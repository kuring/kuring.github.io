title: Linux内核参数（持续更新）
date: 2022-04-01 14:31:38
tags:
author:
---
# 内核参数项
以CentOS7 系统为例，可以看到有1088个内核参数项。
```json
$ uname -a
Linux iZbp17o12gcsq2d87y7x1hZ 3.10.0-1160.36.2.el7.x86_64 #1 SMP Wed Jul 21 11:57:15 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
$ sysctl -a 2>/dev/null | wc -l
1088
```
Linux的内核参数均位于/proc/sys目录下，涉及到如下几个目录：

| 分类 | 描述 |
| --- | --- |
| abi |  |
| crypto |  |
| debug |  |
| dev | 用来配置特定设备，比如raid、scsi设备 |
| fs | 文件子系统 |
| kernel | 内核子系统 |
| net | 网络子系统 |
| user |  |
| vm | 内存子系统 |

## kernel子系统

| 参数 | 描述 |
| --- | --- |
| kernel.panic | 内核出现panic，重新引导前需要等待的时间，单位为秒。如果该值为0，，说明内核禁止自动引导 |
| kernel.core_pattern | core文件的存放路径 |

## vm子系统
| 参数 | 描述 |
| --- | --- |
| vm.min_free_kbytes | 系统所保留空闲内存的最小值。该值通过公式计算，跟当前机器的物理内存相关。 |
| vm.swappiness | 用来控制虚拟内存，支持如下值：<br> 0：关闭虚拟内存 <br> 1：允许开启虚拟内存设置的最小值 <br> 10：剩余内存少于10%时开启虚拟内存 <br> 100：完全适用虚拟内存。<br><br> 该参数可以通过swapon 命令开启，swapoff关闭。<br>参考链接：[https://linuxhint.com/understanding_vm_swappiness/](https://linuxhint.com/understanding_vm_swappiness/)

## net子系统

| 参数 | 描述 |
| --- | --- |
| net.ipv4.ip_local_reserved_ports | 随机端口的黑名单列表，系统在发起连接时，不使用该内核参数内的端口号 |
| net.ipv4.ip_local_port_range | 随机端口的白名单范围，网络连接可以作为源端口的最小和最大端口限制 |
| net.ipv4.rp_filter | 是否开启对数据包源地址的校验, 收到包后根据source ip到route表中检查是否否和最佳路由，否的话扔掉这个包。这次如下值：<br>1. 不开启源地址校验<br> 2. 开启严格的反向路径校验。对每个进来的数据包，校验其反向路径是否是最佳路径。如果反向路径不是最佳路径，则直接丢弃该数据包。<br> 3. 开启松散的反向路径校验。对每个进来的数据包，校验其源地址是否可达，即反向路径是否能通（通过任意网口），如果反向路径不通，则直接丢弃该数据包。该内核参数 net.ipv4.conf.all.log_martians 可以来控制是否打开日志，日志打开后可以在/var/log/message中观察到。 |

### TCP

tcp 相关内核参数可以使用 `man 7 tcp` 查看。

#### 建连相关

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/tcp_connect.webp)

相关文章：[Linux TCP backlog](/post/linux-backlog)

##### syn 队列

syn 队列又称为半连接队列。服务端在接收到客户端的 SYN 包后，服务端向客户端发送 SYN + ACK 报文，此时会进入到半连接队列。

1. net.ipv4.tcp_max_syn_backlog

```
tcp_max_syn_backlog (integer; default: see below; since Linux 2.2)
              The  maximum number of queued connection requests which have still not received an acknowledgement from the connecting client.  If this number is exceeded, the kernel will begin dropping requests.  The default value of 256 is increased to 1024 when the memory present in the system is adequate or greater (>= 128Mb), and  reduced  to 128  for  those systems with very low memory (<= 32Mb).  It is recommended that if this needs to be increased above 1024, TCP_SYNQ_HSIZE in include/net/tcp.h be modified to keep TCP_SYNQ_HSIZE*16<=tcp_max_syn_backlog, and the kernel be recompiled.
```

用来设置 syn 队列的大小，通常也会称为半连接队列。该参数的默认值一般为 1024。如果 syn 队列满，此时 syn 报文会被丢弃，无法回复 syn + ack 报文。可以通过 `netstat -s` 命令看到 "XX SYNs to LISTEN sockets dropped". 的报错信息。

2. net.ipv4.tcp_syncookies

```
tcp_syncookies (Boolean; since Linux 2.2) Enable TCP syncookies. The kernel must be compiled with CONFIG_SYN_COOKIES.  Send out syncookies when the syn backlog queue of a socket overflows.  The syncookies feature attempts to protect a socket from a SYN flood attack.  This should be used as a last resort, if at all.  This is a violation of the  TCP  protocol,  and  con‐ flicts  with other areas of TCP such as TCP extensions.  It can cause problems for clients and relays.  It is not recommended as a tuning mechanism for heavily loaded servers to help with overloaded or misconfigured conditions.  For recommended alternatives see tcp_max_syn_backlog, tcp_synack_retries, and tcp_abort_on_overflow.
```

因为 syn 队列的存在，当客户端一直在发送 syn 包，但是不回 ack 报文时，一旦服务端的队列超过 `net.ipv4.tcp_max_syn_backlog` 设置的大小就会存在队列溢出的问题，从而导致服务端无法响应客户端的请求，这就是 syn flood 攻击。

为了防止 syn flood 攻击，引入了 syn cookies 机制，该机制并非 tcp 协议的一部分。原理参见：[深入浅出TCP中的SYN-Cookies](https://segmentfault.com/a/1190000019292140)

一旦开启了 syn cookies 机制后，即使 syn 队列满，仍可以对新建的连接回复 syn + ack 报文，但是不需要进入队列。

因为 syn cookies 存在部分缺陷，只有当 syn 队列满时该特性才会生效。

3. net.ipv4.tcp_abort_on_overflow

在三次握手完成后，该连接会进入到 ESTABLISHED 状态，并将该连接放入到用户程序队列中。若该队列已满，默认会将该连接重新设置为 SYN_ACK 状态，相当于是服务端没有接收到客户端的 syn + ack 报文，后续可以利用客户端的重传机制重新接收报文。

一旦开启了 `net.ipv4.tcp_abort_on_overflow` 选项后，会直接发送 RST 报文给到客户端，客户端会终止该连接，并报错 `104 Connection reset by peer`。

#### 断开连接相关

[TCP TIME_WAIT](/post/time-wait)

## 内核参数在k8s的支持情况
| 大类 | 子类 | 备注 |
| --- | --- | --- |
| namespace内核参数 | 安全的内核参数 | k8s默认支持的内核参数非常少，仅支持如下的内核参数：<br> 1. kernel.shm_rmid_forced<br> 2. net.ipv4.ip_local_port_range<br> 3. net.ipv4.tcp_syncookies（在内核4.4之前为非namespace化）<br> 4. net.ipv4.ping_group_range （从 Kubernetes 1.18 开始）<br> 5. net.ipv4.ip_unprivileged_port_start （从 Kubernetes 1.22 开始）
 |
| namespace内核参数 | 非安全内核参数  | 默认禁用，pod可以调度成功，但会报错SysctlForbidden。修改kubelet参数开启 `kubelet --allowed-unsafe-sysctls 'kernel.msg*,net.core.somaxconn'` |
| 非namespace内核参数 | 在容器中没有的内核参数 |如：<br>net.core.netdev_max_backlog = 10000<br>net.core.rmem_max = 2097152<br>net.core.wmem_max = 2097152 |
| 非namespace隔离参数 | 直接修改宿主机的内核参数 | 在容器中需要开启特权容器来设置，如：<br>vm.overcommit_memory = 2 <br>vm.overcommit_ratio = 95 |

操作系统的namespace化的内核参数仅支持：
- kernel.shm*,
- kernel.msg*,
- kernel.sem,
- fs.mqueue.*,
- net.*（内核中可以在容器命名空间里被更改的网络配置项相关参数）。然而也有一些特例 （例如，net.netfilter.nf_conntrack_max 和 net.netfilter.nf_conntrack_expect_max 可以在容器命名空间里被更改，但它们是非命名空间的）。

k8s在pod中声明内核参数的方式如下：
```yaml
apiVersion: v1
kind: Pod
metadata:   
  name: sysctl-example
spec:   
  securityContext:     
    sysctls:     
    - name:
      kernel.shm_rmid_forced
      value: "0"
```


# 业界解决方案
## ACK - 安全沙箱容器
阿里云ACK服务的安全沙箱容器，底层实现为runV，pod拥有独立的内核参数，相互之间不受影响。通过扩展pod的annotation来完成内核参数的修改：
```json
annotations:
  securecontainer.alibabacloud.com/sysctls: "net.bridge.bridge-nf-call-ip6tables=1,net.bridge.bridge-nf-call-iptables=1,net.ipv4.ip_forward=1"
```
[阿里云ACK - 配置安全沙箱Pod内核参数](https://help.aliyun.com/document_detail/198645.html)

## 弹性容器实例
完全利用k8s的功能，有限内核参数修改。
[https://help.aliyun.com/document_detail/163023.html](https://help.aliyun.com/document_detail/163023.html)

# 参考文档

- [ASI 配置Security Context](https://help.aliyun.com/document_detail/163023.html)
