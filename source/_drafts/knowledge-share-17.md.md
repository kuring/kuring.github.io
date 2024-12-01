title: 技术分享第 17 期
date: 2023-01-09 19:03:00
tags:
author:
---
# 工具

## [jsoncrack](https://jsoncrack.com/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/jsoncrack.webp)

json 的可视化工具，支持 web 方式查看和集成到 VSCode 中。该工具的一大特色为支持以思维导图的树状结构来展示 json。

## [nerdctl](https://github.com/containerd/nerdctl)

containerd 提供的 docker 风格的命令行工具。随着 k8s 1.24 不再支持 docker，很多的容器运行时会从 docker 切换为 containerd，containerd 的命令行工具为 crictl，该命令的输出格式跟 docker 并不完全一致。因此 containerd 提供了 nerdctl 工具，可以提供 docker 风格的输出结果，操作 containerd 容器就像操作 docker 容器一样。

## [sshfs](https://github.com/deadbeefsociety/sshfs)

sshfs 允许用户将远程主机的目录挂载到本地，只需要可以本地通过 ssh 协议连接到远程主机即可。底层使用 fuse 文件系统实现，基本用法：`sshfs [user@]hostname:[directory] mountpoint`。

## [lazyvim](https://www.lazyvim.org/)

一直在使用 [SpaceVim](https://spacevim.org/)，但是使用起来还是有些繁琐，最近源码的托管从 Github 迁移到了 Gitlab。lazyvim 基于 neovim，无论从文档
还是从用户体验上比 SpaceVim 更好一些。

## [Arc](https://arc.net/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/arc.jpg)

颜值非常高的一款 mac 下的浏览器软件，跟 mac 的适配度非常高。基于 chrome 开发，兼容 chrome 的插件。支持 workspace，可以在不同的 workspace 下切换。除了浏览器的功能外，还不务正业的支持了笔记本和画图的功能。如果你是 mac 用户，不防尝试下该浏览器，或许会让你抛弃 chrome 浏览器。

## [CDK](https://github.com/cdk-team/CDK/wiki/CDK-Home-CN)

容器环境下的渗透测试工具，可以用来测试容器的进程逃逸等安全漏洞。
## [SwitchHosts](https://github.com/oldj/SwitchHosts)
mac 上的免费的用来管理本地的  /etc/hosts 文件的工具。在 mac 上可以直接使用 `brew install switchhosts`命令进行安装。

## [Docker Images Pusher](https://github.com/tech-shrimp/docker_image_pusher)
将 docker 镜像推送到阿里云 ACR 的工具。

# 资源

## [Liber3](https://liber3.eth.limo/)
基于区块链技术建立的去中心化的电子书搜索引擎，想查找电子书的不防试一下该网站。

## 文章

1. [Replication](https://mp.weixin.qq.com/s/LB5SR4ypQwDxzueI1ai2Kg)

美团出品的对于下文书籍《数据密集型应用系统设计》的解读，分为了[《Replication（上）：常见的复制模型&分布式系统的挑战》](https://mp.weixin.qq.com/s/LB5SR4ypQwDxzueI1ai2Kg) 和 [Replication（下）：事务，一致性与共识](https://mp.weixin.qq.com/s/O9Z5e_BzdxKcULHigYMkRg)两部分。
2. [科学上网](https://github.com/haoel/haoel.github.io#-%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91-)
已故技术大拿左耳朵耗子的科学上网指南。

## 书籍

1. 《小米创业思考》

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/xiaomi.jpeg)

2.《数据密集型应用系统设计》

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/ddia.jpeg)

初看书名觉得此书不咋地，但实际上是大名鼎鼎的 DDIA，豆瓣评分高达9.7，对于全面深入理解分布式系统应用有非常大的帮助。

中文翻译链接：https://vonng.gitbooks.io/ddia-cn/content/，但没有纸质版书籍的翻译质量高，推荐阅读纸质版。

# 引用
- [去中心化电子书搜索引擎，打开无限阅读的大门-Liber3](https://mp.weixin.qq.com/s/UiXfQaRFXnkjem_GUNG5Vg)