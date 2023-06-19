---
title: Linux中断
date: 2018-11-30 21:09:20
tags:
---

# 概念

中断由硬件产生，并发送到中断控制器，中断控制器再发送中断到CPU，CPU检测到中断信号后，会中断当前的工作，每个中断都有IRQ（中断请求），基于IRQ，CPU将中断请求分发到对应的硬件驱动上通知操作系统，操作系统会对中断进行处理。

## 中断控制器

常见的中断控制器有两种：可编程中断控制器8259A和高级可编程中断控制器（APIC）。传统的8259A只适合单CPU的情况，现在都是多CPU、多核心的SMP体系，所以为了充分利用SMP体系结构，把中断传递给系统上的每个CPU以便更好实现并行和提高性能，Intel引入了高级可编程中断控制器（APIC）。

光有高级可编程中断控制器的硬件支持还不够，Linux内核还必须能利用这些硬件的特质，所以只有kernel 2.4以后的版本才支持把不同的硬件中断请求（IRQs）分配到特定的CPU核心上，这个绑定技术被称为SMP IRQ Affinity。

在设置网卡中断的cpu core时，有一个限制就是，IO-APIC 有两种工作模式：logic 和 physical，在 logic 模式下 IO-APIC 可以同时分布同一种 IO 中断到8颗 CPU (core) 上（受到 bitmask 寄存器的限制，因为 bitmask 只有8位长。）；在 physical 模式下不能同时分布同一中断到不同 CPU 上，比如，不能让 eth0 中断同时由 CPU0 和 CPU1 处理，这个时候只能定位 eth0 到 CPU0、eth1 到 CPU1，也就是说 eth0 中断不能像 logic 模式那样可以同时由多个 CPU 处理。

## 软中断和硬中断

为了解决中断处理程序执行时间过长和中断丢失的问题，Linux系统将中断分为上半部和下半部。

上半部在中断禁止模式下运行，用来快速处理中断，主要用来处理跟硬件密切相关的工作。

下半部处理上半部未完成的工作，通常以内核线程的方式运行。

以Linux接收网卡数据包为例进行说明：

网卡接收到一个数据包后，会通过硬件中断的方式通知内核新的数据到了。内核会调用中断处理程序进行处理。

上半部将网卡中的数据写入到内存中，并更新一下硬件寄存器的状态，最后发送一个软中断信号，通知下半部进一步的处理。

下半部被软中断信号唤醒后，从内存中读到数据，按照网络协议栈对数据进行解析和处理，并发送给应用程序。

上面所说的上半部即硬中断，下半部即软中断，但一些内核自定义的事件也属于软中断，比如内核调度和RCU锁等。

硬中断：由外设产生，用来通知操作系统外设状态的变化。在处理中断的同时要关闭中断。特点为处理要尽可能的快。

软中断：为了满足实时性要求，硬中断处理时间都比较短，将时间比较长的中断放到软中断中来完成，称为下半部。int就是软中断指令，中断向量表是中断号和中断处理程序的对应表。每个CPU对应一个软中断内核线程`ksoftirqd/cpu编号`。

```
[root@120-14-29-SH-1037-B07 ~]# ps -ef | grep ksoft
root           3       2  0  2016 ?        01:47:12 [ksoftirqd/0]
root          21       2  0  2016 ?        00:47:51 [ksoftirqd/1]
root          26       2  0  2016 ?        00:47:34 [ksoftirqd/2]
root          31       2  0  2016 ?        00:47:46 [ksoftirqd/3]
```

中断嵌套：硬中断可以嵌套，即新的硬中断可以打断正在执行的中断，但同种中断不可以。软中断不可嵌套，但相同类型中断可在不同的cpu上执行。


# 相关命令

## mpstat

