title: k8s在region和zone方面的支持情况
tags: []
categories: []
date: 2022-01-25 12:33:00
---

k8s虽然已经发展了多个版本，但在多region和多zone的场景下支持还是相对比较弱的，且很多的特性在alpha版本就已经废弃，说明k8s官方对于region和zone方面的支持情况有很大的不确定性。业界支持多reigon和容灾的特性更多是从上层的应用层来解决。本文主要是介绍k8s自身以及社区在region和zone方面的支持情况。

# k8s标签
[https://kubernetes.io/docs/reference/labels-annotations-taints/#topologykubernetesiozone](https://kubernetes.io/docs/reference/labels-annotations-taints/#topologykubernetesiozone)
官方推荐的跟region和zone有关的标签：

- failure-domain.beta.kubernetes.io/region：1.17版本已经废弃，被topology.kubernetes.io/region取代
- failure-domain.beta.kubernetes.io/zone：1.17版本已经废弃，被topology.kubernetes.io/zone取代
- topology.kubernetes.io/region：1.17版本开始支持
- topology.kubernetes.io/zone：1.17版本开始支持

# 服务拓扑ServiceTopology
支持版本：在1.17版本引入，在1.21版本废弃

该特性在Service对象上增加了spec.topologyKeys字段，表示访问该Service的流量优先选用的拓扑域列表。访问Service流量的具体过程如下：

1. 当访问该Service时，一定是从某个k8s的node节点上发起，会查看当前node的label topology.kubernetes.io/zone对应的value，如果发现有endpoint所在的node的lable topology.kubernetes.io/zone对应的value相同，那么会将流量转发到该拓扑域的endpoint上。
1. 如果没有找到topology.kubernetes.io/zone相同拓扑域的endpoint，则尝试找topology.kubernetes.io/region相同拓扑域的endpoint。
1. 如果没有找到任何拓扑域的endpoint，那么该service会转发失败。
```yaml
apiVersion: v1
kind: Service
metadata:
  Name: my-app-web
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  topologyKeys:
    - "topology.kubernetes.io/zone"
    - "topology.kubernetes.io/region"
```
在底层实现上，使用了1.16版本新引入的alpha版本特性 EndpointSlice，该特性主要是用来解决Endpoints对象过多时带来的性能问题，从而将Endpoints拆分为多个组，Service Topology特性恰好可以借助该特性来实现，本质上也是为了将Endpoints进行拆分。k8s会根据Service的topologyKeys来将Service拆分为多个EndpointSlice对象，kube-proxy根据EndpointSlice会将Service流量进行拆分。

# 拓扑感知提示Topology Aware Hints
该特性在1.21版本引入，在1.23版本变为beta版本，用来取代Topology Key功能。
该特性会启用两个组件：EndpointSlice控制器和kube-proxy。

在Service Topology功能中，需要给Service来指定topologyKeys字段。该特性会更自动化一些，只需要在Service上增加annotation service.kubernetes.io/topology-aware-hints:auto，EndpointSlice控制器会watch到Service，发现开启了拓扑感知功能，会自动向endpoints的hints中增加forZones字段，该字段的value会根据endpoint所在node的topology.kubernetes.io/zone来决定。

值得一提的是，当前的hints中并没有包含forRegions的字段。
```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: example-hints
  labels:
    kubernetes.io/service-name: example-svc
addressType: IPv4
ports:
  - name: http
    protocol: TCP
    port: 80
endpoints:
  - addresses:
      - "10.1.2.3"
    conditions:
      ready: true
    hostname: pod-1
    zone: zone-a
    hints:
      forZones:
        - name: "zone-a"
```

# 参考

- [Well-Known Labels, Annotations and Taints]()
- Google Ingress for Anthos：[https://cloud.google.com/kubernetes-engine/docs/concepts/ingress-for-anthos#architecture](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress-for-anthos#architecture)
- 阿里云ccm支持多个k8s集群的场景（手工指定lb id和server group方案）：[https://help.aliyun.com/document_detail/335878.html#title-4wt-hc5-p2p](https://help.aliyun.com/document_detail/335878.html#title-4wt-hc5-p2p)
- [https://tencentcloudcontainerteam.github.io/2019/11/26/service-topology/](https://tencentcloudcontainerteam.github.io/2019/11/26/service-topology/)
- [https://kubernetes.io/zh/docs/concepts/services-networking/endpoint-slices/](https://kubernetes.io/zh/docs/concepts/services-networking/endpoint-slices/)
- [使用 EndpointSlice 扩展 Kubernetes 网络](https://zhuanlan.zhihu.com/p/245165617)
- [Running in multiple zones](https://kubernetes.io/docs/setup/best-practices/multiple-zones/)
