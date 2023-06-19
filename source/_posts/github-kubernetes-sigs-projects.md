title: Github Kubernetes SIGs组织下的项目（持续更新）
date: 2022-02-14 23:21:37
tags:
author:
---
# [apiserver-builder-alpha](https://github.com/kubernetes-sigs/apiserver-builder-alpha)

k8s提供了aggregated apiserver的方式来扩容api，该项目提供了代码生成器、基础library来供开发AA使用。

该项目的定位跟kubebuilder比较类似，kubebuilder用来生成CRD的框架，该项目用来生成AA的框架。

相关资料：
- [Set up an Extension API Server](Set up an Extension API Server)

# [cluster-api-provider-nested](https://github.com/kubernetes-sigs/cluster-api-provider-nested/tree/main/virtualcluster?spm=a2c4g.11186623.0.0.23054a15YVTwkg)

在同一个k8s集群内提供多租户的特性，每个租户具有独立的api-server、controller-manager和scheduler。

# [cluster-proportional-autoscaler](https://github.com/kubernetes-sigs/cluster-proportional-autoscaler)


k8s默认提供了hpa机制，可以根据pod的负载情况来对workload进行自动的扩缩容。同时以单独的autoscaler项目提供了vpa功能的支持。

该项目提供提供了类似pod水平扩容的机制，跟hpa不同的是，pod的数量由集群中的节点规模来自动扩缩容pod。特别适合负载跟集群规模的变化成正比的服务，比如coredns、nginx ingress等服务。

hpa功能k8s提供了CRD来作为hpa的配置，本项目没有单独的CRD来定义配置，而是通过在启动的时候指定参数，或者配置放到ConfigMap的方式。而且一个cluster-proportional-autoscaler实例仅能针对一个workload。

# [kube-batch](https://github.com/kubernetes-sigs/kube-batch)

k8s的调度器扩展，实现了Gang Scheduling特性（一组pod必须同时被调度成功，或者处于pending状态），适用于批处理系统。

由于该组件是以单独调度器的形式存在，会跟k8s默认的kube-scheduler并存，因为两个调度器之间并不能相互感知，在两个调度器并存的情况下会存在一定的冲突。


# [prometheus-adapter](https://github.com/kubernetes-sigs/prometheus-adapter)

k8s要实现hpa（水平自动扩容）的功能，需要监控指标。k8s的监控指标分为核心指标和自定义指标两大类。其中核心指标由metrics-server组件从kubelet、cadvisor等组件来获取，并通过aggregated apiserver的形式暴露给k8s，aggregated api的group信息为metrics.k8s.io。

自定义监控指标则由group custom.metrics.k8s.io对通过aggregated api暴露。本项目即为自定义监控指标的aggregated apiserver的实现。

相关参考：[Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-custom-metrics)

# [scheduler-plugins](https://github.com/kubernetes-sigs/scheduler-plugins)

k8s从1.16版本开始提供了新的调度框架Kubernetes Schduling Framework机制，用户可以基于此项目来开发自己的插件。该项目可以直接构建出kube-scheduler的新镜像。

相关参考：[进击的Kubernetes调度系统（一）：Scheduling Framework](https://developer.aliyun.com/article/766273)

# [sig-storage-local-static-provisioner](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner)

k8s提供了local pv功能可以用来给pod挂载本地的数据盘，具体的local pv的定义如下所示，pv中包含了要亲和的节点信息：

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```

但要使用local pv功能，必须要事先创建出pv才可以，k8s本身并没有提供动态创建pv的功能。

该工具可以根据配置的规则，自动将机器上符合条件的磁盘创建出local pv以供后续创建出的pod使用。

另外，Rancher提供了一个类似的项目，[local-path-provisioner](https://github.com/rancher/local-path-provisioner)，该项目已经被拉起k8s开发环境的开源项目kind使用。

相关参考：[LocalVolume数据卷](https://help.aliyun.com/document_detail/178475.html)