```
# 显示cpu处理的中断数量
[root@103-17-164-sh-100-k07 ~]# mpstat -I SUM 1
Linux 3.10.0-327.10.1.el7.x86_64 (103-17-164-sh-100-k07.yidian.com)     09/09/2017      _x86_64_        (4 CPU)

05:27:59 PM  CPU    intr/s
05:28:00 PM  all  61274.00
05:28:01 PM  all  61712.00
05:28:02 PM  all  62315.00
05:28:03 PM  all  59280.00
^C
Average:     all  61145.25

# 显示每个核处理的中断数量
[root@103-17-164-sh-100-k07 ~]# mpstat -I SUM 1 -P ALL
Linux 3.10.0-327.10.1.el7.x86_64 (103-17-164-sh-100-k07.yidian.com)     09/09/2017      _x86_64_        (4 CPU)

05:30:30 PM  CPU    intr/s
05:30:31 PM  all  61446.00
05:30:31 PM    0  40489.00
05:30:31 PM    1   6839.00
05:30:31 PM    2   6935.00
05:30:31 PM    3   7185.00

# 显示更详细的信息
[root@120-14-31-SH-1037-B07 ~]# mpstat -P ALL 1
Linux 3.10.0-327.el7.x86_64 (120-14-31-SH-1037-B07.yidian.com)  09/10/2017      _x86_64_        (4 CPU)

01:15:09 AM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
01:15:10 AM  all    6.35    0.00   11.64    0.00    0.00    7.67    0.00    0.00    0.00   74.34
01:15:10 AM    0    5.05    0.00   11.11    0.00    0.00    0.00    0.00    0.00    0.00   83.84
01:15:10 AM    1    6.38    0.00   12.77    0.00    0.00    9.57    0.00    0.00    0.00   71.28
01:15:10 AM    2    8.70    0.00   10.87    0.00    0.00   14.13    0.00    0.00    0.00   66.30
01:15:10 AM    3    6.32    0.00   12.63    0.00    0.00    7.37    0.00    0.00    0.00   73.68
```

## lspci

可以来查看网卡型号，驱动等信息，内容较多

```
[root@103-17-164-sh-100-k07 ~]# lspci -vvv | more
00:00.0 Host bridge: Intel Corporation Xeon E7 v2/Xeon E5 v2/Core i7 DMI2 (rev 04)
        Subsystem: Super Micro Computer Inc Device 0668
        Control: I/O- Mem- BusMaster- SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx-
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Interrupt: pin A routed to IRQ 0
        Capabilities: [90] Express (v2) Root Port (Slot-), MSI 00
                DevCap: MaxPayload 128 bytes, PhantFunc 0
                        ExtTag- RBE+
                DevCtl: Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
                        RlxdOrd- ExtTag- PhantFunc- AuxPwr- NoSnoop-
                        MaxPayload 128 bytes, MaxReadReq 128 bytes
                DevSta: CorrErr- UncorrErr- FatalErr- UnsuppReq- AuxPwr- TransPend-
                LnkCap: Port #0, Speed 5GT/s, Width x4, ASPM not supported, Exit Latency L0s <64ns, L1 <16us
                        ClockPM- Surprise+ LLActRep+ BwNot+
                LnkCtl: ASPM Disabled; RCB 64 bytes Disabled- CommClk-
                        ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
                LnkSta: Speed unknown, Width x0, TrErr- Train- SlotClk- DLActive- BWMgmt- ABWMgmt-
                RootCtl: ErrCorrectable- ErrNon-Fatal- ErrFatal- PMEIntEna- CRSVisible-
                RootCap: CRSVisible-
                RootSta: PME ReqID 0000, PMEStatus- PMEPending-
                DevCap2: Completion Timeout: Range BCD, TimeoutDis+, LTR-, OBFF Not Supported ARIFwd-
                DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis-, LTR-, OBFF Disabled ARIFwd-
                LnkCtl2: Target Link Speed: 2.5GT/s, EnterCompliance- SpeedDis-
                         Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
                         Compliance De-emphasis: -6dB
                LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete-, EqualizationPhase1-
                         EqualizationPhase2-, EqualizationPhase3-, LinkEqualizationRequest-
        Capabilities: [e0] Power Management version 3
                Flags: PMEClk- DSI- D1- D2- AuxCurrent=0mA PME(D0+,D1-,D2-,D3hot+,D3cold+)
                Status: D0 NoSoftRst- PME-Enable- DSel=0 DScale=0 PME-
        Capabilities: [100 v1] Vendor Specific Information: ID=0002 Rev=0 Len=00c <?>
        Capabilities: [144 v1] Vendor Specific Information: ID=0004 Rev=1 Len=03c <?>
        Capabilities: [1d0 v1] Vendor Specific Information: ID=0003 Rev=1 Len=00a <?>
        Capabilities: [280 v1] Vendor Specific Information: ID=0005 Rev=3 Len=018 <?>

...
```

## ethtool

用来查看网卡信息

