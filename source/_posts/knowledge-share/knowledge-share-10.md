---
title: 知识分享第10期
date: 2019-03-01 19:41:57
tags:
---

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/beixiaohe.jpeg)

春回大地，题图为即将融化的河面以及还在冰面上行走的路人。

## 资源

1.[fastThread](https://fastthread.io/)

在线的JVM线程栈分析工具，通过上传JVM Dump文件，在线查看线程分析结果。

2.[全球空气质量地图](https://www.purpleair.com/map)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/purpleair.png)

可以在线查看全球的PM2.5情况，很多国家的PM2.5都超过了200，但并不包含中国的数据，不知道是不是怕数据把其他国家吓死的缘故。

3.[Walle](https://github.com/meolu/walle-web)

![http://www.walle-web.io/docs/2/zh-cn/static/deploy-console.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/walle-deploy-console.png)

使用Python3开发的CI/CD平台，有相对友好的界面，目前Github Star数在6000+。

4.[Træfɪk](https://traefik.io/)

为微服务而生的HTTP协议反向代理，自带API接口、dashboard，并支持Kubernetes Ingress、Mesos等，可动态加载配置文件等诸多nginx不具备的特性。

5.[kcptun](https://github.com/xtaci/kcptun)

![https://github.com/xtaci/kcptun/blob/master/kcptun.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/kcptun.png)

基于KCP协议的UDP隧道，[KCP](https://github.com/skywind3000/kcp)协议能以比 TCP浪费10%-20%的带宽的代价，换取平均延迟降低 30%-40%。

6.[k3s - 5 less than k8s](https://github.com/ibuildthecloud/k3s)

有人搞了个k3s项目，作为轻量级的k8s，整个二进制包只有40M大小。项目定位为边缘计算、IoT、CI等，支持多种硬件平台，裁剪了k8s的很多功能，比如云依赖，存储插件等，甚至连k8s依赖的etcd存储都默认更换为了sqlite3。

7.[Drone](https://drone.io)

基于Golang的Container Native的CD平台，Github上star 17000+，插件的安装也是基于容器的。

8.[kaniko](https://github.com/GoogleContainerTools/kaniko#how-does-kaniko-work)

通过Dockerfile来构建docker镜像，需要dockerd进程的支持，这在物理机上操作没有任何问题。而如果要想在容器中通过Dockerfile构建docker镜像却变得困难起来，因为dockerd的运行需要root权限，而容器为了安全是不建议开启root权限的。

该工具可以在容器中不运行dockerd的情况下通过Dockerfile构建出docker镜像。

9.[Nacos](https://nacos.io/en-us/)

阿里巴巴开源的微服务框架，支持配置中心、动态服务发现、动态DNS。

10.[PowerDNS](https://www.powerdns.com/)

![https://user-images.githubusercontent.com/6447444/44068603-0d2d81f6-9fa5-11e8-83af-14e2ad79e370.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/powerdns-admin.png)

Linux下除了bind外的另一个可选择的DNS服务器，数据存储在mysql中，还有一个可选择的漂亮UI。

## 精彩书籍

- [《激荡十年，水大鱼大》](http://product.dangdang.com/25180345.html#ddclick_reco_reco_relate)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/jidang10.jpeg)

要想回顾一下过去的十年中都发生了哪些大事，中国发生了哪些变化，经济领域里有哪些大起大落，本书可以拿来一读。
