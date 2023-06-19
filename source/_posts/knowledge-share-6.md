---
title: 知识分享第6期
date: 2018-11-17 00:01:06
tags:
---
![](https://kuring.oss-cn-beijing.aliyuncs.com/images/leaves.jpeg)

题图为公司楼下公园的杨树林。时光易逝弹指间，又到一年叶落时。

## 资源

1.[runV](https://github.com/hyperhq/runv)

基于 hypervisor 的 OCI runtime

2.[operator-sdk](https://github.com/operator-framework/operator-sdk)

operator机制利用CRD机制增强了kubernetes的灵活性，但operator的编写代码很多模式都是固定的，该项目提供了更高层次的抽象。

3.[orchestrator](https://github.com/github/orchestrator)

用来管理mysql的集群拓扑和故障自动转移的工具。

4.[Tars](https://github.com/TarsCloud/Tars)

![https://github.com/TarsCloud/Tars/blob/master/docs/images/tars_jiaohu.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/tars.png)

腾讯开源的RPC框架，在腾讯内部已经有多年的使用历史，目前支持多种语言。

5.[ngx_http_dyups_module](https://github.com/yzprofile/ngx_http_dyups_module)

nginx module，可以提供Restful API的形式来动态修改upstream，而不用重新reload nginx。

6.[Dragonfly](https://github.com/alibaba/Dragonfly)

![https://github.com/alibaba/Dragonfly/raw/master/docs/images/logo/dragonfly-linear.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/dragonfly.png)

阿里巴巴开源的基于P2P的容器镜像分发系统。

7.[SonarQube](https://www.sonarqube.org/)

开源的代码检查和扫描工具，支持多种语言，并提供了友好的web界面用来查看分析结果。

8.[QUIC](https://www.chromium.org/quic)

QUIC是Google开发的基于UDP的传输层协议，提供了像TCP一样的数据可靠性，但降低了数据的传输延时，并具有灵活的拥塞控制和流量控制。

9.[OpenMessaging](http://openmessaging.cloud/)

阿里巴巴发起的分布式消息的应用开发标准，目前github上的star数还较少。

10.[nsenter](https://github.com/jpetazzo/nsenter)

nsenter是一个命令行工具，用来进入到进程的linux namespace中。

docker提供了exec命令可以进入到容器中，nsenter具有跟docker exec差不多的执行效果，但是更底层，特别是docker daemon进程异常的时候，nsenter的作用就显示出来了，因此可以用于排查线上的docker问题。

## 精彩文章

1.[为何程序员永远是高薪行业](http://mp.weixin.qq.com/s?__biz=MzIxNzYxMTU0OQ==&mid=2247486289&idx=1&sn=36950b6c33abbd0fd34dc04c231a1444&chksm=97f66723a081ee35c294fb332f5a530a29d5d69b39b6b77b128efd1260b2222bba622ec3c013&mpshare=1&scene=1&srcid=1026F66EebJRZ6bqonnBKBVi%23rd)

从记者的视角来了解阿里云的历史。

2.[Harbor传奇（1）- Harbor前世](https://mp.weixin.qq.com/s?__biz=MzIwODAzNTA2NQ==&mid=2651219445&idx=1&sn=2cf373851086355ae4ad26cd741480cf&chksm=8cfb8863bb8c01759b536ce04af5d04d8f149701626caa97e73a4012ba059584d63737cf8a3e&mpshare=1&scene=1&srcid=1029OVgZCxr32GyIFbfqKuEs%23rd)

3.[蚂蚁金服 Service Mesh 实践探索](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651010202&idx=1&sn=742179879a25d526402a5b561b769ed1&chksm=bdbeccc98ac945df391f1b54f06495868a683002ac9fb71a80fc001e10344a991d36019ad1f4&mpshare=1&scene=1&srcid=1031sNSkczy7L8lzRXMgWXlv%23rd)

4.[美团容器平台架构及容器技术实践](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651749434&idx=1&sn=92dcd59d05984eaa036e7fa804fccf20&chksm=bd12a5778a652c61f4a181c1967dbcf120dd16a47f63a5779fbf931b476e6e712e02d7c7e3a3&mpshare=1&scene=1&srcid=1115JtuwzXeezCv5UkmOcrFw%23rd)

美团内部的容器平台HULK已经从第一代的自研升级为第二代的基于kubernetes的容器管理平台。由此可以反映出kubernetes在容器管理领域的地位。

5.[Serverless：后端小程序的未来](https://mp.weixin.qq.com/s?__biz=MzAxOTAzMDEwMA==&mid=2652507564&idx=1&sn=cfedc29419e54987803197d8b975df62&chksm=8023e497b7546d81d4b68778c78cd3cc8fb7d0e746e385119c7b9e332ec599495da3017d0cda&mpshare=1&scene=1&srcid=1111C1S9Vcyb1bDDgLiCVIfS%23rd)

Serverless是未来软件架构的一个演进方向，包括BasS（Backend as a Service，后端即服务）和FaaS（Functions as a Service，函数即服务）两个组成部分。

BaaS包括对象存储、数据库、消息队列等服务，并以API的形式提供应用依赖的后端服务。

FaaS中的运行是通过事件触发的方式，代码执行完成后即运行结束，因此代码必须是无状态的。FaaS平台负责服务的自动扩容，并可做到按照服务的使用资源付费，以节省大量开支。

Serverless给开发人员带来了非常大的便利性，但同时也软件跟云平台绑定特别紧密。

## 图书

1.《[奈飞文化手册:“硅谷重要文件”的深度解读](https://www.amazon.cn/dp/B07J34BKTL)》

![https://images-na.ssl-images-amazon.com/images/I/51UmLKXW9%2BL._SX366_BO1,204,203,200_.jpg](https://kuring.oss-cn-beijing.aliyuncs.com/images/netflix_powerful.jpg)

Netflix公司的技术文化一直非常被业界推崇，可以从[Netflix OSS](https://netflix.github.io/)已经开源的软件项目，很多的开源项目在社区也有不错的影响力，本书值得每一位技术从业者一读。

## 精彩句子

> 我们要求大家做出的任何举动，出发点都是以对客户和公司最有利为出发点，而不是试图证明自己正确。

\- 奈飞文化手册:“硅谷重要文件”的深度解读