```
[root@103-17-164-sh-100-k07 ~]# ethtool eth3
Settings for eth3:
        Supported ports: [ FIBRE ]
        Supported link modes:   1000baseT/Full
                                10000baseT/Full
        Supported pause frame use: No
        Supports auto-negotiation: Yes
        Advertised link modes:  1000baseT/Full
                                10000baseT/Full
        Advertised pause frame use: No
        Advertised auto-negotiation: Yes
        Speed: 10000Mb/s
        Duplex: Full
        Port: FIBRE
        PHYAD: 0
        Transceiver: external
        Auto-negotiation: on
        Supports Wake-on: d
        Wake-on: d
        Current message level: 0x00000007 (7)
                               drv probe link
        Link detected: yes

# 可查看Ring buffer的大小
[root@103-17-164-sh-100-k07 ~]# ethtool -g eth3
Ring parameters for eth3:
Pre-set maximums:
RX:             4096
RX Mini:        0
RX Jumbo:       0
TX:             4096
Current hardware settings:
RX:             512
RX Mini:        0
RX Jumbo:       0
TX:             512

# 列出信息较多，包含网卡的统计信息，包括丢包量信息
[root@103-17-164-sh-100-k07 ~]# ethtool -S eth3 | more
NIC statistics:
     rx_packets: 680955795162
     tx_packets: 27260701850
     rx_bytes: 248285654162670
     tx_bytes: 195321924245892
     rx_pkts_nic: 683081802539
     tx_pkts_nic: 27260700665
     rx_bytes_nic: 251132784690871
     tx_bytes_nic: 195447730645152
     lsc_int: 11
     tx_busy: 0
     non_eop_descs: 1811095697
     rx_errors: 67381
     tx_errors: 0
     rx_dropped: 0
     tx_dropped: 0
     multicast: 1025690658
     broadcast: 206937242
     rx_no_buffer_count: 0
     collisions: 0
     rx_over_errors: 0
     rx_crc_errors: 67302
     rx_frame_errors: 0
     hw_rsc_aggregated: 2358414213
     hw_rsc_flushed: 232402327
     fdir_match: 10634169417
     fdir_miss: 669277016191
     fdir_overflow: 3321
     rx_fifo_errors: 0
     rx_missed_errors: 2538
     tx_aborted_errors: 0
     tx_carrier_errors: 0
     tx_fifo_errors: 0
     tx_heartbeat_errors: 0
     tx_timeout_count: 0
     tx_restart_queue: 0
     rx_long_length_errors: 80264
     rx_short_length_errors: 0
     tx_flow_control_xon: 1
     rx_flow_control_xon: 0
     tx_flow_control_xoff: 51
     rx_flow_control_xoff: 0
     rx_csum_offload_errors: 0
     alloc_rx_page_failed: 0
     alloc_rx_buff_failed: 0
     rx_no_dma_resources: 0
     os2bmc_rx_by_bmc: 0
     os2bmc_tx_by_bmc: 0
     os2bmc_tx_by_host: 0
     os2bmc_rx_by_host: 0

# 查看网卡多队列的支持情况，当前网卡支持8个队列，使用了8个队列
[root@103-17-6-sh-100-j11 ~]# ethtool -l eth0
Channel parameters for eth0:
Pre-set maximums:
RX:             0
TX:             0
Other:          1
Combined:       8
Current hardware settings:
RX:             0
TX:             0
Other:          1
Combined:       8

# 设置网卡当前使用的多队列，当前使用的网卡数量不能超过最大值8，该值跟网卡的中断数量一一对应，即/proc/interrupts中看到的eth0的中断数量
[root@103-17-6-sh-100-j11 ~]# ethtool -L eth0 combined 2
[root@103-17-6-sh-100-j11 ~]# ethtool -l eth0
Channel parameters for eth0:
Pre-set maximums:
RX:             0
TX:             0
Other:          1
Combined:       8
Current hardware settings:
RX:             0
TX:             0
Other:          1
Combined:       2
```

## sar

