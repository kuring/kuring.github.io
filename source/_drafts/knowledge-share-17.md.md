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