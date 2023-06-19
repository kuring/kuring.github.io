title: 用来排查k8s问题的常用grafana dashboard
date: 2022-03-14 23:09:31
tags:
author:
---
排查k8s上问题通常需要监控的配合，而k8s上的监控标准为prometheus，prometheus的dashboard最通用的为grafana。本文用来记录排查k8s问题时经常遇到的dashboard，dashboard监控的数据来源包括node-exporter、metrics-server、kube-state-metrics等最场景。

## node top监控

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/grafana-dashboard1.jpg)

[下载链接](https://kuring.oss-cn-beijing.aliyuncs.com/files/node%20top%E7%9B%91%E6%8E%A7-1647271172485.json)

## node上的pod监控

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/grafana-dashboard2.jpg)

[下载链接](https://kuring.oss-cn-beijing.aliyuncs.com/files/node%E4%B8%8A%E7%9A%84pod-1647271326652.json)