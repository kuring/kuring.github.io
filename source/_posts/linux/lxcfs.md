---
title: lxcfs 技术
permalink: /linux/lxcfs/
date: 2024-08-31
---
容器中的执行`top`、`free`等命令展示出来的CPU，内存等信息是从`/proc`目录中的相关文件里读取出来的。而容器并没有对`/proc`，`/sys`等文件系统做隔离，因此容器中读取出来的CPU和内存的信息是宿主机的信息，与容器实际分配和限制的资源量不同。
```
/proc/cpuinfo
/proc/diskstats
/proc/meminfo
/proc/stat
/proc/swaps
/proc/uptime
```

lxcfs是一个常驻进程运行在宿主机上，从而来自动维护宿主机cgroup中容器的真实资源信息与容器内`/proc`下文件的映射关系。

lxcfs实现的基本原理是通过文件挂载的方式，把cgroup中容器相关的信息读取出来，存储到lxcfs相关的目录下，并将相关目录映射到容器内的/proc目录下，从而使得容器内执行top,free等命令时拿到的/proc下的数据是真实的cgroup分配给容器的CPU和内存数据。

![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240830114743.png)

| 类别  | 容器内目录                          | 宿主机lxcfs目录                                   |
| --- | ------------------------------ | -------------------------------------------- |
| cpu | /proc/cpuinfo                  | /var/lib/lxcfs/proc/cpuinfo                  |
| 内存  | /proc/meminfo                  | /var/lib/lxcfs/proc/meminfo                  |
|     | /proc/diskstats                | /var/lib/lxcfs/proc/diskstats                |
|     | /proc/stat                     | /var/lib/lxcfs/proc/stat                     |
|     | /proc/swaps                    | /var/lib/lxcfs/proc/swaps                    |
|     | /proc/uptime                   | /var/lib/lxcfs/proc/uptime                   |
|     | /proc/loadavg                  | /var/lib/lxcfs/proc/loadavg                  |
|     | /sys/devices/system/cpu/online | /var/lib/lxcfs/sys/devices/system/cpu/online |

在每个容器内仅需要挂载 lxcfs 在宿主机上的目录到容器中的目录即可。