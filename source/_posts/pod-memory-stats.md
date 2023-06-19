title: 内存使用分析
date: 2022-04-14 20:33:17
tags:
author:
---
## 操作系统级别

### /proc/meminfo

其中的Buffers和Cached的迷惑性非常大，非常难理解。Buffers是指的磁盘数据的缓存，Cached是指对文件数据的缓存.

### free

该命令实际上是通过读取/proc/meminfo文件得到的如下输出

```
$ free -h
              total        used        free      shared  buff/cache   available
Mem:            30G         22G        412M         47M        7.6G        7.3G
Swap:            0B          0B          0B
```

具体列含义：
- total: 总内存大小
- used：已经使用的内存大小，包含共享内存
- free：未使用的内存大小
- shared：共享内存大小
- buff/cache：缓存和缓冲区的大小
- available：新进程可用的内存大小

### vmstat

```
$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 4  0      0 1091484 1142360 34545436    0    0   146   265    0    0  8  5 86  1  0
```

## 容器级别

### 通过kubectl命令来查看内存使用

```
$ kubectl top pod nginx-ingress-controller-85cd6c7b5d-md6vc
NAME                                        CPU(cores)   MEMORY(bytes)
nginx-ingress-controller-85cd6c7b5d-md6vc   22m          502Mi
```

### 通过docker stats命令来查看容器

```
$ docker stats $container_id
CONTAINER ID  NAME                CPU %   MEM USAGE / LIMIT   MEM %  NET I/O   BLOCK I/O           PIDS
97d8bff3f89f  k8s_nginx-ingress   6.33%   180.3MiB / 512MiB   35.21% 0B / 0B   0B / 0B             119
```

docker通过cgroup的统计数据来获取的内存值

参考文档：https://docs.docker.com/engine/reference/commandline/stats/

## 进程级别

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/vm-1.png)

具体的含义可以通过下文的/proc/$pid/status来查看，其他进程的内存含义

### cat /proc/$pid/status

```
cat status
Name:   nginx
Umask:  0022
State:  S (sleeping)
Tgid:   46787
Ngid:   0
Pid:    46787
PPid:   33
TracerPid:      0
Uid:    1000    1000    1000    1000
Gid:    19062   19062   19062   19062
FDSize: 128
Groups:
VmPeak:   559768 kB
VmSize:   559120 kB
VmLck:         0 kB
VmPin:         0 kB
VmHWM:    285240 kB
VmRSS:    284752 kB
RssAnon:          279016 kB
RssFile:            3420 kB
RssShmem:           2316 kB
VmData:   371784 kB
VmStk:       136 kB
VmExe:      4716 kB
VmLib:      6828 kB
VmPTE:       900 kB
VmSwap:        0 kB
Threads:        33
SigQ:   1/123857
SigPnd: 0000000000000000
ShdPnd: 0000000000000000
SigBlk: 0000000000000000
SigIgn: 0000000040001000
SigCgt: 0000000198016eef
CapInh: 0000001fffffffff
CapPrm: 0000000000000400
CapEff: 0000000000000400
CapBnd: 0000001fffffffff
CapAmb: 0000000000000000
NoNewPrivs:     0
Seccomp:        0
Speculation_Store_Bypass:       vulnerable
Cpus_allowed:   0001
Cpus_allowed_list:      0
Mems_allowed:   00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001
Mems_allowed_list:      0
voluntary_ctxt_switches:        343464
nonvoluntary_ctxt_switches:     35061
```

具体字段含义可以查看man文档：https://man7.org/linux/man-pages/man5/proc.5.html，跟内存相关的字段如下：

- VmRSS：虚拟内存驻留在物理内存中的部分大小
- VmHWM：使用物理内存的峰值
- VmData：进程占用的数据段大小

### top命令查看

```
$ top -p $pid
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
  46787 www-data  20   0  559120 286544   5740 S   1.3  0.9   2:43.64 nginx
```

具体列含义如下：

- VIRT：进程使用虚拟内存总大小，即使没有占用物理内存，包括了进程代码段、数据段、共享内存、已经申请的堆内存和已经换到swap空间的内存等
- RES: 进程实际使用的物理内存大小，不包括swap和共享内存
- SHR：与其他进程的共享内存、加载的动态链接库、程序代码段的大小
- MEM：进程使用的物理内存占系统总内存的百分比

### ps命令查看

```
$ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
www-data       1  0.0  0.0    212     8 ?        Ss   Apr09   0:00 /usr/bin/dumb-init -- /nginx-ingress-controller
www-data       6  0.9  0.3 813500 99644 ?        Ssl  Apr09  67:32 /nginx-ingress-controller
www-data      33  0.0  0.7 458064 242252 ?       S    Apr09   0:53 nginx: master process /usr/local/nginx/sbin/nginx -c /etc/nginx/nginx.conf
www-data   46786  0.0  0.7 459976 239996 ?       S    17:57   0:00 rollback logs/eagleeye.log interval=60 adjust=600
www-data   46787  1.3  0.8 559120 284452 ?       Sl   17:57   2:22 nginx: worker process
www-data   46788  1.0  0.8 558992 285168 ?       Sl   17:57   1:51 nginx: worker process
www-data   46789  0.0  0.7 452012 237152 ?       S    17:57   0:01 nginx: cache manager process
www-data   46790  0.0  0.8 490832 267600 ?       S    17:57   0:00 nginx: x
www-data   47533  0.0  0.0  60052  1832 pts/2    R+   20:50   0:00 ps aux
```

- RSS：虚拟内存中的常驻内存，即实际占用的物理内存，包括所有已经分配的堆内存、栈内存、共享内存，由于共享内存非独占，实际上进程独占的物理内存要少于RSS

