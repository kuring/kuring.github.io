---
title: 技术分享第16期
date: 2022-07-23 16:09:08
tags: 
author:
---

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/header-16.jpg)

题图为望京傍晚的天气，夕阳尽情散发着落山前的最后余光，层次分明的云朵映射在建筑物的上熠熠生辉。上班族结束了一天紧张的工作，朝着地铁站的方向奔向自己的家，这才是城市生活该有的模样。不过可惜的是，对于很多打工族而言，一天的工作还远未结束，晚饭后仍要坐在灯火通明的写字楼内或为生活或为梦想挥霍着自己的时光。

# 资源

## [kaniko](https://github.com/GoogleContainerTools/kaniko)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/kaniko.png)

Google开源的一款可以在容器内部通过Dockerfile构建docker镜像的工具。

我们知道`docker build`命令可以根据Dockerfile构建出docker镜像，但该操作实际上是由docker daemon进程完成。如果`docker build`命令在docker容器中执行，由于容器中并没有docker daemon进程，因此直接执行`docker build`肯定会失败。

kaniko则重新实现了Dockerfile构建镜像的功能，使得构建镜像不再依赖docker daemon。随着gitops的技术普及，CI工具也正逐渐on k8s部署，kaniko正好可以在k8s的环境中根据Dockerfile完成镜像的打包过程，并将镜像推送到镜像仓库中。

## [arc42](https://arc42.org/overview)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/arc42-overview-V8.png)

技术人员在写架构文档的时候，遇到最多的问题是该如何组织技术文档的结构，arc42 提供了架构文档的模板，将架构文档分为了 12 个章节，每个章节又包含了多个子章节，用来帮助技术人员更好的编写架构文档。

相关链接：https://topic.atatech.org/articles/205083?spm=ata.21736010.0.0.18c23b50NAifwr#tF1lZkHm

## [Carina](https://github.com/carina-io/carina/blob/main/README_zh.md)

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/carina.png)

国内云厂商博云发起的一款基于 Kubernetes CSI 标准实现的存储插件，用来管理本地的存储资源，支持本地磁盘的整盘或者LVM方案来管理存储。同时，还包含了Raid管理、磁盘限速、容灾转移等高级特性。

相关链接：[一篇看懂 Carina 全貌](https://mp.weixin.qq.com/s/-435K5O780NS2gkuLvSr5g)

## [kube-capacity](https://github.com/robscott/kube-capacity)

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/kube-capacity.png)

k8s的命令行工具kubectl用来查看集群的整体资源情况往往操作会比较复杂，可能需要多条命令配合在一起才能拿得到想要的结果。kube-capacity命令行工具用来快速查看集群中的资源使用情况，包括node、pod维度。

相关链接：[Check Kubernetes Resource Requests, Limits, and Utilization with Kube-capacity CLI](https://able8.medium.com/check-kubernetes-resource-reqeusts-limits-and-utilization-with-kube-capacity-cli-b00bf2f4acc9)

## [Kubeprober](https://k.erda.cloud/)

在k8s集群运维的过程中，诊断能力非常重要，可用来快速的定位发现问题。Kubeprober为一款定位为k8s多集群的诊断框架，提供了非常好的扩展性来接入诊断项，诊断结果可以通过grafana来统一展示。

社区里类似的解决方案还有Kubehealthy和Kubeeye。

相关链接：[用更云原生的方式做诊断｜大规模 K8s 集群诊断利器深度解析](https://mp.weixin.qq.com/s/Wte75OfQ7Ihzlm4th-pNYA)


## [Open Policy Agent](https://www.openpolicyagent.org/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/opa.png)

OPA为一款开源的基于Rego语言的通用策略引擎，CNCF的毕业项目，可以用来实现一些基于策略的安全防护。比如在k8s中，要求pod的镜像必须为某个特定的registry，用户可以编写策略，一旦pod创建，OPA的gatekeeper组件通过webhook的方式来执行策略校验，一旦校验失败从而会导致pod创建失败。

比如 [阿里云的ACK的gatekeeper](https://help.aliyun.com/document_detail/180803.html?spm=ata.21736010.0.0.3d7e50fddLMBB9) 就是基于OPA的实现。

## [docker-squash](https://github.com/goldmann/docker-squash)

docker-squash为一款docker镜像压缩工具。在使用Dockerfile来构建镜像时，会产生很多的docker镜像层，当Dockerfile中的命令过多时，会产生大量的docker镜像层，从而导致docker镜像过大。该工具可以将镜像进行按照层合并压缩，从而减小镜像的体积。

## [FlowUs](https://flowus.cn/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/flowus.jpg)

FlowUs为国内研发的一款在线编辑器，支持文档、表格和网盘功能，该软件可以实现笔记、项目管理、共享文件等功能，跟蚂蚁集团的产品《[语雀](https://www.yuque.com/)》功能比较类似。但相比语雀做的好的地方在于，FlowUs通过”块编辑器“的方式，在FlowUs看来所有的文档形式都是”块“，作者可以在文档中随意放置各种类型的”块“，在同一个文档中即可以有功能完善的表格，也可以有网盘。而语雀要实现一个相对完整的表格，需要新建一种表格类型的文档，类似于Word和Excel。

## [k8tz](https://github.com/k8tz/k8tz)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/k8tz.png)

k8s中的pod默认的时区跟pod的镜像有关，跟pod宿主机所在的时区没有关系。很多情况下，用户都期望pod里看到的时区能够跟宿主机的保持一致。用户的一种实现方式是将宿主机的时区文件挂载到pod中，但需要修改pod的yaml文件。本工具可以通过webhook的方式自动化将宿主机的时区文件挂载到pod中。


# 文章

1. [中美云巨头歧路，中国云未来增长点在哪？](https://mp.weixin.qq.com/s/4ufpUSq2Qn_QV5vIJcPgqg)

文章结合全球的云计算行业，对国内的云计算行业做了非常透彻的分析。”全球云，看中美；中美云，看六大云“，推荐阅读。

2. [程序员必备的思维能力：结构化思维](https://mp.weixin.qq.com/s/F0KoDD9er7MNKYo-5POfsA)

结构化思维不仅对于程序员，对于职场中的很多职业都非常重要，无论是沟通、汇报、晋升，还是写代码结构化思维都非常重要。本文深度剖析了金字塔原理以及如何应用，非常值得一读。文章的作者将公众号的文章整理为了《程序员底层思维》一书，推荐大家阅读。

3. [中文技术文档的写作规范](https://github.com/ruanyf/document-style-guide)

阮一峰老师的中文技术文档写作规范，写技术文档的同学可以参考。

# 书籍

1. [《程序员的底层思维》](https://book.douban.com/subject/35794819/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/dicengsiwei.jpg)

通过书名中的“程序员”来看有点初级，但实际上书中的内容适合所有软件行业的从业者，甚至同样适合于其他行业的从业者，因为底层思维本来就是共性的东西，万变不离其宗。作者曾在阿里巴巴有过很长一段工作经历，书中结合着工作中的实践经验介绍了16种思维能力，讲解浅显易懂，部分内容上升到了哲学的角度来讲解。

作为软件行业从业者的我，实际上书中的大部分思维能力在工作中都有应用，但却没有形成理论来总结。阅读本书，有助于对工作的内容进行总结，找到工作的理论基础。另一方面，有了书中的理论总结，也可以更好的指导工作。