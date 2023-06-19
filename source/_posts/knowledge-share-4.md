---
title: 知识分享第4期
date: 2018-10-14 01:13:02
tags:
---

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/laiwushi.jpeg)

今年以来，网络上一直在传言济南市要吞并莱芜市的消息，最近几天尤甚。市民们纷纷去市政府门前拍照留念，纪念莱芜市的最后一天，虽然到今天为止传言还未变成现实，但应该是迟早要到来的。

有趣的是，莱芜市是1993才从泰安市中独立出来，很多人都感慨道：出生是泰安人，长大是莱芜人，明天变成济南人。地理位置上而言，莱芜跟济南搭界，而且莱芜市是山东省17地市中面积最小的一个。

山东的发展策略一直是各地市全面发展，济南市作为省会，在中国城市中的存在感确实不够强，吞并莱芜后也一样不会变强太多，一个地市要想变强，要从多方面找原因。

## 资源

1.[k8s-deployment-strategies](https://github.com/ContainerSolutions/k8s-deployment-strategies)

kubernetes内置的deployment和statefulset对象往往很难满足企业的部署需求，比如蓝绿发布、金丝雀发布等，Github上的项目介绍了其他部署方式在kubernetes上的具体实现方式。

2.[Let's Encrypt](https://letsencrypt.org/)

https越来越普及，通常CA颁发的证书都是收费的，Let's Encrypt是一家非盈利的CA机构，为广大的小型站点和博客博主提供了非常大的帮助。Github pages中也是采用了Let's Encrypt来提供自定义域名的https服务支持。

3.[openmetrics](https://openmetrics.io/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/openmetrics.png)

监控领域存在多款开源软件，比如premetheus、influxdb、opentsdb等，每种软件的写入数据格式都不一致，该开源项目旨在定义监控数据的标准格式，目前支持premetheus的文本格式和protobuf两种格式。该项目目前还在起步阶段，已经加入CNCF，期待后续一统行业标准。

4.[NATS](https://github.com/nats-io)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/nats.png)

Go语言实现的消息队列，目前已经加入CNCF。

5.[sequel fumpt](https://sqlfum.pt/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/sqlfum.png)

sql的在线格式化工具。

6.[The Linux Audit Project](https://github.com/linux-audit)

Linux下的日志审计工具，CentOS系统下默认安装，可以通过`man auditd`看到该工具的说明。

7.[kafkabridge](https://github.com/Qihoo360/kafkabridge)

360开源的kafka客户端库的封装，只需调用极少量的接口，就可完成消息的生产和消费。支持多种语言：c++/c、php、python、golang。

## 精彩文章

1.Keyhole,Google Maps发展史

文章为微信公众号*余晟以为*的系列文章，大部分素材来源于[《Never Lost Again》](https://book.douban.com/subject/30243035/)一书，该书作者为Bill Kilday，Keyhole和Google Maps团队的核心成员。文章介绍了Google Maps的前身Keyhole的创业史，后被Google收购后，又推出了基于web的Google Maps产品，继而开发出了Google Earth产品。即使在Google内部，也存在团队之间的孤立及不信任问题。

* [Keyhole，Google Maps前传](https://mp.weixin.qq.com/s/P6IVAAux9N3tNb0QNp4XeQ)
* [Google Maps，Keyhole后传](https://mp.weixin.qq.com/s/-F4XO-NyOqn2wSXlM4CSyA)
* [从Google Maps到Google Earth](https://mp.weixin.qq.com/s/X-XZcSTsKd6Pzv5RzjrFyA)

2.[Kubernetes 调度器介绍](https://mp.weixin.qq.com/s?__biz=MzU4MjQ0MTU4Ng==&mid=2247483917&idx=1&sn=e272bd1d91f148f879b602d6640efb5d&chksm=fdb90d10cace840608fa21e090e5dfbc5c95894151998c4998b13e13b31b423088b61bb4873d&mpshare=1&scene=1&srcid=1011azypmKUvqeurakroNuxT%23rd)

文章对kubernetes的kube-scheduler的整体流程介绍的比较清晰。

3.[A Brief History of Alibaba Founders](https://iprice.sg/trends/insights/history-jack-ma-alibaba-18-founders/)

阿里巴巴的18罗汉介绍。

## 图片

1.电传打字机设备(Teletype)

![tty](https://kuring.oss-cn-beijing.aliyuncs.com/images/tty.png)

早期的计算机设备比较笨重，计算机放在单独的一个房间中，操作计算机的人坐在另外一个房间中，通过终端机设备来操作计算机。

早期的终端设备为电传打字机(Teletype)，该设备价格比较低廉，通过键盘输入，并将输出内容打印出来。图中的设备为ASR-33，在YouTube上可以可以看到[视频](https://www.youtube.com/watch?v=MikoF6KZjm0)。

有意思的是，实际上Teletype的出现要早于计算机，原本用于在电报线路上发送电报，但是后来计算机出现后直接拿来作为计算机的终端设备。

在linux操作系统中设计了tty子系统用于支持tty设备，并将具体的硬件设备放到/dev/tty*目录下，这里的tty设备即Teletype。不过后来随着其他终端设备的引入，tty这个名字仍旧保留了下来，tty目前已基本代表终端的总称。

