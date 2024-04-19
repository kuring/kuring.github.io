---
title: 技术分享第13期
date: 2020-07-28 01:35:26
tags:
---

![望京SOHO](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/wangjing-soho.jpg)

又是很长的一段时间没有更新，果然又是不定期更新，文章的有些内容也是很久以前积累的，并不是因为太懒，而是确实没有太多的精力。

题图为雨中的望京SOHO，今年全国的雨水特别多，北京亦是如此。南方的鱼米之乡地区出现了严重的洪灾，不知道今年的粮食产量会受多大影响。我们的地球在人类翻天覆地的变更后实在经受不了太多的hack，愿雨季早日过去。

## 资源

1.[bocker](https://github.com/p8952/bocker)

bocker=bash + docker，其利用100多行bash代码实现的简易版的docker，使用到的底层技术跟docker是一致的，包括chroot、namespace、cgroup。

2.[kubectx](https://github.com/ahmetb/kubectx)

天天操作k8s的工程师一定少不了使用kubectl命令，而用kubectl命令的工程师一定会特别烦天天输入`-n ${namespace}`这样的操作，该工具可以省去输入namespace的操作。刚开始的时候不是太习惯该工具，直到近期才感知到该工具的价值。🤦‍♂️

3.[KubeOperator](https://kubeoperator.io/)

![https://raw.githubusercontent.com/KubeOperator/website/master/images/kubeoperator-ui.jpg](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/kubeoperator-ui.jpg)

k8s集群的安装操作基本上都是黑屏来完成的，同时集群规模较大时，还需要一些自动化的手段来解决安装和运维物理机的问题。KubeOperator提供了界面化的操作来完成k8s集群的配置、安装、升级等的操作，底层也是调用了ansible来作为自动化的工具。该项目已经加入CNCF，期望后面可以做的功能更加强大，给k8s集群的运维带来便利。

4.[awesome-operators](https://github.com/operator-framework/awesome-operators)

k8s生态的operator非常火爆，作为k8s扩展能力的一个重要组成部分，该项目汇总了常见的operator项目。

5.[chaos-mesh](https://github.com/pingcap/chaos-mesh)

pingcap开源的Kubernetes的混沌工程项目，可以使用CRD的方式来注入故障到Kubernetes集群中。

6.[devops-exercises](https://github.com/bregman-arie/devops-exercises)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/devops-exercises.jpg)

DevOps相关的一些面试题，涉及到的方面还是比较全的。

7.[shell2http](https://github.com/msoap/shell2http)

可以将shell脚本放到业务页面上执行的工具，在web页面上点击按钮后，会执行shell脚本，shell脚本的输出会在web页面上显示。

8.[Google Shell 风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-shell-styleguide/contents/)

Google编程规范还是比较有权威性的，此为Shell的编码规范。

9.[shellcheck](https://github.com/koalaman/shellcheck)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/shellcheck.png)

Shell作为弱类型的编程语言，稍有不慎还是非常容易写错语法的，至少很多的语法我是记不住的，每次都是边查语法边写🤦‍。该项目为Shell的静态检查工具，用来检查可能的语法错误，在Github上的start数量还是非常高的。

不仅支持命令行工具检查，而且还可以跟常用的编辑器集成（比如vim、vscode），用来实现边写边检查的效果。还提供了web界面，可以将shell脚本输入到web界面上来在线检查。

10.[teambition](https://www.teambition.com/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/teambition.png)

阿里的一款的远程协作工具，类似于国外slack+trello的结合版，在产品设计上能看到太多地方借鉴了trello，非常像是trello的本土化版本，更贴近国人的使用习惯，可用于管理团队和个人的任务。

11.[IcePanel](https://icepanel.io/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/icepanel.png)

IcePanel为vscode的一款插件，提供了k8s一些基础对象的编辑生成器，通过ui的界面即可生成k8s的ConfigMap、Deployment、Service等对象。

12. [Play with Kubernetes](https://labs.play-with-k8s.com/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/play-with-kubernetes.jpg)

一个提供在线的kubernetes集群的工具，在界面上点一下按钮就可以创建一个k8s集群，不需要注册，非常方便，但创建的集群只有四个小时的使用时间。可以用来熟悉k8s的基本操作，或者试验一些功能。

## 精彩文章

1.[腾讯自研业务上云：优化Kubernetes集群负载的技术方案探讨](https://mp.weixin.qq.com/s/FK9cYbGvzCLUsO7q63TrCA)

k8s虽然在服务器的资源利用率上比起传统的物理机或虚拟机部署服务方式有了非常大的提升，本文结合实践经验，从pod、node、hpa等多个维护来优化以便进一步的压榨服务器的资源。

## 书籍

1.[Linux开源网络全栈详解：从DPDK到OpenFlow])(http://product.china-pub.com/8061094)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/linux_opensource_network_stack.jpg)

该书可以作为全面了解开源软件网络的相关技术，涉及到Linux虚拟网络、DPDK、OpenStack、容器相关网络等知识。

2.[Kubernetes 网络权威指南：基础、原理与实践](http://product.china-pub.com/8064178)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/k8s-network.jpg)

该书可以作为全面了解k8s相关的容器网络的相关技术，如果对k8s周边的虚拟网络知识有所全面了解，该书籍还是比较适合的。