```
# 列出网卡的接收包信息，比iftop更直观
[root@103-17-164-sh-100-k07 ~]# sar -n DEV 1
Linux 3.10.0-327.10.1.el7.x86_64 (103-17-164-sh-100-k07.yidian.com)     09/09/2017      _x86_64_        (4 CPU)

05:06:12 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
05:06:13 PM      eth0      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:06:13 PM      eth1      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:06:13 PM      eth2      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:06:13 PM      eth3  77893.00   4328.00  31413.92  30134.16      0.00      0.00     46.00
05:06:13 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00

# 可列出错误包的相关信息
[root@103-17-164-sh-100-k07 ~]# sar -n EDEV 1
Linux 3.10.0-327.10.1.el7.x86_64 (103-17-164-sh-100-k07.yidian.com)     09/09/2017      _x86_64_        (4 CPU)

05:07:13 PM     IFACE   rxerr/s   txerr/s    coll/s  rxdrop/s  txdrop/s  txcarr/s  rxfram/s  rxfifo/s  txfifo/s
05:07:14 PM      eth0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:07:14 PM      eth1      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:07:14 PM      eth2      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:07:14 PM      eth3      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
05:07:14 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

通过中断可以看到网卡包含四个中断55-58，均位于cpu0上。

```
[root@103-17-164-sh-100-k07 ~]# cat /proc/interrupts
           CPU0       CPU1       CPU2       CPU3
  0:         44          0          0          0  IR-IO-APIC-edge      timer
  1:          3          0          0          0  IR-IO-APIC-edge      i8042
  8:         42          0          0          0  IR-IO-APIC-edge      rtc0
  9:          1          0          0          0  IR-IO-APIC-fasteoi   acpi
 12:          4          0          0          0  IR-IO-APIC-edge      i8042
 16:         99          0          0          0  IR-IO-APIC-fasteoi   ehci_hcd:usb1
 18:          0          0          0          0  IR-IO-APIC-fasteoi   i801_smbus
 23:         83          0          0          0  IR-IO-APIC-fasteoi   ehci_hcd:usb2
 37:   12019917          0          0          0  IR-PCI-MSI-edge      0000:00:1f.2
 48:          0          0          0          0  DMAR_MSI-edge      dmar0
 55:  746827548          0          0          0  IR-PCI-MSI-edge      eth3-TxRx-0
 56: 2674693551          0          0          0  IR-PCI-MSI-edge      eth3-TxRx-1
 57: 2341522223          0          0          0  IR-PCI-MSI-edge      eth3-TxRx-2
 58: 3587929355          0          0          0  IR-PCI-MSI-edge      eth3-TxRx-3
 59:       3334          0          0          0  IR-PCI-MSI-edge      eth3
 61:          2          0          0          0  IR-PCI-MSI-edge      ioat-msix
 63:          2          0          0          0  IR-PCI-MSI-edge      ioat-msix
 64:          2          0          0          0  IR-PCI-MSI-edge      ioat-msix
 65:          2          0          0          0  IR-PCI-MSI-edge      ioat-msix
 66:          2          0          0          0  IR-PCI-MSI-edge      ioat-msix
 67:          2          0          0          0  IR-PCI-MSI-edge      ioat-msix
 68:          2          0          0          0  IR-PCI-MSI-edge      ioat-msix
 69:          2          0          0          0  IR-PCI-MSI-edge      ioat-msix
NMI:    1100069     693493     635982     615953   Non-maskable interrupts
LOC: 1120358899  146726541 4134846029  168005659   Local timer interrupts
SPU:          0          0          0          0   Spurious interrupts
PMI:    1100069     693493     635982     615953   Performance monitoring interrupts
IWI:   57255892  115292160  113458706  112987848   IRQ work interrupts
RTR:          0          0          0          0   APIC ICR read retries
RES:  525229423 2791640970  427214674 1986396041   Rescheduling interrupts
CAL: 4294536344 4294488238 4294478163 4294552026   Function call interrupts
TLB:   65239533   57937650   55104990   52690662   TLB shootdowns
TRM:          0          0          0          0   Thermal event interrupts
THR:          0          0          0          0   Threshold APIC interrupts
MCE:          0          0          0          0   Machine check exceptions
MCP:     160132     160132     160132     160132   Machine check polls
ERR:          0
MIS:          0
```

## 修改中断的cpu分配

echo "2" > /proc/irq/49/smp_affinity

其中2表示cpu1, 49表示中断号。

## 查看软中断

```
[root@120-14-31-SH-1037-B07 ~]# cat /proc/softirqs
                    CPU0       CPU1       CPU2       CPU3
          HI:          1          3          0          1
       TIMER: 1795378091 3617740778 2674553229 1524071492
      NET_TX:  202188392   22218135   17427628   17205883
      NET_RX: 3388060179   60871361   68145291   38670323
       BLOCK:    4741950       2422       1309       1489
BLOCK_IOPOLL:          0          0          0          0
     TASKLET: 2102131738    3720214    3104944    1942912
       SCHED:  526046585  612421231  496815061  456047989
     HRTIMER:          0          0          0          0
         RCU: 3147020579 4237695975 3820083676 3267816268
```

软中断包括10个类别，NET_RX（网络接收中断）、NET_TX（网络发送中断）

## 查看数据包统计

```
[root@c1-g08-120-166-30 ~]# netstat -s
Ip:
    7586513384 total packets received
    0 forwarded
    741 with unknown protocol
    0 incoming packets discarded
    7586512643 incoming packets delivered
    7948370396 requests sent out
