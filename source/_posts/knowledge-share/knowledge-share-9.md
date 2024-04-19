---
title: 知识分享第9期
date: 2019-01-19 20:18:13
tags:
---

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/fujishan.jpeg)

富士山&富士吉田市，富士山的海拔高达3776米，远在80公里外的东京都能够看到。令人称奇的是，富士山海拔3360米以上的土地并不是归日本政府所有，而是归富士山上的浅间寺所有，日本政府每年都要支付大量的租金给浅间寺。在富士山周边游览后，突然萌生了登顶富士山的想法，不知是否有志同道合的驴友，可以相约在某年的夏季去一起实现梦想。

## 资源

1.[CRIU](https://criu.org/Main_Page)

Linux下的一款实现checkpoint/restore功能的软件，该软件可以冻结某个正在运行的应用程序，并将应用程序的当前状态作为checkpoint存放在磁盘上的文件中，此后正在运行的应用程序会被kill。

此后，可以通过读取磁盘上的文件，恢复之前冻结的应用程序继续执行，而不是从main函数开始执行。

2.[bindfs](https://bindfs.org/)

将一个目录mount到另外一个目录的工具，利用该命令可以将docker中的路径挂载到宿主机上。具体操作命令类似如下：

```
PID=$(docker inspect b991b7ad105f --format {{.State.Pid}})
bindfs /proc/$PID/root /tmp/root
# 别忘了卸载目录
umount /tmp/root
```

3.[微软亚洲研究院-对联电脑](https://www.msra.cn/)

微软亚洲研究院的自动对对联系统，给出上联后，可以自动给出多个下联，最终生成横批。

4.[bcc](https://github.com/iovisor/bcc)

基于Linux eBPF的一系列的性能分析工具，包括IO、网络等多个方面。

5.[pcstat](https://github.com/tobert/pcstat)

基于golang开发的linux下的文件缓存统计工具。

6.[Electron](https://electronjs.org/)

利用前端技术（JavaScript、HTML、CSS）来构建桌面程序的框架，当前很多流行的桌面应用都是使用该技术来开发的，比如VSCode、Slack、Atom等技术。

得益于ES6、V8引擎和Node.js，JavaScript技术已经横跨前端、后端、桌面端的技术栈。

7.[Reading-and-comprehense-linux-Kernel-network-protocol-stack](https://github.com/y123456yz/Reading-and-comprehense-linux-Kernel-network-protocol-stack)

该项目包含了对Linux网络协议栈的源码中文注释，对阅读Linux网络协议栈的代码有一些帮助。

## 精彩文章

1. [Kubernetes API 与 Operator：不为人知的开发者战争（一）](https://mp.weixin.qq.com/s?__biz=MzA5OTAyNzQ2OA==&mid=2649699945&idx=1&sn=c5b32baf1ea063d908b547381d4c13da&chksm=8893090abfe4801caa8ae854b2eb2ef6f37d79a83f7147db74a5656616baac00d60c80053fdd&scene=21#wechat_redirect)
2. [Kubernetes API 与 Operator：不为人知的开发者战争（二）](https://mp.weixin.qq.com/s?__biz=MzA5OTAyNzQ2OA==&mid=2649700020&idx=1&sn=4e8b2ae1c1d9d0457eb4134b3624da96&chksm=889309d7bfe480c11030dc181a5e6c7b8998ae2850deed5637b4e4cd4a9c33339e87abab73ea&mpshare=1&scene=1&srcid=0108IZNbW8zHwqP9OH0anKe6%23rd)

## 精彩语句

1.
> “不能用”“不好用”“需要定制开发”，这才是落地开源基础设施项目的三大常态。

-- 张磊《深入剖析Kubernetes》

开源项目在落地到公司内部实际使用时，会发现有这样或者那样的问题。开源项目往往是个通用项目，公司在落地时，总有其特殊需求之处，开源软件无法面面俱到，往往只能覆盖一些通用的需求。再加上靠社区来驱动，在bug方面、功能方面跟商业软件也还有较大差距。

## 娱乐

1.《塞尔达传说-旷野之息》

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/BreathoftheWildFinalCover.jpg)

任天堂Switch上的游戏神作，历时四年时间，300人的团队开发，最近一直在玩，已经深深被游戏设计的海拉鲁大陆所折服，完全开放的世界，不同于传统的闯关类游戏，该游戏的自由度非常高，有时候就单纯的在地图中瞎逛都是一种享受，随时都会有惊喜发生。

曾天真的以为，一个单机游戏能好玩到哪里去，但在玩游戏的每一刻都能体会到制作团队的用心，心里总是念到这才是我想要玩的游戏。自从玩了该游戏后，手机上的游戏再也没有打开过。我甚至一度感叹，在国内快糙猛的环境下是产生不了如此细腻良心作品的。如果大家有机会，可以尝试下这款游戏，或许会发现单机游戏还可以做得如此出彩。

2.[ZELDA MAPS](https://zeldamaps.com)

同样是跟《塞尔达传说-旷野之息》相关的，由于塞尔达传说的地图实在过于庞大，包含了神庙、驿站、村庄、回忆（没错主人公Link失忆了）、各种支线任务、装备、呀哈哈、各类大小boss、迷宫等等，有玩家制作了一款在线的地图，可以在线查询地图中的各类元素，使用体验类似Google Map。还包含了账号体系，可以在地图上标记自己已经完成的任务。
