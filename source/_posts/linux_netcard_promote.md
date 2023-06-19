---
Status: public
url: linux_netcard_promote
tags: Linux
title: Linux下的网卡速度提升方案
date: 2014-06-14
---

[TOC]

本文研究了Linux下的一些网卡调优，用于提升网卡的性能。

下文中的所有关于网络的参数可以在/etc/sysctl.conf文件中修改，如果没有相应的参数，可以添加。

查看相应参数在当前运行机器的值可以通过/proc/sys/net/目录下的文件内容查看，也可以对该目录下相应文件的值进行修改，但是由于/proc目录下的文件全部位于内存中，修改的值不会保存到下次开机时。因此要修改参数的值可以通过修改/etc/sysctl.conf文件来完成。

还可以通过sysctl -a命令来查看所有的系统配置参数。

# IP协议相关参数配置

* net.ipv4.ip_default_ttl：设置从本机发出的ip包的生存时间，参数值为整数，范围为0～128，缺省值为64。如果系统经常得到“Time to live exceeded”的icmp回应，可以适当增大该参数的值，但是也不能过大，因为如果你的路由的环路的话，就会增加系统报错的时间。

# TCP协议相关参数配置

TCP链接是有很多开销的，一个是会占用文件描述符，另一个是会开缓存，一般来说一个系统可以支持的TCP链接数是有限的。

## TCP常规参数

* /proc/sys/net/ipv4/tcp_window_scaling：设置tcp/ip会话的滑动窗口大小是否可变。参数值为布尔值，为1时表示可变，为0时表示不可变。Tcp/ip 通常使用的窗口最大可达到65535字节，对于高速网络，该值可能太小，这时候如果启用了该功能，可以使tcp/ip滑动窗口大小增大数个数量级，从而提高数据传输的能力。

* net.core.rmem_default：默认的接收窗口大小。

* net.core.rmem_max：接收窗口的最大大小。

* net.core.wmem_default：默认的发送窗口大小。

* net.core.wmem_max：发送窗口的最大大小。

* /proc/sys/net/core/wmem_max。最大的TCP发送数据缓冲区大小。

* /proc/sys/net/ipv4/tcp_timestamps。时间戳在(请参考RFC 1323)TCP的包头增加10个字节，以一种比重发超时更精确的方法（请参阅 RFC 1323）来启用对 RTT 的计算；为了实现更好的性能应该启用这个选项。


## 配置KeepAlive参数

这个参数的意思是定义一个时间，如果链接上没有数据传输，系统会在这个时间发一个包，如果没有收到回应，那么TCP就认为链接断了，然后就会把链接关闭，这样可以回收系统资源开销。（注：HTTP层上也有KeepAlive参数）对于像HTTP这样的短链接，设置一个1-2分钟的keepalive非常重要。

* net.ipv4.tcp_keepalive_time：当keepalive打开的情况下，TCP发送keepalive消息的频率。缺省值为7200,即两小时，建议将其更改为1800。

* net.ipv4.tcp_keepalive_probes：TCP发送keepalive探测以确定该连接已经断开的次数。(默认值是9，设置为5比较合适)。

* net.ipv4.tcp_keepalive_intvl：探测消息发送的频率，乘以tcp_keepalive_probes就得到对于从开始探测以来没有响应的连接杀除的时间。(默认值为75秒，推荐设为15秒)

## 配置建立连接参数

* /proc/sys/net/ipv4/tcp_syn_retries：设置开始建立一个tcp会话时，重试发送syn连接请求包的次数。参数值为小于255的整数，缺省值为10。假如你的连接速度很快，可以考虑降低该值来提高系统响应时间，即便对连接速度很慢的用户，缺省值的设定也足够大了。

* net.ipv4.tcp_retries1：建立一个连接的最大重试次数，默认为3,不建议修改。

## 配置关闭连接参数

主动关闭的一方进入TIME_WAIT状态，TIME_WAIT状态将持续2个MSL(Max Segment Lifetime)，默认为4分钟，TIME_WAIT状态下的资源不能回收。有大量的TIME_WAIT链接的情况一般是在HTTP服务器上。

* net.ipv4.tcp_retries2：普通数据的重传次数，在丢弃激活(已建立通讯状况)的TCP连接之前﹐需要进行多少次重试。默认值为15，根据RTO的值来决定，相当于13-30分钟(RFC1122规定，必须大于100秒)。

* net.ipv4.tcp_tw_reuse：该文件表示是否允许重新应用处于TIME-WAIT状态的socket用于新的TCP连接。可以将其设置为1

* net.ipv4.tcp_tw_recycle：打开快速TIME-WAIT sockets回收。默认关闭，建议打开。

