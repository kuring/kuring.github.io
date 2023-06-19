---
title: 本地连接远程的内网k8s集群
date: 2020-05-11 19:41:53
tags:
---

在日常开发的过程中，经常会需要在本地开发的程序需要在k8s中调试的场景，比如，写了一个operator。如果此时，本地又没有可以直接可达的k8s集群，比如k8s是在公有云的vpc环境内，外面无法直接访问。此时，ssh又是可以直接通过公网vip访问的vpc的网络内的。为了满足此类需要，可以采用ssh tunnel的方式来打通本地跟远程的k8s集群。

## 1. 本地建立ssh tunnel到远程集群网络

mac用户可以通过SSH TUNNEL这个软件来在界面上自动化配置，具体的配置方式如下：

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/local-connect-remote-k8s-1.jpg)

需要增加一个ssh的动态代理，监听本地的9909端口号。

## 2. 将远程集群的kubeconfig文件复制到本地

将k8s集群中`~/.kube/config`文件复制到本地的`~/.kube/config`目录下

## 3. 在命令行中执行kubectl命令

在命令行中配置http和https代理

```
export http_proxy=127.0.0.1:9909
export https_proxy=127.0.0.1:9909
```

然后至此就可以通过kubectl命令来访问远程的k8s集群了。
