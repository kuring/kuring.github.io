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

# 资源

## [Liber3](https://liber3.eth.limo/)
基于区块链技术建立的去中心化的电子书搜索引擎，想查找电子书的不防试一下该网站。

# 引用
- [去中心化电子书搜索引擎，打开无限阅读的大门-Liber3](https://mp.weixin.qq.com/s/UiXfQaRFXnkjem_GUNG5Vg)