### pmap

```
# pmap -x 452021
452021:   nginx: worker process
Address           Kbytes     RSS   Dirty Mode  Mapping
0000000000400000    4716    1540       0 r-x-- tengine
0000000000a9a000      20      16       4 r---- tengine
0000000000a9f000     212     196     176 rw--- tengine
0000000000ad4000     248      60      60 rw---   [ anon ]
00002b0da6cb0000     136       4       0 r-x-- ld-2.17.so
00002b0da6cd2000       4       4       4 rw---   [ anon ]
00002b0da6cd3000       4       4       4 rw-s- zero (deleted)
00002b0da6cd7000      28      24      24 rw---   [ anon ]
00002b0da6cde000      64       8       8 rwx--   [ anon ]
00002b0da6ed1000       4       4       4 r---- ld-2.17.so
00002b0da6ed2000       4       4       4 rw--- ld-2.17.so
00002b0da6ed3000       4       4       4 rw---   [ anon ]
00002b0da6ed4000       8       0       0 r-x-- libdl-2.17.so
00002b0da70d8000      92      32       0 r-x-- libpthread-2.17.so
00002b0da72f0000      16       4       4 rw---   [ anon ]
00002b0da72f4000      32       0       0 r-x-- libcrypt-2.17.so
00002b0da72fc000    2044       0       0 ----- libcrypt-2.17.so
00002b0da74fb000       4       4       4 r---- libcrypt-2.17.so
00002b0da74fc000       4       4       4 rw--- libcrypt-2.17.so
00002b0da74fd000     184       0       0 rw---   [ anon ]
00002b0da752b000    1028       8       0 r-x-- libm-2.17.so
00002b0da7a35000     760     372       0 r-x-- libssl.so.1.1
00002b0da7d03000    2812    1124       0 r-x-- libcrypto.so.1.1
00002b0da7fc2000    2044       0       0 ----- libcrypto.so.1.1
00002b0da81c1000     172     172     172 r---- libcrypto.so.1.1
00002b0da81ec000      12      12      12 rw--- libcrypto.so.1.1
00002b0da81ef000      16       8       8 rw---   [ anon ]
00002b0da81f3000      84      40       0 r-x-- libgcc_s-4.8.5-20150702.so.1
00002b0da8409000    1808     316       0 r-x-- libc-2.17.so
00002b0da85cd000    2044       0       0 ----- libc-2.17.so
00002b0da87cc000      16      16      16 r---- libc-2.17.so
00002b0da87d0000       8       8       8 rw--- libc-2.17.so
00002b0da87d2000      20      12      12 rw---   [ anon ]
00002b0da87d7000       8       0       0 r-x-- libfreebl3.so
00002b0da87d9000    2044       0       0 ----- libfreebl3.so
00002b0da89d8000       4       4       4 r---- libfreebl3.so
00002b0da89d9000       4       4       4 rw--- libfreebl3.so
00002b0da8a00000   24576   21936   21936 rw---   [ anon ]
00002b0daa200000    8192    8180    8180 rw---   [ anon ]
00002b0daaa00000    8192    8192    8192 rw---   [ anon ]
00002b0dab200000    8192    8192    8192 rw---   [ anon ]
00002b0daba00000    8192    8172    8172 rw---   [ anon ]
00002b0dac200000    4096    4096    4096 rw---   [ anon ]
00002b0dac600000    4096    4096    4096 rw---   [ anon ]
00002b0daca00000    4096      12      12 rw-s- zero (deleted)
00002b0dace00000    1024       0       0 rw-s- zero (deleted)
00002b0dacf00000    1024       0       0 rw-s- zero (deleted)
00002b0dad000000   16384   16384   16384 rw---   [ anon ]
00002b0dae000000   10240       0       0 rw-s- zero (deleted)
00002b0db3f00000      24      16       0 r-x-- cjson.so
00002b0db3f06000    2048       0       0 ----- cjson.so
00002b0db4106000       4       4       4 r---- cjson.so
00002b0db4107000       4       4       4 rw--- cjson.so
00002b0db4108000       8       0       0 r-x-- librestychash.so
00002b0db410a000    2044       0       0 ----- librestychash.so
00002b0db4309000       4       4       4 r---- librestychash.so
00002b0db430a000       4       4       4 rw--- librestychash.so
00002b0db430b000   10240    1784    1784 rw-s- zero (deleted)
00002b0db4d0b000   10240       0       0 rw-s- zero (deleted)
00002b0db5800000    6144    6144    6144 rw---   [ anon ]
00002b0db5e00000    6144    6144    6144 rw---   [ anon ]
00002b0db6400000    8192    8192    8192 rw---   [ anon ]
00002b0db6c0b000    5120       8       8 rw-s- zero (deleted)
00002b0db7200000    8192    8192    8192 rw---   [ anon ]
00002b0dc2800000    1024       0       0 rw-s- zero (deleted)
00002b0dc2900000   10240       0       0 rw-s- zero (deleted)
00002b0dc3300000   10240       0       0 rw-s- zero (deleted)
00002b0dc861f000    2048       8       8 rw---   [ anon ]
00002b0dc881f000       4       0       0 -----   [ anon ]
00002b0dc8820000    2048       8       8 rw---   [ anon ]
00007fff60f6c000     136      44      44 rw---   [ stack ]
00007fff60fdd000       8       4       0 r-x--   [ anon ]
ffffffffff600000       4       0       0 r-x--   [ anon ]
---------------- ------- ------- -------
total kB          558996  289376  285888
```

- Mapping: 支持如下值
	- [anon]：分配的内存