---
title: 部署java应用到容器
date: 2018-11-17 20:16:02
tags:
---

java 8u131之后的版本开始支持容器特性，之前的版本中并不支持容器相关的特性。

## java基础知识

JVM默认的最大堆内存大小为系统内存的1/4，可以使用参数`-XX:MaxRAMFraction=1`表示将所有可用内存作为最大堆。

cgroup的限制在docker中能够看到，通过查看/sys/fs/cgroup目录下的文件可以获取。

JVM的用户地址空间分为JVM数据区和direct memory。JVM数据区由heap、stack等组成，GC是操作的这一片内存。direct memory是额外划分出来的一片内存空间，需要手工管理内存的申请和释放。

direct memory使用`Unsafe.allocateMemory`和`Unsafe.setMemory`来申请和设置内存，是直接使用了C语言中的malloc来申请内存。由jvm参数`MaxDirectMemorySize`来限制direct memory可使用的内存大小。

## java < 8u131

没有对容器的任何支持，对cpu和内存的限制需要通过jvm的参数来配置。

java中并不能看到内存资源的限制，会存在使用内存超过限制而被OOM的问题。可通过在程序中设置`-Xmx`来解决该问题。

JVM GC（垃圾对象回收）对Java程序执行性能有一定的影响。默认的JVM使用公式“ParallelGCThreads = (ncpus <= 8) ? ncpus : 3 + ((ncpus * 5) / 8)” 来计算做并行GC的线程数，其中ncpus是JVM发现的系统CPU个数。一旦容器中JVM发现了宿主机的CPU个数（通常比容器实际CPU限制多很多），这就会导致JVM启动过多的GC线程，直接的结果就导致GC性能下降。Java服务的感受就是延时增加，TP监控曲线突刺增加，吞吐量下降。

显式的传递JVM启动参数`-XX:ParallelGCThreads`告诉JVM应该启动几个并行GC线程。它的缺点是需要业务感知，为不同配置的容器传不同的JVM参数。

## java9 and java >= 8u131

增加了`XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap`参数来检查内存限制。JVM中可以看到cgroup中的内存限制。

可以根据容器中的cpu限制来动态设置GC线程数，不再需要单独设置`-XX:ParallelGCThreads`。

## java10

jvm参数`XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap`已经默认开启，但新增加`-XX:-UseContainerSupport`参数来更好支持容器，支持内存和cpu。

在开启`-XX:-UseContainerSupport`的同时，`XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap`会被关闭。

## ref

- [Java SE support for Docker CPU and memory limits](https://blogs.oracle.com/java-platform-group/java-se-support-for-docker-cpu-and-memory-limits)
- [美团容器平台架构及容器技术实践](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651749434&idx=1&sn=92dcd59d05984eaa036e7fa804fccf20&chksm=bd12a5778a652c61f4a181c1967dbcf120dd16a47f63a5779fbf931b476e6e712e02d7c7e3a3&mpshare=1&scene=1&srcid=1115JtuwzXeezCv5UkmOcrFw%23rd)