* net.ipv4.tcp_fin_timeout：在一个tcp会话过程中，在会话结束时，A首先向B发送一个fin包，在获得B的ack确认包后，A就进入FIN WAIT2状态等待B的fin包然后给B发ack确认包。这个参数就是用来设置A进入FIN WAIT2状态等待对方fin包的超时时间。如果时间到了仍未收到对方的fin包就主动释放该会话。参数值为整数，单位为秒，缺省为180秒，建议设置成30秒。

# 网卡的参数设置

## 调整网卡的txqueuelen
txqueuelen的涵义为网卡的发送队列长度，可以通过ifconfig命令找到网卡的txqueuelen参数配置，默认为1000,建议将其更改为5000。

## 网卡的中断设置
网卡的读写是通过硬件的中断机制来实现的，默认网卡是中断在cpu的内核0上。CPU0非常重要，CPU0具有调整功能，如果CPU0利用率过高，其他cpu核心的利用率也会下降。因此可以考虑将linux的网卡中断绑定到其他的cpu内核上。

可以通过/proc/interrupt文件内容来查看网卡在cpu核的中断情况。
```
           CPU0       CPU1       CPU2       CPU3       
  0:         16          0          0          0   IO-APIC-edge      timer
  1:      17144       2552       2312       2011   IO-APIC-edge      i8042
  5:          0          0          0          0   IO-APIC-edge      parport0
  8:          1          0          0          0   IO-APIC-edge      rtc0
  9:          0          0          0          0   IO-APIC-fasteoi   acpi
 19:      75011     144479      28826      16212   IO-APIC-fasteoi   ata_piix, ata_piix
 23:     214627          1          0          2   IO-APIC-fasteoi   ehci_hcd:usb1, ehci_hcd:usb2
 41:          0          0          0          0   PCI-MSI-edge      xhci_hcd
 42:     245675         11         17          6   PCI-MSI-edge      eth0
 43:         18          6          0          0   PCI-MSI-edge      mei
 44:     423481      80408      67559      54037   PCI-MSI-edge      i915
 45:        268        488        112         16   PCI-MSI-edge      snd_hda_intel
NMI:         21         18         23         18   Non-maskable interrupts
LOC:    7076902    6761203    8711912    9042018   Local timer interrupts
SPU:          0          0          0          0   Spurious interrupts
PMI:         21         18         23         18   Performance monitoring interrupts
IWI:          0          0          0          0   IRQ work interrupts
RTR:          2          0          0          0   APIC ICR read retries
RES:     365564     155654      52242      40754   Rescheduling interrupts
CAL:      12775      15182      23713      23003   Function call interrupts
TLB:      31028      26721      21584      25548   TLB shootdowns
TRM:          0          0          0          0   Thermal event interrupts
THR:          0          0          0          0   Threshold APIC interrupts
MCE:          0          0          0          0   Machine check exceptions
MCP:         32         32         32         32   Machine check polls
ERR:          0
MIS:          0

```
可以看到eth0的中断是在CPU0上，可以通过/proc/irq/42/smp_affinity文件查看eth0默认的中断分配情况，文件的内容为1,对应二进制为0001,对应的为CPU0。

要想修改中断分配方式，需要先停掉IRQ自动调节的服务进程。
```
/etc/init.d/irqbalance stop
echo "2" > /proc/irq/42/smp_affinity
```
这里的2表示将中断分配到CPU1上。

## 双网卡负载均衡
两个网卡共用一个IP地址，中断用两个核，效率可以提升一倍。

# 服务器的其他设置

## 修改服务器启动级别
可以通过runlevel命令查看机器的启动级别，带图形界面的启动级别为5。可以通过`init 3`来切换到启动级别3;可以修改/etc/inittab文件中`id:5:initdefault:`来修改默认级别。

## 修改系统的文件描述符数
如果网络链接较多，可以修改每个进程打开的最大文件描述符数目，默认为1024.可以通过ulimit -a查看，可以通过ulimit -n 32768来修改。

# 参考网页

* [性能调优攻略](http://coolshell.cn/articles/7490.html/comment-page-1)
* [Linux 内核网络参数配置资料](http://www.cnblogs.com/gunl/archive/2010/09/25/1834526.html)
* [Linux 多核下绑定硬件中断到不同 CPU（IRQ Affinity）](http://www.vpsee.com/2010/07/load-balancing-with-irq-smp-affinity/)
* [计算 SMP IRQ Affinity](http://www.vpsee.com/2010/07/smp-irq-affinity/)
* [提高 Linux 上 socket 性能](http://www.ibm.com/developerworks/cn/linux/l-hisock.html)
