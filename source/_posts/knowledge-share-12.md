---
title: 技术分享第12期
date: 2019-10-09 01:35:26
tags:
---

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/12.JPG)

距离上次的知识分享系列已经过去了半年之久，难以想象。在该系列的开始我就说过该系列是不定期的，果不食言，只是这次的不定期有些久😓。该系列会继续下去，但节奏仍然是不定期的，但应该不会间隔半年之巨了。

题图来自首钢工业遗址公园，首钢于2010年完成了搬迁到唐山市的工作，位于石景山的工厂被废弃。2019年国庆节前夕以公园的形式部分对外开放，跟普通公园不同的地方在于保留了很多之前的工厂建筑及大型生产机械，足够硬核，非常原汁原味。

2022年要举办的冬奥会也非常明智的将场地选择在了该公园，预计将来会有很多的场馆位于该公园内且对外开放。

国内很多的地市都有一些类似的建筑，比如90年代下岗潮之前的一些国企工厂，我知道的济南的机床厂就有很多个，但很多这些倒闭关门的工厂后来的建筑及地皮都给卖掉了，建筑物也直接给拆掉了，殊不知其衍生价值也是有不少的。位于北京酒仙桥的798就是个非常好的改造案例，798园区的建筑稍加改造，给很多艺术工作室提供了非常好的办公场地，也是城市中的一个亮眼的名片。

## 资源

