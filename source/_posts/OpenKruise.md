title: OpenKruise调研
date: 2022-03-08 21:14:34
tags:
author:
---
OpenKruise是阿里云开源的一系列基于k8s的扩展组件的集合，其中包含了像增强版的workload、sidecar容器管理、高可用性防护等特性，包含了很多的“黑科技”。

如果k8s的kube-controller-manager组件可以提供非常强的扩展能力，可以实现自定义的Deployment、StatefulSet的controller，而不是使用原生的kube-controller-manager的功能，类似于实现自定义的调度器扩展功能。那么很有可能OpenKruise的实现方案就不再会采用CRD扩展的方式，而是直接在原生的Deployment、StatefulSet等对象上通过annotation的方式来实现。

## 安装
可以直接使用helm的方式安装
```yaml
helm repo add openkruise https://openkruise.github.io/charts/
helm install kruise openkruise/kruise --version 1.0.1
```
安装完成后，可以看到在kruise-system下创建了一个DeamonSet和一个Deployment。并且安装了很多的CRD和webhook组件。
```yaml
$ kubectl  get pod -n kruise-system
NAME                                        READY   STATUS    RESTARTS   AGE
kruise-controller-manager-67878b65d-cv6f4   1/1     Running   0          92s
kruise-controller-manager-67878b65d-jrmnd   1/1     Running   0          92s
kruise-daemon-ktwvv                         1/1     Running   0          92s
kruise-daemon-nf84r                         1/1     Running   0          92s
kruise-daemon-rjs26                         1/1     Running   0          92s
kruise-daemon-vghw4                         1/1     Running   0          92s

$ kubectl get crd | grep kruise.io
advancedcronjobs.apps.kruise.io                           2022-03-05T13:21:39Z
broadcastjobs.apps.kruise.io                              2022-03-05T13:21:39Z
clonesets.apps.kruise.io                                  2022-03-05T13:21:39Z
containerrecreaterequests.apps.kruise.io                  2022-03-05T13:21:39Z
daemonsets.apps.kruise.io                                 2022-03-05T13:21:39Z
imagepulljobs.apps.kruise.io                              2022-03-05T13:21:39Z
nodeimages.apps.kruise.io                                 2022-03-05T13:21:39Z
podunavailablebudgets.policy.kruise.io                    2022-03-05T13:21:39Z
resourcedistributions.apps.kruise.io                      2022-03-05T13:21:39Z
sidecarsets.apps.kruise.io                                2022-03-05T13:21:39Z
statefulsets.apps.kruise.io                               2022-03-05T13:21:39Z
uniteddeployments.apps.kruise.io                          2022-03-05T13:21:39Z
workloadspreads.apps.kruise.io                            2022-03-05T13:21:39Z

$ kubectl  get validatingwebhookconfigurations kruise-validating-webhook-configuration 
NAME                                      WEBHOOKS   AGE
kruise-validating-webhook-configuration   17         17m
$ kubectl  get mutatingwebhookconfigurations kruise-mutating-webhook-configuration 
NAME                                    WEBHOOKS   AGE
kruise-mutating-webhook-configuration   11         17m
```
## 功能
| 大类 | 子类 | 描述 |
| --- | --- | --- |
| 通用工作负载 | CloneSet | 定位是用来代替k8s的Deployment，但做了很多能力的增强。增强的功能点：<br>1. 支持声明pvc，给pod来申请pv。当pod销毁后，pvc会同步销毁。<br>2. 指定pod来进行缩容。<br>3. 流式扩容，可以指定扩容的步长等更高级的库容特性。<br >4. 分批灰度。<br>5. 通过partition回滚。<br>6. 控制pod的升级顺序。<br>7. 发布暂停。<br>8. 原地升级自动镜像预热。<br>9. 生命周期钩子。pod的多个声明周期之间可以读取finalizer，如果finalizer中有指定的值，则controller会停止。该行为作为k8s的一种hook方式，用户可以自定义controller来控制finalizer的行为。 |
| 通用工作负载 | Advanced StatefulSet | 用来取代k8s原生的StatefulSet，很多增强特性跟CloneSet比较类似。<br>1. 原地升级。<br>2. 升级顺序增强。<br>3. 发布暂停。<br>4. 原地升级自动预热。<br>5. 序号跳过。StatefulSet创建的pod的后缀会从0开始依次累加，可以指定某个特定的序号跳过。<br>6. 流式扩容。|
| 通用工作负载 | Advanced DaemonSet | 用来取代k8s原生的DaemonSet。1. 热升级<br>2. 暂停升级<br> |
| 任务工作负载 | BroadcastJob | agent类型的job，每个节点上都会执行 |
| 任务工作负载 | AdvancedCronJob | 原生的CronJob的扩展版本，可以周期性创建BroadcastJob。 |
| Sidecar容器管理 | SidecarSet | 用来管理Sidecar容器，其最核心的功能是支持在pod不重启的情况下Sidecar容器的热升级 |
| 多区域管理 | WorkloadSpread | 将workload按照不同的策略来打散，随着k8s功能不断完善，部分功能k8s已经具备。支持Deployment、ReplicaSet、CloneSet。 |
| 多区域管理 | UnitedDeployment | k8s集群内可能存在不同种类型的node，该特性通过UnitedDeployment对象来管理将一个workload的不同pod分发到不同类型的节点上，并且可以指定不同类型节点的pod副本数。 |
| 增强运维 | 重启一个pod中的某个容器 | 该特性依赖于kurise-daemon组件实现，通过将容器进程停掉，kubelet检测到容器停掉后会自动将容器拉起。停掉容器的方式跟kubelet实现一致。 |
| 增强运维 | 镜像预热 | 通过ImagePullJob CR提供操作入口，底层的实现通过调用CRI的pod image接口实现 |
| 增强运维 | 控制pod中容器的启动顺序 | kruise创建一个ConfigMap，并在pod中注入来挂载该ConfigMap，每个容器使用ConfigMap中的不同key。kruise依次往CM中增加key来实现控制容器启动顺序的目的。 |
| 增强运维 | 资源分发ResourceDestribution | 可以跨namespace来分发CM、Secret，保证每个namespace下均有 |
| 应用安全防护 | 资源删除防护 | 通过webhook技术实现 |
| 应用安全防护 | PodUnavailableBudget | 通过webhook实现的k8s原生的pdb能力的增强，覆盖pdb不具备的场景 |

## 资料

- [https://openkruise.io/zh/docs/installation](https://openkruise.io/zh/docs/installation)
- [如何基于 OpenKruise 打破原生 Kubernetes 中的容器运行时操作局限？](https://mp.weixin.qq.com/s/0YulmrteQSARXHa2NOBLeA)