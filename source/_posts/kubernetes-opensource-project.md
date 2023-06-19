title: Github Kubernetes组织下开源项目（持续更新）
tags: []
categories: []
date: 2022-02-09 20:12:00
author:
---
在k8s官方的Github kubernetes group下除了k8s的核心组件外，还有很多开源项目，本文用来简要分析这些开源项目的用途。
# [apimachinery](https://github.com/kubernetes/apimachinery)

关于k8s的scheme等的sdk项目，代码跟kubernetes项目的如下路径保持同步：
https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apimachinery

# [autoscaler](https://github.com/kubernetes/autoscaler)

跟k8s的弹性扩缩容相关的组件，有如下两个功能：

## vpa（pod垂直扩缩容）
k8s的kube-controller-manager默认支持了hpa功能，即水平扩缩容。同时k8s还提供了vpa功能，即垂直扩缩容，会根据pod历史的资源占用，修改pod的request值，并不会修改pod的limit值。之所以k8s没有默认提供vpa功能，原因是因为vpa实现要复杂很多，需要通过webhook的技术来在pod创建的时候修改pod的request值。vpa的功能通常用于大型单体应用。

autoscaler的功能之一即提供了vpa的实现。

## Cluster Autoscaler（node节点自动扩缩容）

该功能是为了重新利用k8s node的节点资源，在节点资源不足的时候可以动态创建资源，在节点资源空闲的时候可以自动回收资源。k8s node的创建和释放需要公有云平台的支持，该功能对接了多个公有云厂商的api。

# [cloud-provider](https://github.com/kubernetes/cloud-provider)

k8s的cloud provider功能用来跟云厂商对接以实现对k8s的node、LoadBalancer Service管理，比如LoadBalancer Service申请vip需要跟云厂商的LB API进行对接。

在k8s 1.6版本之前，cloud provider的代码完全是耦合在k8s的kube-apiserver、kubelet、kube-contorller-manager的代码中的，如果要支持更多的云厂商，只能修改k8s的代码，扩展性比较差。

从k8s 1.6版本之后，cloud provider作为单独的二进制组件来提供服务，并提供了接口定义。本项目为非二进制项目，仅包含k8s的cloud-provider机制的接口定义，为了让第三方的cloud-provider的实现项目引用接口定义方便，将kubernetes项目中的定义为单独的项目，且代码跟[kubernetes项目](https://github.com/kubernetes/kubernetes/tree/master/staging/src/k8s.io/cloud-provider)中的保持同步。

参考资料：
- [从 K8S 的 Cloud Provider 到 CCM 的演进之路](https://mp.weixin.qq.com/s/a_540yJ1EGVroJ9TpvYtPw)


# Descheduler
项目地址：https://github.com/kubernetes-sigs/descheduler

k8s的pod调度完全动态的，kube-scheduler组件在调度pod的时候会根据当时k8s集群的运行时环境来决定pod可以调度的节点，可以保证pod的调度在当时是最优的。但是随着的推移，比如环境中增加了新的node、创建了一批亲和节点的pod，都有可能会导致如果相同的pod再重新调度会到其他的节点上。但k8s的设计是，一旦pod调度完成后，就不会再重新调度。

Descheduler组件用来解决pod的重新调度问题，可以根据配置的一系列的规则来触发驱逐pod，让pod可以重新调度，从而使k8s集群内的pod尽可能达到最优状态，有点类似于计算机在运行了一段时间后的磁盘脆片整理功能。Descheduler组件可以以job、cronjob或者deployment的方式运行在k8s集群中。

# [kubernetes-template-project](https://github.com/kubernetes/kubernetes-template-project)

用来存放了k8s项目的标准模板，里面包含了一些必要的文件，新建的项目可以fork该项目。


# node-problem-detector(npd)
项目地址：https://github.com/kubernetes/node-problem-detector

k8s的管控组件对于iaas层的node运行状态是完全不感知的，比如节点出现了ntp服务挂掉、硬件告警（cpu、内存、磁盘故障）、内核死锁。node-problem-detector旨在将node节点的问题转换k8s node event，以DaemonSet的方式部署在所有的k8s节点上。

上报故障的方式支持如下两种方式：
- 对于永久性故障，通过修改node status中的condition上报给apiserver
- 对于临时性故障，通过Event的方式上报

node-problem-detector在将节点的故障信息上报给k8s后，通常会配合一些自愈系统搭配使用，比如Draino和Descheduler 。

# [sample-apiserver](https://github.com/kubernetes/sample-apiserver)

k8s提供了aggregated apiserver的方式来扩展api，该项目为一个简单的aaggregated apiserver的例子，可以直接编译出二进制文件。要想开发一个AA类型的服务，只需要frok一下该项目，并在项目的基础上进行修改即可完成。

相关资料：
- [Set up an Extension API Server](Set up an Extension API Server)