1.[MessagePack](https://msgpack.org/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/messagepack.png)

json作为一种常见的数据序列化方案，存在占用空间过多、反序列化过于消耗cpu的问题。MessagePack是一种基于二进制的高效轻量的数据序列化方案，支持数据的压缩，支持丰富的编程语言。在上图中，可以看出原27字节的json数据转换为MessagePack后仅占用了18字节。

另外，据今日头条的压测，要比Thrift的二进制序列化方案更高效一些。

2.[GoAst Viewer](https://github.com/yuroyoro/goast-viewer)

![https://raw.githubusercontent.com/yuroyoro/goast-viewer/master/goast-viewer.png](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/goast-viewer.png)

Golang中的ast、parser、token包可用来对golang的源码进行语法分析，并构建出AST树。GoAst Viewer支持在线输入Golang源码来构建AST树。

3.[KubeEdge](https://kubeedge.io)

![https://github.com/kubeedge/kubeedge/blob/master/docs/images/kubeedge_arch.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/kubeedge_arch.png)

IoT目前正在大力发展，边缘设备的计算能力在逐渐增强，同时处理的数据量的需求正在快速增加，而数据中心的数据处理能力、网络带宽、扩展能力并没有太多的增强，未来势必会将部分计算能力下放到边缘设备，以降低数据中心的成本。

华为开源的KubeEdge为基于Kubernetes的边缘计算平台，支持边缘集群的编排和管理。

4.[Mycat](http://mycat.io/)

![http://mycat.io/index_files/mycat2.jpg](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/mycat.png)

国内开源的关系型数据库中间件，支持MySQL、Oracle等常见的关系型数据库。关系型数据库单表过大导致性能下降后的解决思路往往是分库分表，分库分表后需要增加中间件层来解决多个数据库多张表的数据增删改查问题，而Mycat是一个不错的解决方案。

当然Mycat的功能不仅限于此，比如支持跨库的两张表join、Mycat-eye可以来监控Mycat等，更多功能请参照官方网站。

5.[WeChat Format](https://lab.lyric.im/wxformat/)

一款转换markdown格式的文档为微信公众号排版的工具，排版比较精美，推荐一试。

6.[GitNote](https://gitnoteapp.com/zh/)

![https://gitnoteapp.com/imgs/gitnote.png](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/gitnote.png)

 一款基于git的笔记管理软件，可以将笔记存放到Github中，支持多种图床插件，支持多个平台（还没有移动端），支持富文本编辑和Markdown编辑。由于笔记是可以同步Github上的，可以做到永久保存和版本控制，而且笔记的存放目录和格式不受该工具的影响，可以说是完全没有侵入性，脱离了该工具仍然可以通过直接编辑git项目的方式来发布笔记。

7.[Vlang开源啦](http://www.vlang.org)

V语言宣布开源，从V语言的特性上看到了很多Golang的影子，并未看到耳目一新的特性。

8.[krew](https://github.com/kubernetes-sigs/krew)

一款kubectl的plunin管理工具，mac平台下有brew包管理工具，随着kubectl的plugin机制的成熟，plugin管理工具应运而生。

9.[cert-manager](https://github.com/jetstack/cert-manager)

运行在k8s上的证书管理工具，可以签发证书，基于CRD实现。

Let's Encrypt提供了免费的tls证书，但证书有有效期限制，过期后需要手工重新申请证书，cert-manager可以做到从Let's Encrypt自动申请证书，并过期后重新申请证书。

10.[cmatrix](https://github.com/abishekvashok/cmatrix)

![https://github.com/abishekvashok/cmatrix/raw/master/data/img/capture_orig.gif?raw=true](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/capture_orig.gif)

一款黑客帝国效果的命令行工具，除了炫酷也没啥其他用途了。

11.[boxes](https://boxes.thomasjensen.com/)

boxes为一款有趣的命令行工具，可以显示很多神奇效果。

```
          __   _,--="=--,_   __
         /  \."    .-.    "./  \
        /  ,/  _   : :   _  \/` \
        \  `| /o\  :_:  /o\ |\__/
         `-'| :="~` _ `~"=: |
            \`     (_)     `/
     .-"-.   \      |      /   .-"-.
.---{     }--|  /,.-'-.,\  |--{     }---.
 )  (_)_)_)  \_/`~-===-~`\_/  (_(_(_)  (
(        Different all twisty a         )
 )         of in maze are you,         (
(           passages little.            )
 )                                     (
'---------------------------------------'
```

12.[rally](https://github.com/elastic/rally)

一款elasticsearch的压测工具。

13.[tekton](https://tekton.dev)

![https://tekton.dev/img/logos/tekton-horizontal-color.png](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/tekton-horizontal-color.png)

Google开源的一款基于Kubernetes的应用发布框架，Google在云原生生态中出品一般质量都比较高，主要用来做CI/CD。

14.[kubectl-debug](https://github.com/aylei/kubectl-debug)

一款基于kubectl插件的debug工具，基础镜像使用nicolaka/netshoot(内置了大量的网络排查工具)，可用于kubernetes集群中快速定位问题。值得一提的是，该工具的初版是作者在参加pingcap面试时的小作业。

15.[Monocular](https://github.com/helm/monocular)

Rancher出品的一款基于管理helm chart的ui工具。

16.[gitmoji](https://gitmoji.carloscuesta.me)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/gitmoji.jpg)

github上的开源项目中经常会看到一些git commit message中包含了moji表情，而且有越来越多的趋势，这些moji表情不紧紧是好玩，而且还非常生动形象的表达了commit message的含义，并且非常醒目，但这些moji表情可不应该是滥用的。该网站记录了一些常用的moji表情在git commit中的含义。

## 精彩文章

1.[Monitoring and Tuning the Linux Networking Stack: Receiving Data](https://blog.packagecloud.io/eng/2016/06/22/monitoring-tuning-linux-networking-stack-receiving-data/)

本文讲解了一个数据包到达网卡后是怎么一步步从网卡 -> 操作系统 -> 应用程序，并讲解了Linux中的实现方式。

绝大多数的工程师对于这一块的知识是较为模糊的，建议一读。

## 视频

https://mp.weixin.qq.com/s?__biz=MzU3OTc1Njk4MQ==&mid=2247486851&idx=1&sn=d0322f6d1a59c21e977488d9701d0476&chksm=fd607b59ca17f24fb07f60b8f488e602ac7e75302c999c206938682387e1d3cac44feb67b208&mpshare=1&scene=1&srcid=%23rd

半年多前比较火的视频，但我还是经常会想起来，给大家重温一下。

视频中为杭州一小伙深夜骑车逆行被交警拦下后情绪崩溃，失声痛哭。小伙每晚加班到十一二点，一方面女朋友在催着给送药匙，另一方面公司还在催着赶回公司，再加上被交警拦下，最终来自三方面的催促导致积压在小伙内心长久以来的压力爆发而情绪失控。隔着屏幕都能感受到小伙长期以来的压力，我猜想如果给他一些自由的时间，他一定会选择独自一人到一个安静的地方过上一段时间与世隔绝的生活。

生活本不易，在觉大多数的成年人生活中没有简单二字，祝愿各位生活如意！

