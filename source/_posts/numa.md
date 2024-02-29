title: numa
date: 2024-02-21 18:04:26
tags:
author:
---
随着处理器数量的增加，所有的处理器均通过同一个北桥来访问内存，导致内存访问延迟增加，内存带宽成为瓶颈。

NUMA（Non-Uniform Memory Access）非统一内存访问，是一种针对多处理器系统的组织结构。处理器被分配到不同的节点，每个节点有自己的本地内存，处理器可以访问本地内存和其他节点的内存，但访问本地内存的速度要远远快于访问其他节点的内存。

几个概念：

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/numa.jpg)

1. Socket：一颗物理 CPU。
2. Numa Node：逻辑概念，对 CPU 分组的抽象，一个 Node 即为一个分组，一个分组下可以包含多个 CPU。每个 Node 都有自己的本地资源，包括内存和 IO。Numa Node 之间不共享 L3 cache。一个 Numa Node 内的不同 core 之间共享 L3.
3. Core：一颗物理 CPU 的物理核，每个 Core 有独立的 L1 和 L2，Core 之间共享 L3。
4. Thread/Processor：一颗物理 CPU 的逻辑核，用超线程的方式模拟出两个逻辑核。Processor 之间共享 L1 和 L2 cache，在开启超线程后单核的性能会下降一些。

一个 NUMA Node 可以有一个 Socket，每个 Socket 包含一个或者多个物理 Core。


# 常用命令
查看物理核数 Socket：`lscpu | grep Socket`
每个 Socket 包含的物理核 Core：`lscpu  | grep 'Core(s) per socket'`
每个 Socket 包含的 Thead/Processor 数量：`lscpu  | grep 'Thread(s) per core'`
查看所有的 Processor 数量：`cat /proc/cpuinfo | grep "processor" | wc -l`
```yaml
$ numactl --hardware
available: 4 nodes (0-3)
node 0 cpus: 0 1 2 3 4 5 6 7 32 33 34 35 36 37 38 39
node 0 size: 128470 MB
node 0 free: 88204 MB
node 1 cpus: 8 9 10 11 12 13 14 15 40 41 42 43 44 45 46 47
node 1 size: 129019 MB
node 1 free: 67982 MB
node 2 cpus: 16 17 18 19 20 21 22 23 48 49 50 51 52 53 54 55
node 2 size: 129019 MB
node 2 free: 38304 MB
node 3 cpus: 24 25 26 27 28 29 30 31 56 57 58 59 60 61 62 63
node 3 size: 128965 MB
node 3 free: 45689 MB
node distances:
node   0   1   2   3
  0:  10  16  28  22
  1:  16  10  22  28
  2:  28  22  10  16
  3:  22  28  16  10
```
如果没有开启 numa，默认只会看到一个 node，类似如下：
```yaml
$ numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63
node 0 size: 772904 MB
node 0 free: 510590 MB
node distances:
node   0
  0:  10
```
通过 numastat 命令可以看到 numa 的 miss 等情况，对于排查性能问题非常有帮助。
```yaml
$ numastat
                           node0           node1           node2           node3
numa_hit              6010986094      3863188690      7861549559      9798877648
numa_miss                      0               0       113102530               0
numa_foreign                   0               0               0       113102530
interleave_hit             17976           17846           17965           17839
local_node            6010748681      3862984578      7861301902      9798751537
other_node                237411          204112       113350178          126111
```
# 资料

- [CPU 拓扑：从 SMP 谈到 NUMA （理论篇）](https://ctimbai.github.io/2018/05/03/tech/linux/cpu/CPU%E6%8B%93%E6%89%91%E4%BB%8ESMP%E8%B0%88%E5%88%B0NUMA%EF%BC%88%E7%90%86%E8%AE%BA%E7%AF%87%EF%BC%89/)
