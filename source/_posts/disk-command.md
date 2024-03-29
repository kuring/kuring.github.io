---
title: Linux下磁盘常用命令
date: 2022-01-18 21:27:07
tags:
---

# 查看磁盘是否为ssd

如果其中的rota值为1，说明为hdd磁盘；如果rota值为0，说明为ssd。

```
# lsblk -o name,size,type,rota,mountpoint
NAME    SIZE TYPE ROTA MOUNTPOINT
vdc      20G disk    1 /var/lib/kubelet/pods/eea3a54c-b211-4ee3-bcbe-70ba3fe84c05/volumes/kubernetes.io~csi/d-t4n36xdqey47v9e0ej8r/mount
vda     120G disk    1
└─vda1  120G part    1 /
```

但该规则在很多虚拟机的场景下并不成立，即使虚拟机的磁盘为ssd，但rota值仍然为1。可以通过修改rota的值的方式来标记磁盘的类型：echo '0'> /sys/block/vdd/queue/rotational

# iostat

只能观察磁盘整体性能，不能看到进程级别的

- iops = r/s+w/s
- 吞吐量 = rkB/s + wkB/s
- 响应时间 = r_await + w_await

iostat -xm $interval: interval为秒

```
$ iostat -xm 1
Linux 3.10.0-327.10.1.el7.x86_64 (103-17-52-sh-100-I03.yidian.com)      12/11/2016      _x86_64_        (32 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           4.99    0.00    1.46    0.01    0.00   93.54

Device:         rrqm/s   wrqm/s     r/s     w/s    rMB/s    wMB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     0.26    0.02    1.62     0.00     0.02    29.31     0.00    0.89    3.37    0.87   0.05   0.01
sdb               0.00     0.81   10.23   67.59     1.19    29.07   796.57     0.01    0.10    0.85    2.85   0.19   1.46
```

* r/s: 每秒发送给磁盘的读请求数
* w/s: 每秒发送给磁盘的写请求数
* rMB/s: 每秒从磁盘读取的数据量
* wMB/s: 每秒向磁盘写入的数据量
* %iowait：cpu等待磁盘时间
* %util：表示磁盘使用率，该值较大说明磁盘性能存在问题，io队列不为空的时间。由于存在并行io，100%不代表磁盘io饱和
* r_await：读请求处理完成时间，包括队列中的等待时间和设备实际处理时间，单位为毫秒
* w_await：写请求处理完成时间，包括队列中的等待时间和设备实际处理时间，单位为毫秒
* svctm: 瓶颈每次io操作的时间，单位为毫秒，可以反映出磁盘的瓶颈
* avgrq-sz：平均每次请求的数据大小
* avgqu-sz：平均请求io队列长度
* svctm：处理io请求所需要的平均时间，不包括等待时间，单位为毫秒


```
$ iostat -d -x 1
Linux 3.10.0-327.10.1.el7.x86_64 (103-17-42-sh-100-i08.yidian.com)      01/19/2019      _x86_64_        (24 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
sda               0.00     7.58    0.31  266.07     9.54   888.01     6.74     0.03    0.12    8.84    0.11   0.03   0.90
sdb               0.00     0.67    4.39   40.68   386.20  9277.38   428.79     0.03    0.64    3.10    0.37   0.30   1.33
sdd               0.00     0.82    5.08   66.78   451.99 15587.89   446.42     0.02    0.30    3.50    0.05   0.28   2.01
sde               0.00     0.67    4.14   53.77   312.74 12121.84   429.43     0.00    0.03    4.00    1.44   0.30   1.72
sdc               0.00     0.50    0.05   27.47     2.74  6098.56   443.28     0.03    0.95   12.42    0.93   0.21   0.58
sdf               0.00     0.47    1.07   34.58    57.05  7504.68   424.24     0.00    0.05    2.60    2.63   0.25   0.88
sdg               0.00     0.72    0.14   63.87    13.79 14876.74   465.22     0.03    0.42   10.57    0.40   0.19   1.24
sdj               0.00     0.79    1.85   37.81   206.01  7881.89   407.90     0.03    0.67    2.27    0.59   0.36   1.42
sdi               0.00     0.65    0.29   59.54    22.01 13501.71   452.05     0.01    0.25    5.27    0.22   0.19   1.14
sdk               0.00     0.50    0.30   29.46    16.74  6330.68   426.57     0.05    1.58    4.03    1.55   0.24   0.70
sdm               0.00     0.63    0.73   55.70    63.73 13008.16   463.34     0.02    0.35    5.98    0.28   0.23   1.29
sdh               0.00     0.47    0.06   15.68     5.16  2970.09   378.01     0.03    1.82    7.22    1.80   0.27   0.42
sdl               0.00     0.56    1.32   38.14    81.04  8945.16   457.50     0.09    2.22    4.30    2.15   0.19   0.76
```

- wrqm: 每秒合并的写请求数
- rrqm：每秒合并的读请求数

# pidstat

如果要想查看进程级别的磁盘 io 使用情况，可以使用 `pidstat -d` 指令。

```
$ pidstat -d 1
Linux 3.10.0-1160.88.1.el7.x86_64 (izbp1feliilr5yri84y6saz)     05/08/2023      _x86_64_        (16 CPU)

09:44:24 AM   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
09:44:25 AM     0       371      0.00    184.91      0.00  jbd2/vda1-8
09:44:25 AM     0      1395      0.00     22.64      7.55  rsyslogd
09:44:25 AM     0     19053      0.00      3.77      0.00  jbd2/dm-0-8
09:44:25 AM  1000     25479      0.00      3.77      0.00  org_start
09:44:25 AM  1000     25754      0.00      3.77      0.00  org_start
09:44:25 AM     0     38784      0.00    505.66      0.00  systemd-journal
09:44:25 AM     0     40474      0.00    143.40      0.00  jbd2/dm-1-8
09:44:25 AM     0     41130      0.00     11.32      0.00  jbd2/dm-2-8
09:44:25 AM  1000     41675      0.00      3.77      0.00  start
09:44:25 AM     0     41730      0.00      3.77      0.00  org_start
```

# iotop

该命令可以查看所有进程读写 io 情况，该命令通常 os 不会默认安装。在 CentOS 下可以执行 `yum install iotop` 命令安装。
