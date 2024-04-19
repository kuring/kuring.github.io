---
title: 知识分享第5期
date: 2018-10-26 21:02:27
tags:
---

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/tanzhesi.jpeg)

题图为北京城西部的潭柘寺，始建于西晋年间，有“先有潭柘寺，后有幽州城”的说法。

明朝燕王朱棣听取了重臣姚广孝的建议后，起兵“靖难”，并成功夺取皇位。朱棣继皇帝位后，姚广孝辞官到京西的潭柘寺隐居修行。据说当年修建北京城时，设计师就是姚广孝，他从潭柘寺的建筑和布局中获得了不少灵感。

## 资源

1.[virtual-kubelet](https://github.com/virtual-kubelet/virtual-kubelet)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/virtual-kubelet.svg)

很多公有云厂商都提供了弹性容器服务实例，比如阿里云的ECI（Elastic Container Instance）、AWS Fargate、Azure Container Instances等，但这些平台都提供了私有的API，与kubernetes的API不兼容。该项目将公有云厂商的的容器组虚拟为kubernetes集群中的一个超级node，以便支持kubernetes的API，与此同时失去了很多kubernetes的特性。

2.[Kata Containers](https://katacontainers.io/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/katacontainers.jpg)

容器在部署服务方面有得天独厚的优势，但受限于内核特性，在隔离性和安全性方面仍然较弱。虚拟机（VM）在隔离性和安全性方面都比较好，但启动速度和占用资源方面却不如容器。Kata Containers项目作为轻量级的虚拟机，但提供了快速的启动速度。同时支持Docker容器的OCI标准和kubernetes的CRI。目前华为公有云已经将此技术用于生产环境中。

3.[Knative](https://github.com/knative/)

在今年的Google Cloud Next大会上，Google发布了Knative, 这是由Google、Pivotal、Redhat和IBM等云厂商共同推出的Serverless开源工具组件，它与Istio，Kubernetes一起，形成了开源Serverless服务的三驾马车。

4.[naftis](https://github.com/XiaoMi/naftis)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/naftis.png)

小米信息部武汉研发中心开源的istio的dashboard。

5.[kubespy](https://github.com/pulumi/kubespy)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/kubespy.gif)

用来查看kubernetes中资源实时变化的命令行工具。

6.[Md2All](http://md.aclickall.com/)

如果你已经习惯了markdown写作，在微信公账号发文时，可以使用该工具渲染后，将文章复制到微信公众号后台。

## 精彩文章

1.[阿里云的这群疯子](https://mp.weixin.qq.com/s?__biz=MzU0NDEwMTc1MA==&mid=2247490379&idx=1&sn=17857e09e980b41bc188e592422c3459&chksm=fb001f52cc7796444cb18e5a483d3ad26e44fc70d543fa03fb2f56ae74d4db7d65e8d5df7b4e&mpshare=1&scene=1&srcid=1016QBaRRz9JrroPgKJ8xXBp%23rd)

从记者的视角来了解阿里云的历史。

2.[王垠最近博客-更新一下](http://www.yinwang.org/blog-cn/2018/10/14/update)

曾经以天才自居桀骜不驯傲视一切的垠神，突然变得温顺了许多，开始意识到自己的缺点，开始享受生活。

3.[为何“秀恩爱，死得快”？我是认真的](https://mp.weixin.qq.com/s/9vYnko3l8asVr2Mly5NjbQ)

用量子物理学的知识来解释为啥“秀恩爱，死得快”。

4.[CTO、技术总监、首席架构师的区别](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665515178&idx=1&sn=52cb7a6363a93a50fb5a6608fe0006a5&chksm=80d670e9b7a1f9ff3c99e444ad383905ea26793673ed6d220e889cfce66c833e4f48000e4525&mpshare=1&scene=1&srcid=1019RCPQVkN8CT6NF0xNe8Y5%23rd)

5.[面对云厂商插管吸血，MongoDB使出绝杀](https://mp.weixin.qq.com/s?__biz=MzI5OTM3MjMyNA==&mid=2247485701&idx=1&sn=ff6e2e6d3090555339bd93b111ae3d20&chksm=ec96d34edbe15a58a0aa8252defd77c378565e88bcf2d5f9a522043ecfc8302f6133f185f79b&mpshare=1&scene=1&srcid=1019jffSXvtK3BUofnCzPx9n%23rd)

半年后看下MongoDB的修改开源协议的做法在国内奏效否。

6.[终于明白了 K8S 亲和性调度](http://mp.weixin.qq.com/s?__biz=MzU4MjQ0MTU4Ng==&mid=2247483928&idx=1&sn=157497314c4c2a2f5ad93f4a0d4b20b7&chksm=fdb90d05cace8413f9e2470f6f8eacec4479031e7acc8acb63ae99c9b57a5e4d9a15c3d37db8&mpshare=1&scene=1&srcid=1024ksuGmq7gQIMXBcJPNFcF%23rd)

通过该文章，已经差不多可以了解kubernetes调度的亲和性、反亲和性、taint和toleration机制了。

7.[微软资深工程师详解 K8S 容器运行时](http://mp.weixin.qq.com/s?__biz=MzU1OTAzNzc5MQ==&mid=2247486496&idx=1&sn=0214f3d62b7d65350cd58ea6b4173184&chksm=fc1c2010cb6ba906a94f851184a15f462d16836c3ab5ab362ca5faeb40726f056a40261ccba6&mpshare=1&scene=1&srcid=1022SujMyNtcwnqLaT3fI3aM%23rd)


## 图书

1.[鸟哥的Linux私房菜：基础学习篇（第四版）](http://product.china-pub.com/8053114#ml)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/niaoge_linux.png)

鸟哥的linux私房菜终于出新版了，最新版本是基于CentOS7的。

## 电影

1.嗝嗝老师

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/hichki.jpg)

电影讲述了印度贫民窟中的孩子在学校上学总是遭受歧视不爱学习各种调皮捣蛋，在一位新老师来了后，将学生们带向正轨的故事。印度电影总能将平凡的电影演绎的很魔性，单就这些故事就已经足够了。偏偏这位老师还是抽动秽语综合征患者，在受到老师和学生们的双重歧视下，给故事情节增加了许多感人和励志色彩。

强迫症患者谨慎观看，看完电影后，总感觉得抽搐两下才舒服。

## 精彩句子

> 货币的贬值，是永恒的趋势。明天的物价，一定比今天的贵。
你想要赚钱，就一定要把今天的钱，换成明天的物。
而且时间越紧凑越好。

-- [八年之后 房价多少？](https://mp.weixin.qq.com/s?__biz=MzUxMDAwOTE4OQ==&mid=2247483881&idx=2&sn=1226590dd47dd171e833fee97e43d30d&chksm=f908cbe3ce7f42f57d07e6c060a7f607b2c11d7f0542a3215800e1610b65c5f1ad0ddfded56d&mpshare=1&scene=1&srcid=10197m4srqox3aXZOybWcQgm%23rd)