Icmp:
    23 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
        destination unreachable: 1
        echo requests: 19
        echo replies: 3
    46 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 9
        echo request: 18
        echo replies: 19
IcmpMsg:
        InType0: 3
        InType3: 1
        InType8: 19
        OutType0: 19
        OutType3: 9
        OutType8: 18
Tcp:
    561299810 active connections openings
    2005002 passive connection openings
    8 failed connection attempts
    282644949 connection resets received
    725 connections established
    7585817181 segments received
    13957880471 segments send out
    1742807 segments retransmited
    136 bad segments received.
    523811266 resets sent
Udp:
    445457 packets received
    3 packets to unknown port received.
    0 packet receive errors
    553840367 packets sent
    0 receive buffer errors
    0 send buffer errors
UdpLite:
    InErrors: 3
TcpExt:
    341304 invalid SYN cookies received
    1 resets received for embryonic SYN_RECV sockets
    1 ICMP packets dropped because they were out-of-window
    1997421 TCP sockets finished time wait in fast timer
    222787 delayed acks sent
    1819 delayed acks further delayed because of locked socket
    Quick ack mode was activated 178186 times
    13 packets directly queued to recvmsg prequeue.
    3280604069 packet headers predicted
    1972740996 acknowledgments not containing data payload received
    798190042 predicted acknowledgments
    642444 times recovered from packet loss by selective acknowledgements
    Detected reordering 23 times using FACK
    Detected reordering 1618 times using SACK
    59 congestion windows fully recovered without slow start
    16 congestion windows partially recovered using Hoe heuristic
    17089 congestion windows recovered without slow start by DSACK
    9594 congestion windows recovered without slow start after partial ack
    TCPLostRetransmit: 2849
    4926 timeouts after SACK recovery
    672451 fast retransmits
    28946 forward retransmits
    680 retransmits in slow start
    5024 other TCP timeouts
    TCPLossProbes: 1390602
    TCPLossProbeRecovery: 997235
    4723 SACK retransmits failed
    178189 DSACKs sent for old packets
    979863 DSACKs received
    62 DSACKs for out of order packets received
    282645553 connections reset due to unexpected data
    9 connections reset due to early user close
    TCPDSACKIgnoredNoUndo: 936061
    TCPSpuriousRTOs: 5150
    TCPSackShifted: 233309
    TCPSackMerged: 733778
    TCPSackShiftFallback: 1768422
    IPReversePathFilter: 1
    TCPRetransFail: 11
    TCPRcvCoalesce: 1290965687
    TCPOFOQueue: 1753072
    TCPChallengeACK: 136
    TCPSYNChallenge: 136
    TCPSpuriousRtxHostQueues: 73
    TCPAutoCorking: 272452106
    TCPSynRetrans: 4538
    TCPOrigDataSent: 9260789170
    TCPHystartTrainDetect: 3721895
    TCPHystartTrainCwnd: 64417412
    TCPHystartDelayDetect: 80
    TCPHystartDelayCwnd: 2579
    TCPACKSkippedSynRecv: 4
IpExt:
    InMcastPkts: 249744
    OutMcastPkts: 83317
    InBcastPkts: 226
    InOctets: 12312144006244
    OutOctets: 12449967070063
    InMcastOctets: 22309104
    OutMcastOctets: 9664620
    InBcastOctets: 104240
    InNoECTPkts: 7586513474
    InECT0Pkts: 47
```

# irqbalance

查看是否运行：systemctl status irqbalance

irqbalance根据系统中断负载的情况，自动迁移中断保持中断的平衡，同时会考虑到省电因素等等。 但是在实时系统中会导致中断自动漂移，对性能造成不稳定因素，在高性能的场合建议关闭。

irqbalance用于优化中断分配，它会自动收集系统数据以分析使用模式，并依据系统负载状况将工作状态置于 Performance mode 或 Power-save mode。处于Performance mode 时，irqbalance 会将中断尽可能均匀地分发给各个 CPU core，以充分利用 CPU 多核，提升性能。
处于Power-save mode 时，irqbalance 会将中断集中分配给第一个 CPU，以保证其它空闲 CPU 的睡眠时间，降低能耗。

# ref

* [Linux 多核下绑定硬件中断到不同 CPU（IRQ Affinity）](http://www.vpsee.com/2010/07/load-balancing-with-irq-smp-affinity/)
* [深度剖析告诉你irqbalance有用吗？](http://blog.yufeng.info/archives/2422)
* [怎么理解Linux软中断？](https://time.geekbang.org/column/article/71868)
