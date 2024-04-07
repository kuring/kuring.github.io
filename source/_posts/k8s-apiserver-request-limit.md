title: k8s apiserver 的限流
date: 2024-04-07 15:34:37
tags:
author:
---
# client 端限流
在 client-go 中会默认对客户端进行限流，并发度为 5。可以通过修改 rest.Conifg 来修改并发度。
# MaxInFlightLimit 限流
通过如下参数来控制：

- --max-requests-inflight：代表只读请求的最大并发量
- --max-mutating-requests-inflight：代表写请求的最大并发量

该实现为单个 kube-apiserver 层面的，可以针对所有的请求。
# EventRateLimit 
用来对 Event 类型的对象进行限制，可以通过 kube-apiserver 的参数 --admission-control-config-file 来指定配置文件，文件格式如下：
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
  - name: EventRateLimit
    path: eventconfig.yaml
...
```
其中 EventRateLimit 为对 Event 的限制，eventconfig.yaml 文件为详细的对 Event 的限流策略，可以精确到 Namespace 和 User 信息。
```yaml
apiVersion: eventratelimit.admission.k8s.io/v1alpha1
kind: Configuration
limits:
  - type: Namespace
    qps: 50
    burst: 100
    cacheSize: 2000
  - type: User
    qps: 10
    burst: 50
```
# API 优先级和公平性
版本状态：

1. alpha：1.18

通过 kube-apiserver 的参数 `--enable-priority-fairness` 来控制是否开启 APF 特性。

# 资料

- [kubernetes apiserver限流方案](https://qingwave.github.io/k8s-rate-limit/)
- [API 优先级和公平性](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/flow-control/)



