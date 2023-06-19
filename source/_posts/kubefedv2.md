---
title: k8s多集群管理方案 - KubeFed V2
date: 2022-01-09 11:33:15
tags: 多集群
---

# 背景

## Federation V1
Federation v1 在 K8s v1.3 左右就已经着手设计（Design Proposal），并在后面几个版本中发布了相关的组件与命令行工具（kubefed），用于帮助使用者快速建立联邦集群，并在 v1.6 时，进入了 Beta 阶段；但 Federation v1 在进入 Beta 后，就没有更进一步的发展，由于灵活性和 API 成熟度的问题，在 K8s v1.11 左右正式被弃用。

## Federation V2架构
v2 版本利用 CRD 实现了整体功能，通过定义多种自定义资源（CR），从而省掉了 v1 中的 API Server；v2 版本由两个组件构成：

- admission-webhook 提供了准入控制
- controller-manager 处理自定义资源以及协调不同集群间的状态

![image](https://kuring.oss-cn-beijing.aliyuncs.com/common/kubefedv2-1.png)
![image](https://kuring.oss-cn-beijing.aliyuncs.com/common/kubefedv2-2.png)

代码行数在2万行左右。

## 术语

- Host Cluster：主集群/父集群
- Member Cluster：子集群/成员集群

# CRD扩展
## KubeFedCluster
子集群抽象，通过kubefedctl join命令创建。后续controller会使用该信息来访问子k8s集群。
```powershell
kind: KubeFedCluster
metadata:
  name: kind-cluster1
  namespace: kube-federation-system
spec:
  apiEndpoint: https://172.21.115.165:6443
  caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1USXlOREUyTlRNME9Wb1hEVE14TVRJeU1qRTJOVE0wT1Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTmxjCnF3UHd5cHNFc2M0aXB3QnFuRndDN044RXo2Slhqajh3ZzRybnlXTDNSdlZ0cmVua1Nha1VyYlRjZWVIck9lQTUKWGlNUVo2T1FBY25tUGU0Q2NWSkFoL2ZLQzBkeU9uL0ZZeXgyQXppRjBCK1ZNaUFhK2dvME1VMmhMZ1R5eVFGdQpDbWFmUGtsNmJxZUFJNCtCajZJUWRqY3dVMHBjY3lrNGhSTUxnQmhnTUh4NWkzVkpQckQ2Y284dHcwVnllWncyCkdDUlh2ZzlSd0QweUs5eitOVS9LVS83QjBiMTBvekpNRlVJMktPZmI4N1RkQ0h2NmlBdlVRYVdKc1BqQ0M3MzQKcnBma1ZGZXB2S2liKy9lUVBFaDE4VE5qaitveEVGMnl0Vmo2ZWVKeFc3VVZrbit0T3BxdGpEeUtKVDllNnlwdAp6U1VDTnRZWTAzckdhNTlLczBVQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZOTnRQU3hsMlMxUldOb1BCeXNIcHFoRXJONlVNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFDME0vaktCL2dqcWRSNlRLNXJQVktSSFU2cy8xTTh2eTY2WFBiVEx4M0srazdicUFWdAoxVUFzcUZDZmNrVk91WHY3eFQ0ZHIzVzdtMHE1bDYzRGg3ZDJRZDNiQ00zY2FuclpNd01OM0lSMlpmYzR0VlBGCnRTMFFONElTa0hsYnBudXQxb0F3cy9CaXNwaXNRQ0VtbHF3Zy9xbmdPMStlWWRoWm5vRW40SEFDamF4Slc5MS8KNXlOR1pKRXdia2dYQTVCbSs3VEZRL2FiYnp5a1JvOWtTMnl5c29pMnVzcUg0ZnlVS0ZWK2RETnp3Ujh0ck16cgpjWkRBNHpaVFB1WGlYRkVsWlNRa2NJSGIyV0VsYmZNRGpKNjlyVG5wakJCOWNPQ25jaHVmK0xiOXQwN1lJQ01wCmNlK0prNVp2RElRUFlKK3hTeGdRaVJxMTFraWlKcFE4Wm1FWgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  proxyURL: ""
  secretRef:
    name: kind-cluster1-dbxns
status:
  conditions:
  - lastProbeTime: "2021-12-24T17:00:47Z"
    lastTransitionTime: "2021-12-24T17:00:07Z"
    message: /healthz responded with ok
    reason: ClusterReady
    status: "True"
    type: Ready
```
## Fedrated<Kind>
发布应用到子集群时手工创建。每一个要被联邦管理的资源都会对应一个Fedrated<Kind>类型的资源，比如ConfigMap对应的是FederatedConfigMap。FederatedConfigMap包含了三部分的信息：

- template：联邦的资源详细spec信息，需要特别注意的是并没有包含metadata部分的信息。
- placement：template中的spec信息要部署的k8s子集群信息
- overrides：允许对部分k8s的部分资源进行自定义修改
```powershell
apiVersion: types.kubefed.io/v1beta1
kind: FederatedConfigMap
metadata:
  name: test-configmap
  namespace: test-namespace
spec:
  template:
    data:
      A: ala ma kota
  placement:
    clusters:
    - name: cluster2
    - name: cluster1
  overrides:
  - clusterName: cluster2
    clusterOverrides:
    - path: /data
      value:
        foo: bar
```
## FederatedTypeConfig
通过kubefedctl enable <target API type>创建。定义了哪些k8s api资源需要联邦，下面的例子描述了k8s ConfigMap要被联邦资源FederatedConfigMap所管理。
```powershell
apiVersion: core.kubefed.io/v1beta1
kind: FederatedTypeConfig
metadata:
  annotations:
    meta.helm.sh/release-name: kubefed
    meta.helm.sh/release-namespace: kube-federation-system
  finalizers:
  - core.kubefed.io/federated-type-config
  labels:
    app.kubernetes.io/managed-by: Helm
  name: configmaps
  namespace: kube-federation-system
spec:
  federatedType:
    group: types.kubefed.io
    kind: FederatedConfigMap
    pluralName: federatedconfigmaps
    scope: Namespaced
    version: v1beta1
  propagation: Enabled # 是否启用该联邦对象
  targetType:
    kind: ConfigMap
    pluralName: configmaps
    scope: Namespaced
    version: v1
status:
  observedGeneration: 1
  propagationController: Running
  statusController: NotRunning
```
## ReplicaSchedulingPreference
用来管理相同名字的FederatedDeployment或FederatedReplicaset资源，保证所有子集群的副本数为spec.totalReplicas。
```powershell
apiVersion: scheduling.kubefed.io/v1alpha1
kind: ReplicaSchedulingPreference
metadata:
  name: test-deployment
  namespace: test-ns
spec:
  targetKind: FederatedDeployment
  totalReplicas: 9
  clusters:
    A:
      minReplicas: 4
      maxReplicas: 6
      weight: 1
    B:
      minReplicas: 4
      maxReplicas: 8
      weight: 2
```
# 安装
使用kind来在本机创建多个k8s集群的方式测试。通过kind创建两个k8s集群 kind-cluster1和kind-cluster2。
​

安装kubefedctl
```powershell
wget https://github.com/kubernetes-sigs/kubefed/releases/download/v0.9.0/kubefedctl-0.9.0-linux-amd64.tgz
tar zvxf kubefedctl-0.9.0-linux-amd64.tgz
chmod u+x kubefedctl
mv kubefedctl /usr/local/bin/
```
在第一个集群安装KubeFed，在第一个集群执行如下的命令
```powershell
$ helm repo add kubefed-charts https://raw.githubusercontent.com/kubernetes-sigs/kubefed/master/charts
$ helm repo list
$ helm --namespace kube-federation-system upgrade -i kubefed kubefed-charts/kubefed --version=0.9.0 --create-namespace

# 会安装如下两个deployment，其中一个是controller，另外一个是webhook
$ kubectl  get deploy -n kube-federation-system
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
kubefed-admission-webhook    1/1     1            1           7m40s
kubefed-controller-manager   2/2     2            2           7m40s
```
# 子集群注册
将cluster1和cluster2集群加入到cluster1中
```powershell
kubefedctl join kind-cluster1 --cluster-context kind-cluster1 --host-cluster-context kind-cluster1 --v=2
kubefedctl join kind-cluster2 --cluster-context kind-cluster2 --host-cluster-context kind-cluster1 --v=2
```
在第一个集群可以看到在kube-federation-system中创建出了两个新的kubefedcluster对象
```powershell
$ kubectl -n kube-federation-system get kubefedcluster
NAME            AGE   READY
kind-cluster1   17s   True
kind-cluster2   16s   True

$ apiVersion: core.kubefed.io/v1beta1
kind: KubeFedCluster
metadata:
  name: kind-cluster1
  namespace: kube-federation-system
spec:
  apiEndpoint: https://172.21.115.165:6443
  caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1USXlOREUyTlRNME9Wb1hEVE14TVRJeU1qRTJOVE0wT1Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTmxjCnF3UHd5cHNFc2M0aXB3QnFuRndDN044RXo2Slhqajh3ZzRybnlXTDNSdlZ0cmVua1Nha1VyYlRjZWVIck9lQTUKWGlNUVo2T1FBY25tUGU0Q2NWSkFoL2ZLQzBkeU9uL0ZZeXgyQXppRjBCK1ZNaUFhK2dvME1VMmhMZ1R5eVFGdQpDbWFmUGtsNmJxZUFJNCtCajZJUWRqY3dVMHBjY3lrNGhSTUxnQmhnTUh4NWkzVkpQckQ2Y284dHcwVnllWncyCkdDUlh2ZzlSd0QweUs5eitOVS9LVS83QjBiMTBvekpNRlVJMktPZmI4N1RkQ0h2NmlBdlVRYVdKc1BqQ0M3MzQKcnBma1ZGZXB2S2liKy9lUVBFaDE4VE5qaitveEVGMnl0Vmo2ZWVKeFc3VVZrbit0T3BxdGpEeUtKVDllNnlwdAp6U1VDTnRZWTAzckdhNTlLczBVQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZOTnRQU3hsMlMxUldOb1BCeXNIcHFoRXJONlVNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFDME0vaktCL2dqcWRSNlRLNXJQVktSSFU2cy8xTTh2eTY2WFBiVEx4M0srazdicUFWdAoxVUFzcUZDZmNrVk91WHY3eFQ0ZHIzVzdtMHE1bDYzRGg3ZDJRZDNiQ00zY2FuclpNd01OM0lSMlpmYzR0VlBGCnRTMFFONElTa0hsYnBudXQxb0F3cy9CaXNwaXNRQ0VtbHF3Zy9xbmdPMStlWWRoWm5vRW40SEFDamF4Slc5MS8KNXlOR1pKRXdia2dYQTVCbSs3VEZRL2FiYnp5a1JvOWtTMnl5c29pMnVzcUg0ZnlVS0ZWK2RETnp3Ujh0ck16cgpjWkRBNHpaVFB1WGlYRkVsWlNRa2NJSGIyV0VsYmZNRGpKNjlyVG5wakJCOWNPQ25jaHVmK0xiOXQwN1lJQ01wCmNlK0prNVp2RElRUFlKK3hTeGdRaVJxMTFraWlKcFE4Wm1FWgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  proxyURL: ""
  secretRef:
    name: kind-cluster1-dbxns
status:
  conditions:
  - lastProbeTime: "2021-12-24T17:00:47Z"
    lastTransitionTime: "2021-12-24T17:00:07Z"
    message: /healthz responded with ok
    reason: ClusterReady
    status: "True"
    type: Ready
```
要想将cluster2从主集群中移除，可以执行 kubefedctl unjoin kind-cluster2 --cluster-context kind-cluster2 --host-cluster-context kind-cluster1 --v=2
# 应用发布
## 集群联邦API
kubefed将资源分为普通k8s资源类型和联邦的资源类型。在默认的场景下，kubefed已经内置将很多资源类型做了集群联邦。
```powershell
$ k get FederatedTypeConfig -n kube-federation-system
NAME                                            AGE
clusterrolebindings.rbac.authorization.k8s.io   19h
clusterroles.rbac.authorization.k8s.io          19h
configmaps                                      19h
deployments.apps                                19h
ingresses.extensions                            19h
jobs.batch                                      19h
namespaces                                      19h
replicasets.apps                                19h
secrets                                         19h
serviceaccounts                                 19h
services                                        19h
```
将crd资源类型集群联邦，执行 kubefedctl enable 命令
```powershell
$ kubefedctl enable customresourcedefinitions
I1224 20:32:54.537112  687543 util.go:141] Api resource found.
customresourcedefinition.apiextensions.k8s.io/federatedcustomresourcedefinitions.types.kubefed.io created
federatedtypeconfig.core.kubefed.io/customresourcedefinitions.apiextensions.k8s.io created in namespace kube-federation-system
```
可以看到会多出一个名字为federatedcustomresourcedefinitions.types.kubefed.io 的CRD 资源，同时会新创建一个FederatedTypeConfig类型的资源。当创建了FederatedTypeConfig后，就可以通过创建federatedcustomresourcedefinition类型的实例来向各个子集群发布CRD资源了。
```powershell
$ k get crd federatedcustomresourcedefinitions.types.kubefed.io
NAME                                                  CREATED AT
federatedcustomresourcedefinitions.types.kubefed.io   2021-12-24T12:32:54Z

$ k get FederatedTypeConfig -n kube-federation-system customresourcedefinitions.apiextensions.k8s.io -o yaml
apiVersion: core.kubefed.io/v1beta1
kind: FederatedTypeConfig
metadata:
  finalizers:
  - core.kubefed.io/federated-type-config
  name: customresourcedefinitions.apiextensions.k8s.io
  namespace: kube-federation-system
spec:
  federatedType:
    group: types.kubefed.io
    kind: FederatedCustomResourceDefinition
    pluralName: federatedcustomresourcedefinitions
    scope: Cluster
    version: v1beta1
  propagation: Enabled
  targetType:
    group: apiextensions.k8s.io
    kind: CustomResourceDefinition
    pluralName: customresourcedefinitions
    scope: Cluster
    version: v1
status:
  observedGeneration: 1
  propagationController: Running
  statusController: NotRunning
```
在example/sample1下包含了很多例子，可以直接参考。接下来使用sample1中的例子进行试验。
​

在父集群创建namespace test-namespace
```powershell
$ kubectl create -f namespace.yaml
namespace/test-namespace created
```
在主集群中创建federatednamespace资源
```powershell
$ k apply -f federatednamespace.yaml
federatednamespace.types.kubefed.io/test-namespace created
```
在子集群中即可查询到新创建的namespace资源
```powershell
$ k get ns test-namespace
NAME             STATUS   AGE
test-namespace   Active   4m49s
```
其他的对象均可以按照跟上述namespace同样的方式来创建，比较特殊的对象为clusterrulebinding，该对象在默认情况下没有联邦api FederatedClusterRoleBinding，因此需要手工先创建FederatedClusterRoleBinding联邦api类型。
```powershell
$ kubefedctl enable clusterrolebinding
I1225 01:46:42.779166  818254 util.go:141] Api resource found.
customresourcedefinition.apiextensions.k8s.io/federatedclusterrolebindings.types.kubefed.io created
federatedtypeconfig.core.kubefed.io/clusterrolebindings.rbac.authorization.k8s.io created in namespace kube-federation-system
```
在创建完成FederatedClusterRoleBinding联邦api类型后，即可以创建出FederatedClusterRoleBinding类型的对象。
## 子集群的个性化配置
通过Fedrated<Kind>中的spec.overrides来完成，可以覆盖spec.template中的内容
```powershell
apiVersion: types.kubefed.io/v1beta1
kind: FederatedDeployment
metadata:
  name: test-deployment
  namespace: test-namespace
spec:
  template:
    metadata:
      labels:
        app: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - image: nginx
            name: nginx
  placement:
    clusters:
    - name: cluster2
    - name: cluster1
  overrides:
  - clusterName: cluster2
    clusterOverrides:
    - path: "/spec/replicas"
      value: 5
    - path: "/spec/template/spec/containers/0/image"
      value: "nginx:1.17.0-alpine"
    - path: "/metadata/annotations"
      op: "add"
      value:
        foo: bar
    - path: "/metadata/annotations/foo"
      op: "remove"
```

## 多集群应用调度
### 发布应用到特定的子集群
在Fedrated<Kind>的spec.placement.clusters中，定义了要发布到哪个子集群的信息。
```powershell
placement:
    clusters:
    - name: cluster2
    - name: cluster1
```
不过上述方法需要指定特定的集群，为了有更丰富的灵活性，kubefed还提供了label selector的机制，可以提供spec.placement.clusterSelector来指定一组集群。
```powershell
spec:
  placement:
    clusters: []
    clusterSelector:
      matchLabels:
        foo: bar
```
标签选择器通过跟kubefedclusters对象的label来进行匹配，可以执行下述命令给kubefedclusters来打标签。
```powershell
kubectl label kubefedclusters -n kube-federation-system kind-cluster1 foo=bar
```

### 子集群差异化调度
上述调度功能被选择的子集群为平等关系，如果要想子集群能够有所差异，可以使用ReplicaSchedulingPreference来完成，目前仅支持deployment和replicaset两种类型的资源，还不支持statefulset。
```powershell
apiVersion: scheduling.kubefed.io/v1alpha1
kind: ReplicaSchedulingPreference
metadata:
  name: test-deployment
  namespace: test-ns
spec:
  targetKind: FederatedDeployment
  totalReplicas: 9
  clusters:
    A:
      minReplicas: 4
      maxReplicas: 6
      weight: 1
    B:
      minReplicas: 4
      maxReplicas: 8
      weight: 2
```
spec.targetKind用来指定管理的类型，仅支持FederatedDeployment和FederatedReplicaset。
spec.totalReplicas用来指定所有子集群的副本数总和。优先级要高于FederatedDeployment和FederatedReplicaset的spec.template中指定的副本数。
spec.clusters用来指定每个子集群的最大、最小副本数和权重。
spec.rebalance自动调整整个集群中的副本数，比如一个集群中的pod挂掉后，可以将pod迁移到另外的集群中。

# 子集群之间的交互
无

# 缺点
1. 父集群充当了所有集群的单一控制面
1. 通过联邦CRD来管理资源，无法直接使用k8s原生的资源，集群间维护CRD版本和API版本一致性导致升级比较复杂。

# 参考资料

- 官方文档：[https://github.com/kubernetes-sigs/kubefed/blob/master/docs/userguide.md#using-cluster-selector](https://github.com/kubernetes-sigs/kubefed/blob/master/docs/userguide.md#using-cluster-selector)
