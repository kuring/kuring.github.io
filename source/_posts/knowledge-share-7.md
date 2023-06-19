---
title: 知识分享第7期
date: 2018-12-06 23:58:09
tags:
---

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/jinshanling.jpeg)

题图为金山岭长城，明代著名抗倭名将戚继光从南方调任至此修筑，为明长城之精华，

## 资源

1.[GoAccess](https://goaccess.io/)

![http://rt.goaccess.io/?20180926071813&ref=hpimg](https://kuring.oss-cn-beijing.aliyuncs.com/images/goaccess-bright.png)

一款开源的实时分析nginx日志的工具，并拥有一个比较强大的dashboard。

2.[Wayne](https://github.com/Qihoo360/wayne)

![https://raw.githubusercontent.com/wiki/Qihoo360/wayne/image/dashboard-ui.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/wayne-dashboard-ui.png)

360开源的kubernetes的多集群管理平台。

3.[MacKey](http://www.mackeynote.com/)

一个分享KeyNote模版的网站，每个KeyNote模版都带有动画和图片截图。

4.[Nomad](https://github.com/hashicorp/nomad)

Hashicorp公司开源的集群调度工具，该公司另一款较为出名的产品为Vagrant。

5.[registrator](https://github.com/gliderlabs/registrator)

该服务部署在宿主机上，自动将docker的容器注册到服务注册中心中，如consul、etcd等。

6.[CNI-Genie](https://github.com/Huawei-PaaS/CNI-Genie)

华为开源的容器网络解决方案，CNI（Container Network Interface）仅支持加载一个插件，该插件可以同时一次加载多个网络插件，在容器中可以同时存在多个网络解决方案的ip。

7.[stress-ng](http://manpages.ubuntu.com/manpages/bionic/man1/stress-ng.1.html)

Linux下有一个命令行的压测测试工具stress，可以用来测试cpu、内存、io等，stress-ng提供了更丰富的选项。

8.[Resilience4j](https://github.com/resilience4j/resilience4j)

java版的开源熔断工具Hystrix宣布停止开发，并推荐了Resilience4j工具，该工具灵感来自于Hystrix，主要为java 8和函数式编程设计的自动熔断工具。

9.[Standard Go Project Layout](https://github.com/golang-standards/project-layout)

我刚开始写go的时候，一度被golang的源码目录结构所困惑，这个项目提供了一个标准的goalng目录结构的用法，很多开源项目都是按照这个标准组织的。

10.[dive](https://github.com/wagoodman/dive)

![https://github.com/wagoodman/dive/blob/master/.data/demo.gif](https://kuring.oss-cn-beijing.aliyuncs.com/images/dive.gif)

docker images不是一个单独的文件存储在宿主机上，而是采用分层设计，以便于多个镜像之间复用相同的层数据。dive可以用来分析docker image的每一层的具体组成。

11.[Swoole](https://www.swoole.com/)

php号称是世界上最好的编程语言之一，但最为人诟病的是其网络模型是同步模型，导致其性能一直上不去。Swoole可以实现类似于Golang中的goroutine同步编程模型来实现异步的功能。

## 精彩文章

1.[知乎社区核心业务 Golang 化实践](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653550448&idx=1&sn=e2ab178782f3fc1a6423f44c5b71afe3&chksm=813a66e8b64deffeec07664c15a315eb570ece4a3415f6ab06b4e9a15e4608dbd9838d5af071&mpshare=1&scene=1&srcid=1204G7pzjRvrQz0iFiDeYUfx%23rd)

本文记录了知乎内部使用golang来重构python的实践经验，用来解决python编程语言的运行效率低和维护成本高的问题。

2.[如何在Docker内部使用gdb调试器](https://mp.weixin.qq.com/s?__biz=MzI0NjI4MDg5MQ==&mid=2715292188&idx=1&sn=2b7f26203aa594027550e324460bc901&chksm=cd6d15c8fa1a9cde757868fd34c8336433c4877d3e7689ed0a2bd90eb1ef6271bda97aa3bb03&mpshare=1&scene=1&srcid=12045vIwpmKLu97HvFOssitt%23rd)

本文记录了一些docker关于权限相关的技术实现。

3.[ofo剧中人：我不愿谢幕](https://mp.weixin.qq.com/s?__biz=MzA3NDI0ODMzMw==&mid=2651302666&idx=1&sn=1ada632809e5c2a3895214a3590e5a0c&chksm=84f1b2e8b3863bfea2dfa0868e930ed3cf2b3f0860aeec5b8bbc2f6e0e73a20c7670b3bfc93f&mpshare=1&scene=1&srcid=1206BiRCjxkExnHMuFyQX0JM%23rd)

以记者的角度记录了OFO的发家、辉煌、衰败，曾有过彷徨与迷茫，曾有过野性与嚣张，但最终还是要倒在资本面前。

大家都在吐槽OFO押金退不了的事情，看到一个评论中的不错的点子，可以在OFO的退押金页面增加广告位，毕竟流量就是金钱，退押金页面的流量也是流量，反正押金也退不了，不如借此来一波，至少比在公众号中卖蜂蜜要好的多。

> 一个生动的细节是，有黑摩的司机不爽共享单车影响他们生意，砸ofo的车。ofo后期转化了一批相当数量的司机当修车师傅，化干戈为玉帛。

上述操作还是非常犀利的，说白了还是利益在作怪。

