title: 大页内存
date: 2022-03-07 18:20:17
tags:
author:
---
## 大页内存介绍
一个操作系统上一台机器的物理内存是有限的，而操作系统上却运行着大量的进程，为了对进程屏蔽物理内存的差异，操作系统引入了虚拟内存的概念。Linux系统中虚拟内存是按照页的方式来进行管理的，每个页的默认大小为4K。如果物理内存非常大，就会导致虚拟内存和物理内存映射的页表（TLB）非常大。为了减少页表，一个可行的方法为增加页的大小。
​

可以通过如下命令来查看默认页大小：
```yaml
$ getconf PAGE_SIZE
4096
```
在对内存要求特别严格的场景下，比如很多数据库的场景，都有巨页的需求，比如将页设置为1GB，甚至几十GB。
​

通常在大页内存的场景下，会将Linux的swap功能关闭，避免当页换出时导致的性能下降。
​

在Linux中Huge Page有2MB和1GB两种规则，其中2MB适用于GB级别的内存，1GB适用于TB级别的内存。
​

大页内存可以分为静态大页和透明大页两类。

- 静态大页需要用户自行控制大页的分配、释放和使用。但该方式需要事先配置大页内存的量，因此用起来不够灵活。配置可以在系统启动的时候配置hugepages（页数量）和hugepagesz（页大小）两个参数。也可以使用修改内核参数的方式来预留。
- 透明大页由操作系统的后台内核线程khugepaged控制大页的分配、释放和使用。

## 静态大页内存操作
预留大页内存
```shell
echo 20 >  /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```
可以通过/proc/meminfo文件系统看到大页内存的使用情况
```shell
# cat /proc/meminfo | grep Huge
AnonHugePages:   2037760 kB
HugePages_Total:      20 # 预先分配的大页数量
HugePages_Free:       20 # 空闲大页数量
HugePages_Rsvd:        0 
HugePages_Surp:        0
Hugepagesize:       2048 kB
```
挂载hugetlb文件系统
```shell
mount -t hugetlbfs none /mnt/huge
```
在应用程序中mmap映射hugetlb文件系统即可使用，对内存的操作类似于访问文件。

## 透明大页内存操作（待补充）
cat /sys/kernel/mm/transparent_hugepage/enabled

## 大页内存在k8s的支持情况
HugePage是k8s 1.9版本中引入的特性，1.10变为beta版本，1.23版本stable版本。k8s大于1.10版本该feature默认开启。

在节点上配置了大页内存后，kubelet会自动感知到大页内存的配置，会修改node配置。会在hugepages-2Mi中有对应的大页内存容量，memory字段会去掉对应的大页内存量。
```yaml
  allocatable:
    cpu: "15"
    ephemeral-storage: 41152812Ki
    hugepages-1Gi: "0"
    hugepages-2Mi: 40Mi
    memory: 63998924Ki
    pods: "110"
  capacity:
    cpu: "16"
    ephemeral-storage: 41152812Ki
    hugepages-1Gi: "0"
    hugepages-2Mi: 40Mi
    memory: 64756684Ki
    pods: "110"
```
pod在使用大页内存的时候，需要以volume的方式挂载到pod中，pod对大页内存的使用跟普通文件系统相同。
