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

一直在使用 [SpaceVim](https://spacevim.org/)，但是使用起来还是有些繁琐，最近源码的托管从 Github 迁移到了 Gitlab。lazyvim 基于 neovim，无论从文档还是从用户体验上比 SpaceVim 更好一些。