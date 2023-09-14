title: k8s service
date: 2023-09-12 12:05:42
tags:
author:
---
k8s Service 是 k8s 的核心对象，本文主要是针对 k8s Service 的一些细节进行说明。

# ExternalName Service

service 的 `spec.type` 为 `ExternalName`，该种类型的服务会向 dns 中注册 CNAME 记录，CNAME 记录指向 externalName 字段。例子如下：

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

当访问 `my-service.prod.svc.cluster.local` 时，DNS 服务会返回 CNAME 记录，指向地址为 `my.database.example.com`。


# externalIPs 字段

可以针对所有类型的 Service 生效，用来配置多个外部的 ip 地址（该 ip 地址不是 k8s 分配），kube-proxy 会设置该 ip 地址的规则，确保在 k8s 集群内部访问该 ip 地址时，可以路由到后端的 pod。效果就跟访问普通的 ClusterIP 类型 Service 没有区别。

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 49152
  externalIPs:
    - 198.51.100.32
```

当在集群内部访问 `198.51.100.32:80` 时，流量会被 kube-proxy 路由到当前 Service 的 Endpoint。

该字段存在中间人攻击的风险，不推荐使用。[Detect CVE-2020-8554 – Unpatched Man-In-The-Middle (MITM) Attack in Kubernetes](https://sysdig.com/blog/detect-cve-2020-8554-using-falco/)


# Headless Service

翻译成中文又叫无头 Service，显式的将 Service `spec.clusterIP` 设置为 `"None"`，表示该 Service 为 Headless Service。此时，该 Service 不会分配 clusterIP。因为没有 clusterIP，因此 kube-proxy 并不会处理该 service。

Headless Service 按照是否配置了 `spec.selector` 在实现上又有不同的区分。

未配置 `spec.selector` 的 Service，不会创建 EndpointSlice 对象，但是会注册如下的记录：
- 对于 ExternalName Service，配置 CNAME 记录。
- 对于非 ExternalName Service，配置 A/AAAA 记录，指向 EndPoint 的所有 ip 地址。如果未配置 Endpoint，但配置了 externalIPs 字段，则指向 externalIPs。

配置 `spec.selector` 的 Service，会创建 EndpointSlice 对象，并修改 DNS 配置返回 A 或者 AAAA 记录，指向 pod 的集合。


# DNS

除了 headless service 之外的其他 service 会在 DNS 中生成 `my-svc.my-namespace.svc.cluster-domain.example` 的 A 或者 AAAA 记录，A 记录指向 ClusterIP。

headless service 会在 DNS 中生成 `my-svc.my-namespace.svc.cluster-domain.example` 的 A 或者 AAAA 记录，但指向的为 pod ip 地址集合。


k8s 在 pod 的 /etc/resolv.conf 配置如下：

```
nameserver 10.32.0.10
search <namespace>.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

对于跟 pod 同一个 namespace 下的 service，要访问可以直接使用 service 名字接口。跟 pod 不在同一个 namespace 下的 service，访问 service 必须为 `service name.service namespace`。



# 资料

- [k8s 官方 Service 文档](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)
- [k8s 官方 Service 与 Pod 的 DNS](https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/)