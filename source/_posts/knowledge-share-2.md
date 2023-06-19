---
title: 知识分享第2期
date: 2018-09-09 01:21:03
tags: 知识分享
---

![北京灵山](https://kuring.oss-cn-beijing.aliyuncs.com/images/lingshan.jpeg)

题图为北京灵山主峰，海拔2303米，北京最高峰。登上主峰的时候，恰巧一头牛就在山顶悠闲，拍照的时候，牛哥把我带上去的枣、葡萄、花生米全部吃光了，甚至连橘子皮都没剩下，吃完后牛哥又悠闲的去吃草了，让我见识了啥叫吃葡萄不吐葡萄皮。

## 教程

1. 《Kubernetes权威指南 企业级容器云实战》

kubernetes的书籍并不多，该书八月份刚初版，内容较新，并不是一本kubernetes的入门书籍，而是讲解kubernetes在企业落地为PASS平台时需要做的工作，建议对kubernetes有一定了解后再看。

书的内容为HPE的多名工程师拼凑而成，有些部分的内容明显是没有经过实践验证的理论派想法，但总体来看值得一读。

书中提到了很多kubernetes较新版本才有的特性、微服务、service mesh、lstio，对于补充自己已经掌握的知识点有一定帮助。

书的后半部分反而显的干货少了非常多，我仅草草的过了一遍。

2. [《Go语言高级编程》](https://chai2010.gitbooks.io/advanced-go-programming-book/content/)

Golang中相对进阶的中文教程，我还没来得及看。

3. [Gorilla](https://www.vldb.org/pvldb/vol8/p1816-teller.pdf)

facebook发表的分布式的时序数据库论文，如果英文看起来吃力，可以看一下小米运维公众号中的[翻译版本](https://mp.weixin.qq.com/s/-1jpuCm20LSTrVU7nTw5_Q)。facebook并未提供开源的实现，但在github上能找到一些开源的实现。

4. 《深入剖析Kubernetes》

极客时间app的专栏，本来购买之前没有报特别大的预期，但读完头几篇文章后被作者的文字功底折服，将PASS、容器的来龙去脉、docker的发展讲解的很到位，超出了我的预期。期待后面更新的专栏能够保持搞水准。

5. [SDN手册](https://tonydeng.github.io/sdn-handbook)

一本介绍SDN相关知识的开源电子书。

## 资源

1. [M3DB](https://github.com/m3db)

监控领域还是比较缺少特别好用的分布式时间序列存储数据库，性能特别优异的数据库往往都是单机版的，缺少高可用的方案，比如rrdtool、influxdb、graphite等。OpenTSDB、KairosDB、Druid等虽为分布式的时序数据库，但使用或者运维起来总有各种不方便的地方。uber开源的m3在分布式时序数据库领域又多了一个方案，并可作为prometheus的远程存储。

2. [cilium](https://cilium.io/)

使用BPF(Berkeley Packet Filter)和XDP(eXpress Data Path)内核技术来提供网络安全控制的高性能开源网络方案。

3. [kubeless](https://github.com/kubeless/kubeless)

kubernetes平台上的Serverless项目，Faas（功能即服务）一定是云计算发展的一个趋势。目前CNCF中还没有Serverless项目，期待CNCF下能够孵化一个Serverless项目。

## 工具

1. vagrant

还在使用virturalbox的你，是时候使用vagrant了。vagrant作为对虚拟机的管理，虽然引入了一些概念带来了更大的复杂性。但同时功能上也更强大，比如对box的管理，可以将box理解为docker image，便于将虚拟机的环境在不同的主机上分发。

## 公众账号推荐

1. 小米运维

开通时间不算特别长，但文章的质量不错，都是比较接地气的干货，看得出确实是在工作中遇到的问题或者是总结经验，值的一读。

![](https://open.weixin.qq.com/qr/code?username=MI-SRE)

2. 开柒

曾经公众号的名字为开八，江湖人称八姐，忘记为何更改为开柒了，曾经的搜狐记者。总能非常及时的爆料很多互联网的内幕，消息来源往往非常准确，可见八姐在圈内的人脉非同一般。

![](https://open.weixin.qq.com/qr/code?username=hlkaiba)

3. 毕导

打发时间非常好的公众号，用理科男的思维方式进行恶搞，是不是拿出冗长的数学公式来证明日常生活中的小尝试，语言诙谐幽默，绝对是公众号中的一股清流。可惜每篇文章都很长，我没有太多时间把每一篇文章都看一遍。

![](https://open.weixin.qq.com/qr/code?username=bxt_thu)

