---
title: 知识分享第3期
date: 2018-09-26 00:01:27
tags:
---

![玉渡山](https://kuring.oss-cn-beijing.aliyuncs.com/images/yudushan.jpeg)

题图为北京玉渡山风景区中的盘山公路，旁边有个观景台，在观景台上可以鸟瞰官厅水库。

## 资源

1.[Intel RDT](https://www.intel.cn/content/www/cn/zh/architecture-and-technology/resource-director-technology.html)

Intel RDT(Resource Director Technology)资源调配技术框架，包括高速缓存监控技术（CMT）、高速缓存分配技术（CAT）、内存带宽监控（MBM）和代码和数据优先级（CDP），容器技术的runc项目中使用到了CAT技术来解决cgroup下的CPU的三级缓存隔离性问题。

在linux 4.10以上内核中通过资源控制文件系统的方式来提供给用户接口，类似cgroup的管理方式。

感兴趣的可以了解下[runc项目源码](https://github.com/opencontainers/runc/blob/master/libcontainer/intelrdt/intelrdt.go)。

2.[Thanos](https://github.com/improbable-eng/thanos)

![Thanos](https://kuring.oss-cn-beijing.aliyuncs.com/images/Thanos-logo_fullmedium.png)

Prometheus作为Google内部监控系统Borgmon的开源实现版本，存在高可用和历史数据存储两个致命的缺点，Thanos利用Sidecar等技术来解决Prometheus的缺点。

3.[netshoot](https://github.com/nicolaka/netshoot)

```
                    dP            dP                           dP
                    88            88                           88
88d888b. .d8888b. d8888P .d8888b. 88d888b. .d8888b. .d8888b. d8888P
88'  `88 88ooood8   88   Y8ooooo. 88'  `88 88'  `88 88'  `88   88
88    88 88.  ...   88         88 88    88 88.  .88 88.  .88   88
dP    dP `88888P'   dP   `88888P' dP    dP `88888P' `88888P'   dP
```

用于排查docker网络问题的工具，以容器的方式运行在跟要排查问题的容器同一个网络命名空间中，该容器中已经具备了较为丰富的网络命令行工具，用于排查容器中的网络问题。

4.[bat](https://github.com/sharkdp/bat)

用来替代cat的命令行工具，支持语法高亮、自动分页。mac下可直接使用`brew install bat`来安装。

![image](https://camo.githubusercontent.com/9d3d89364f2cc83ace8f29646a6236bc15ea1da0/68747470733a2f2f696d6775722e636f6d2f724773646e44652e706e67)

5.[asciiflow](http://asciiflow.com/)

![asciiflow](https://kuring.oss-cn-beijing.aliyuncs.com/images/asciiflow.png)

写博客的往往都比较痛恨图片的存储问题，尤其是使用markdown语法写作的，图片往往需要图床来存储，常常跟文章不在一起存储。asciiflow是较为小众的一款ascii图形工具，可以应付较为简单的图形绘制，直接以文字的形式呈现简单图形，省去了存储图片的繁琐。

6.[processon](https://www.processon.com)

![processon](https://kuring.oss-cn-beijing.aliyuncs.com/images/processon.gif)

免费的在线图行绘制协作工具，支持流程图、思维导图等多种图形，有类似visio的使用体验，同时是web版的，支持多人协作。我目前在使用，不过免费版有使用限制。

## 精彩文章

1.[手把手教你打造高效的 Kubernetes 命令行终端
](https://mp.weixin.qq.com/s?__biz=MzI3MTI2NzkxMA==&mid=2247486254&idx=1&sn=c78b509e84a64cb921280a5e1e111bb7&chksm=eac52a07ddb2a311c2a21a3decf26c8ab5d9b0d6c9ff8701a8db3e369d76fe9c6045e34808f1&mpshare=1&scene=1&srcid=0915GAFGR2dEjnoRI1ngfo8f%23rd)

文中汇总了各种可以取代kubernetes的命令行kubectl的工具，以便提供更方便的操作，比如更完善的自动补全。

2.[Understand Container - Index Page](http://pierrchen.blogspot.com/2018/08/understand-container-index.html)

学习容器的cgroup和namespace的系列文章。

3.[gVisor是什么？可以解决什么问题？](https://mp.weixin.qq.com/s?__biz=MzA5OTAyNzQ2OA==&mid=2649698847&idx=1&sn=b1ffce56c1397b3f60f8c776aff0ae85&chksm=88930d7cbfe4846a14fe50e9824e8a81abce19097717b1fbd3834101b21872f5ccdb6aab9366&mpshare=1&scene=1&srcid=0919nVdgzYLK9dtdRUsf2ifa%23rd)

docker容器技术基于cgroup和namespace来实现，但系统调用仍是调用宿主机的系统调用，比如在其中一个容器中通过系统调用修改了当前系统时间，在其他容器中看到的时间也已经修改过了，这显然不是符合期望的，通常可以通过Seccomp来限制容器中的系统调用。

gVisor为Google开源的容器Runtime，通过pstrace技术来截获系统调用，从而保证系统的安全。目前还不成熟，单就凭Google的开源项目，该项目还是非常值得关注的。

4.[Use multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)

Dockerfile的多阶段构建技术，对于解决编译型语言的发布非常有帮助，可以在其中一个image中编译源码，另外一个image用于将编译完成后的二进制文件复制过来后打包成单独的线上运行镜像。而这两部操作可以合并到一个Dockerfile中来完成。

5.[唯品会Noah云平台实现内幕披露](https://mp.weixin.qq.com/s?__biz=MzA5OTAyNzQ2OA==&mid=2649698903&idx=1&sn=6392175b0cf62825e4981b08acc85fda&chksm=88930d34bfe48422ee85d50037489868e2432b6aa4c7bef6dc46b8eaa3852f91bd5da14c5da1&mpshare=1&scene=1&srcid=09255FcTm8fVdN6r5OqWkTvK%23rd)

唯品会内部云平台的实践，涉及到大量的干货，值的花时间一读。

## App推荐

1.Nike Training

健身类app我用过keep、火辣健身、FitTime（以收费课程居多），偶然间在AppStore上看到了Nike Training，如果厌倦了国内的健身类app，不防尝试一下。

## 新奇

1.手机QQ扫一扫

用手机QQ扫一扫100元人民币正面，可以出现浮动的凤凰图案，并会跳转到人民币鉴别真伪的视频页面，视频效果确实不错，忍不住会多扫描几遍。

2.kubeadm

kubernetes的组件非常多，部署起来非常复杂，因此社区就推出了kubeadm工具来简化集群的部署，将除了kubelet外的其他组件都部署在容器中。令人惊奇的是，kubeadm几乎完全是一个芬兰高中生Lucas KaIdstrom的作品，是他在17岁时利用业余时间完成的一个社区项目。
