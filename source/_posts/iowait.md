---
title: linux iowait
date: 2018-12-08 22:58:53
tags:
---

iowait和load一样，都是非常容易让人产生误解的系统指标。

iowait表示cpu空闲且有未完成的io请求的时间，iowait高并不能反映出磁盘是系统的性能瓶颈。iowait高的时候cpu正处于空闲状态，没有任务可以执行。此时存在已经发出的磁盘io，此时的cpu空闲状态称之为iowait。本质上，iowait是一种特殊的cpu空闲状态。

iowait状态的cpu是运行在pid为0的idle线程上。

cpu此时之所以进入睡眠状态，是因为进程处于睡眠状态，在等待某个特定的事件（比如网络数据，io操作完成等）。

iowait仅能反应磁盘io的指标，并不能反应其他io设备的指标，比如网络丢包。

在io wait的进程处于不可中断状态，通过top命令可以看到进程状态为

由此可见，iowait包含的信息量非常少，仅凭iowait升高不能判断出系统io有问题。要想判断系统io有问题，还需要使用iostat等命令来查看系统的svctm、util、avgqu-sz等指标。

## case 1

![http://linuxperf.com/wp-content/uploads/2015/02/iowait1.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/iowait1.png)

仅cpu的繁忙程度变化的情况下，会影响到iowait的值。

## case 2

![http://linuxperf.com/wp-content/uploads/2015/02/iowait.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/iowait2.png)

在cpu繁忙程序不变的情况下，发起io请求的时间不同也会影响到iowait的值。
