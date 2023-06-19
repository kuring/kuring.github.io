---
title: yum源搭建
date: 2017-12-07 12:10:19
tags:
---

某些情况下需要搭建自己的yum源，比如维持特定的软件包版本等，只需要从网上下载合适的rpm包，即可构建yum源。

# repodata数据

创建/data/yum.repo目录用来存放rpm包。

可以使用yumdownloader命令来下载rpm包到本地，并且不安装。这里以安装mesos为例，在/data/yum.repo目录下执行`yumdownloader mesos`即可下载mesos的rpm包到本地。

安装createrepo：`yum install createrepo`，用来根据rpm包产生对应的包信息。

每加入一个rpm包需要更新下repo的信息，执行`createrepo --update /data/yum.repo`。会自动产生repodata目录。

# 搭建web服务

需要对外提供web服务，通常会使用nginx或者apache来对外提供服务，这里使用python SimpleHTTPServer来对外提供服务，执行`cd /data/yum.repo && python -m SimpleHTTPServer 1080`。

# 客户端的repo文件设置

安装yum优先级插件，用来设置yum源的优先级: `yum install -y yum-plugin-priorities`

每个需要使用该yum源的客户端需要在/etc/yum.repo.d/目录下增加devops.repo文件。

```
[devops]
name=dev-ops
baseurl=http://10.103.17.184:1080/
enabled=1
gpgcheck=0
priority=1
```
