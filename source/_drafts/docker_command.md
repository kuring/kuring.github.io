title: docker client常用操作
date: 2022-05-31 14:28:37
tags:
author:
---
- 打包操作：`docker save k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.3  | gzip > tmp.gzip`
- 镜像导入操作：`gunzip -c mycontainer.tgz | docker load`: 导入打好的包



## docker ps 

`--size`： 用来查看磁盘空间。格式为`237kB (virtual 11GB)`。其中 237kB表示可写层的总大小，即容器大小。11GB表示可写层 + 镜像大小。其中容器大小并不包含：

1. 标准输出和标准错误的大小
2. 容器使用的卷大小
3. 容器配置文件大小，比如 /etc/resolv.conf 
4. 其他使用磁盘空间的大小
