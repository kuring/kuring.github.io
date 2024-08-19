---
title: 深入解析 Kubernetes 驱逐机制：确保高可用的关键一环
---
本文修改自：[Kubernetes教程(二十)---深入解析 Kubernetes 驱逐机制：确保高可用的关键一环](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#3-%E8%8A%82%E7%82%B9%E5%8E%8B%E5%8A%9B%E9%A9%B1%E9%80%90)

为什么 k8s 集群中的 Node 很少因为 Pod 压力大而崩溃，为什么节点宕机后 Pod 会自动在新的节点拉起继续运行，本文就带大家一起深入解析 k8s 中的驱逐机制，解密该机制是如何提升 k8s 集群高可用的。

---

## [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#1-%E4%BB%80%E4%B9%88%E6%98%AF-pod-%E9%A9%B1%E9%80%90)1. 什么是 Pod 驱逐？

**Pod 驱逐指将指定 Pod 从指定节点上删除，一般在节点资源紧张时触发驱逐以实现资源回收，或者节点宕机时将 Pod 驱逐到其他正常节点**。

Pod 驱逐分为四种情况：

- 1）**用户主动发起**：用户主动调用 API 发起的驱逐请求，比如节点维护时驱逐该节点上所有 Pod，避免节点突然下线对业务造成影响。
- 2）**Kubelet 发起**：周期性检查本节点资源，当资源不足时，按照优先级驱逐部分 pod。
- 3）**kube-controller-manager 发起**：周期性检测所有节点，当节点处于 NotReady 状态超过一段时间后，驱逐该节点上所有 Pod 使其被调度到其他正常节点重新运行。
- 4）**kube-scheduler 发起**：抢占式调度时可能会驱逐低优先级 Pod 给高优先级&抢占式 Pod 腾出空间，是的高优先级 Pod 能够正常调度

可以看出虽然发起者不同，但是目的都是相同的，都是为了保证 Pod 高可用。

其中 kube-scheduler 发起的驱逐，我们放到调度的时候一起分析，本文主要分析另外三种情况下的驱逐。

对于不同发起者，主要分析以下几个问题：

- 驱逐目的，为什么要执行 Pod 驱逐
- 驱逐场景，什么情况下会触发驱逐
- 驱逐对象，什么样的 Pod 会被驱逐

## [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#2-api-initiated-%E9%A9%B1%E9%80%90)2. API-initiated 驱逐

主动调用 API 触发的驱逐也称为 **API-initiated 驱逐**。

[API-initiated 驱逐](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/api-eviction/) 是一个创建 `Eviction` 对象从而触发 pod 优雅终止的过程。

我们可以直接请求`Eviction API` 或者使用客户端来发起请求，例如 kubectl drain 命令。

这将会创建一个 `Eviction` 对象，并导致 APIServer 终止对应 Pod。

> 由 API 发起的驱逐会遵循 PodDisruptionBudget 和 terminationGracePeriodSeconds 的配置。

就是调用 Kube-Apiserver 创建 eviction 对象，就像这样：

|   |
|---|
|```JSON<br>{<br>  "apiVersion": "policy/v1",<br>  "kind": "Eviction",<br>  "metadata": {<br>    "name": "quux",<br>    "namespace": "default"  }<br>}<br>```|

或者直接执行 kubectl drain 命令，效果是一样的。

### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#demo)Demo

创建一个 nginx deploy

|   |
|---|
|```Bash<br>kubectl create deployment nginx --image=nginx --replicas=1<br>```|

查看 Pod 所在节点

|   |
|---|
|```Bash<br>[root@demo-1 ~]# kubectl get po -owide<br>NAME                     READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES<br>nginx-77b4fdf86c-vpkdf   1/1     Running   0          6s    172.25.86.70   demo-2   <none>           <none><br>```|

> 可以看到，当前在 demo-2 节点

接下来我们调用 API 驱逐该 Pod

使用 `kubectl proxy` 开启代理：

|   |
|---|
|```Bash<br># 开一个代理<br>[root@demo-1 ~]# kubectl proxy<br>Starting to serve on 127.0.0.1:8001<br>```|

新开一个终端执行 curl 命令，发起 POST 请求创建一个 Eviction 对象，名字和命名空间必须和要驱逐的 Pod 一致。

|                                                                                                                                                                                                                                                                                                                                                                                              |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ```Bash<br>podName="nginx-77b4fdf86c-vfqbc"<br>namespace="default"<br>curl -X POST -H "Content-Type: application/json" -d "{<br>  \"apiVersion\": \"policy/v1\",<br>  \"kind\": \"Eviction\",<br>  \"metadata\": {<br>    \"name\": \"${podName}\",<br>    \"namespace\": \"${namespace}\"<br>  }<br>}" http://localhost:8001/api/v1/namespaces/${namespace}/pods/${podName}/eviction<br>``` |

输出如下：

|   |
|---|
|```JSON<br>{<br>  "kind": "Status",<br>  "apiVersion": "v1",<br>  "metadata": {},<br>  "status": "Success",<br>  "code": 201<br>}<br>```|

由于该 Pod 是被 deploy 管理的，因此 Pod 会再次被创建出来，查询新 Pod 所在节点：

|   |
|---|
|```Bash<br>[root@demo-1 ~]# kubectl get po -owide<br>NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES<br>nginx-77b4fdf86c-l2nlb   1/1     Running   0          50s   172.25.29.5   demo-3   <none>           <none><br>```|

可以看到，Pod 已经被驱逐到 demo-3 了，使用 `kubectl get po -w` 可以看到驱逐过程：

|   |
|---|
|```Bash<br>[root@demo-1 ~]# kubectl get po -w<br>NAME                     READY   STATUS    RESTARTS   AGE<br>nginx-77b4fdf86c-djvsl   1/1     Running   0          21s<br>nginx-77b4fdf86c-djvsl   1/1     Running   0          32s<br>nginx-77b4fdf86c-djvsl   1/1     Terminating   0          32s<br>nginx-77b4fdf86c-djvsl   1/1     Terminating   0          33s<br>nginx-77b4fdf86c-vfqbc   0/1     Pending       0          1s<br>nginx-77b4fdf86c-vfqbc   0/1     Pending       0          1s<br>nginx-77b4fdf86c-vfqbc   0/1     ContainerCreating   0          1s<br>nginx-77b4fdf86c-djvsl   1/1     Terminating         0          33s<br>nginx-77b4fdf86c-djvsl   0/1     Terminating         0          33s<br>nginx-77b4fdf86c-vfqbc   0/1     ContainerCreating   0          1s<br>nginx-77b4fdf86c-djvsl   0/1     Terminating         0          34s<br>nginx-77b4fdf86c-djvsl   0/1     Terminating         0          34s<br>nginx-77b4fdf86c-djvsl   0/1     Terminating         0          34s<br>nginx-77b4fdf86c-vfqbc   1/1     Running             0          4s<br>```|

旧 Pod 先被终止，然后因为该 Pod 是由 Deploy 管理的，因此 kube-controller-manager 会创建出来一个新 Pod，接着 scheduler 对该 Pod 进行调度，当然新建的 Pod 也有可能被再次调度到原节点。

### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF)使用场景

一般是节点下线维护时会手动把对应节点上 Pod 全部驱逐开，以避免突然的节点下线对业务造成影响。

`kubectl drain` 命令就是使用 Eviction API 实现的。

一般节点下线流程为：

- Kubectl drain 驱逐该节点上所有 Pod
    - Drain 命令也会先把该节点更新为 `Unschedulable` 状态，因此不需要先手动执行 kubectl cordon 了
    - 当然也可以手动一个一个 Pod 驱逐以降低对业务的影响，不过 drain 会保证 PDB，因此，如果配置了 PDB 则不用担心
- Kubectl delete node 删除该节点
- 节点关机下线

## [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#3-%E8%8A%82%E7%82%B9%E5%8E%8B%E5%8A%9B%E9%A9%B1%E9%80%90)3. 节点压力驱逐

**节点压力驱逐是 kubelet 主动终止** **Pod** **以回收节点资源的过程。**

kubelet 监控集群节点的内存、磁盘空间和文件系统的 inode 等资源。 当这些资源中的一个或者多个达到特定的消耗水平， kubelet 可以主动驱逐节点上一个或者多个 Pod ，以回收资源防止节点最终崩溃。

**需要注意的是**：由 Kubelet 发起的驱逐并不会遵循 PodDisruptionBudget 和 terminationGracePeriodSeconds 的配置。

> 对 k8s 来说，保证节点资源充足优先级更高，牺牲小部分 Pod 来保证剩余的大部分 Pod。

另外 Kubelet 在真正驱逐 Pod 之前会执行一次垃圾回收，尝试回收节点资源，如果回收后资源充足了，就可以避免驱逐 Pod。

### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E9%A9%B1%E9%80%90%E4%BF%A1%E5%8F%B7%E5%92%8C%E9%98%88%E5%80%BC%E4%BB%80%E4%B9%88%E6%97%B6%E5%80%99%E4%BC%9A%E9%A9%B1%E9%80%90-pod)驱逐信号和阈值：什么时候会驱逐 Pod?

kubelet 使用各种参数来做出驱逐决定，具体包含以下 3 个部分：

- 1）驱逐信号
- 2）驱逐条件
- 3）监控间隔

#### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E9%A9%B1%E9%80%90%E4%BF%A1%E5%8F%B7%E4%B8%8E%E8%8A%82%E7%82%B9-condition)驱逐信号与节点 condition

##### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E9%A9%B1%E9%80%90%E4%BF%A1%E5%8F%B7)驱逐信号

**Kubelet 使用驱逐信号来代表特定资源在特定时间点的状态**，根据不同资源，kubelet 中定义了 5 种驱逐信号。

> 驱逐信号这个词感觉有点迷，实际看下来就是检测了这几种资源的余量用于判断是否需要驱逐 Pod

在 Linux 系统中，5 种信号分别为：

|驱逐信号|描述|
|---|---|
|memory.available|memory.available := node.status.capacity[memory] - node.stats.memory.workingSet|
|nodefs.available|nodefs.available := node.stats.fs.available|
|nodefs.inodesFree|nodefs.inodesFree := node.stats.fs.inodesFree|
|imagefs.available|imagefs.available := node.stats.runtime.imagefs.available|
|imagefs.inodesFree|imagefs.inodesFree := node.stats.runtime.imagefs.inodesFree|
|pid.available|pid.available := node.stats.rlimit.maxpid - node.stats.rlimit.curproc|

可以看到，主要是对 `内存`、`磁盘`、`PID` 这三种类型资源进行检测。其中磁盘资源又分为两类：

1. `nodefs`：节点的主要文件系统，用于本地磁盘卷、不受内存支持的 emptyDir 卷、日志存储等。 例如，`nodefs` 包含 `/var/lib/kubelet/`。
2. `imagefs`：可选文件系统，供容器运行时存储容器镜像和容器可写层。

##### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E8%8A%82%E7%82%B9-condition-%E4%B8%8E%E6%B1%A1%E7%82%B9)节点 condition 与污点

除了用驱逐信号来判断是否需要驱逐 Pod 之外，Kubelet 还会把驱逐信号反应为节点的状态。

> 即：更新到 node 对象的 status.conditions 字段里，此时 node 的状态会变为 NotReady。

对应关系如下：

|Node Condition|Eviction Signal|Description|
|---|---|---|
|MemoryPressure|memory.available|节点可用内存余量满足驱逐阈值|
|DiskPressure|nodefs.available, nodefs.inodesFree, imagefs.available, or imagefs.inodesFree|节点主文件系统或者镜像文件系统剩余磁盘空间或者 inodes 数量满足驱逐阈值|
|PIDPressure|pid.available|节点上可用进程标识符(processes identifiers) 低于驱逐阈值|

总的来说就是节点上对应资源不足时 kubelet 就会被节点打上对应的标记。

同时 `control plane(具体为 node controller)` 会自动把节点上相关 condition 转换为 taint 标记，具体见：[#taint-nodes-by-condition](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/#taint-nodes-by-condition)，这样可以避免把 Pod 调度到本来就资源不足的 Pod 上。

> 如果不给资源不足的节点打上标记，可能发生 Pod 刚调度过去就被驱逐的情况。

**整体运作流程**：

- 1）首先 Kubelet 检测到节点上剩余磁盘空间不足，更新 Node Condition 增加 `DiskPressure` 状态
- 2）然后 node controller 自动将 Condition 转换为污点 `node.kubernetes.io/disk-pressure`
- 3）最后 Pod 调度的时候，由于没有添加对应的容忍，因此会优先调度到其他节点，或者最终调度失败

有时候节点状态处于阈值附近上下波动，导致软驱逐条件也在 true 和 false 之前反复切换，为了过滤掉这种误差， kubelet 提供了 `eviction-pressure-transition-period` 参数来限制，节点状态转换前必须要等待对应的时间，默认为 5m。

> tips：一般这种检测状态的为了提升稳定性，都会给一个静默期，比如 k8s 中的 HPA 扩缩容也会设置静默期，防止频繁触发扩缩容动作。

#### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E9%A9%B1%E9%80%90%E9%98%88%E5%80%BC%E5%88%A4%E6%96%AD%E6%9D%A1%E4%BB%B6)驱逐阈值：判断条件

拿到资源当前状态之后，就可以根据阈值判断是否需要触发驱逐动作了。

Kubelet 使用 `[eviction-signal][operator][quantity]` 格式定义驱逐条件，其中：

- `eviction-signal` 是要使用的驱逐信号
    - 比如剩余内存 memory.available
- `operator` 是你想要的关系运算符
    - 比如 `<`（小于）。
- `quantity` 是驱逐条件数量，例如 `1Gi`。
    - `quantity` 的值必须与 Kubernetes 使用的数量表示相匹配。
    - 你可以使用文字值或百分比（`%`）。

例如，如果一个节点的总内存为 10GiB 并且你希望在可用内存低于 1GiB 时触发驱逐， 则可以将驱逐条件定义为 `memory.available<10%` 或 `memory.available< 1G`。

根据紧急程度驱逐条件又分为**软驱逐 eviction-soft** 和 **硬驱逐 eviction-hard**：

##### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E8%BD%AF%E9%A9%B1%E9%80%90-eviction-soft)软驱逐 eviction-soft

一般驱逐条件比较保守，此时还可以等待 Pod 优雅终止，需要配置以下参数

- `eviction-soft`：软驱逐条件，例如 `memory.available<1.5Gi`
- `eviction-soft-grace-period`：软驱逐宽限期，需要保持驱逐条件这么久之后才开始驱逐 Pod
    - 必须指定，否则 kubelet 启动时会直接报错
- `eviction-max-pod-grace-period`：软驱逐最大 Pod 宽限期，驱逐 Pod 的时候给 Pod 优雅终止的时间

在 [Pod 生命周期部分](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination) 我们可以配置 `spec.terminationGracePeriodSeconds`来控制 Pod 优雅终止时间，就像这样：

|   |
|---|
|```YAML<br>apiVersion: v1<br>kind: Pod<br>metadata:<br>  name: example<br>spec:<br>  containers:<br>  - name: my-container<br>    image: my-image<br>  terminationGracePeriodSeconds: 60  # 设置终止期限为 60 秒<br>```|

然后驱逐这里又配置了一个 `eviction-max-pod-grace-period`,实际驱逐发生时，**Kubelet 会取二者中的较小值来作为最终优雅终止宽限期**。

##### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E7%A1%AC%E9%A9%B1%E9%80%90-eviction-hard)硬驱逐 eviction-hard

相比之下则是比较紧急，配置条件都很极限，在往上节点可能会崩溃的那种。

比如 内存小于 100Mi 这种，因此硬驱逐没有容忍时间，需要配置硬驱逐条件 `eviction-hard`。

只要满足驱逐条件 kubelet 就会立马将 pod kill 掉，而不是发送 SIGTERM 信号。只

Kubelet 也是提供了以下的默认硬驱逐条件：

- `memory.available<100Mi`
- `nodefs.available<10%`
- `imagefs.available<15%`
- `nodefs.inodesFree<5%` (Linux nodes)

例如下面这个配置：

- eviction-soft： memory.available < 1 Gi
- eviction-soft-grace-period：60s
- eviction-max-pod-grace-period：30s
- eviction-hard：memory.available < 100 Mi

当节点可用内存余量低于 1Gi 连续持续 60s 之后，kubelet 就会开始软驱逐，给被选择的 Pod 发送 SIGTERM，使其优化终止，如果终止时间超过 30s 则强制 kill。

如果节点内存可用余量低于 100Mi，则 kubelet 进入硬驱逐，立马 Kill 掉被选中的 Pod。

#### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E7%8A%B6%E6%80%81%E6%A3%80%E6%B5%8B--%E9%A9%B1%E9%80%90%E9%A2%91%E7%8E%87)状态检测 & 驱逐频率

Kubelet 默认每 10s 会检测一次节点状态，即驱逐判断条件为 10s 一次，当然也可以通过`housekeeping-interval` 参数进行配置。

### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E5%9B%9E%E6%94%B6%E8%8A%82%E7%82%B9%E7%BA%A7%E8%B5%84%E6%BA%90)回收节点级资源

为了提升稳定性，减少 Pod 驱逐次数，Kubelet 在执行驱逐前会进行一次垃圾回收。如果本地垃圾回收后资源充足了就不再驱逐。

> 具体 kubelet 垃圾回收详情见：[Kubernetes 教程(十九)—Kubelet 垃圾回收机制揭秘：本地镜像离奇消失之谜](https://www.lixueduan.com/posts/kubernetes/19-kubelet-gc/)
> 
> 到公众号阅读：[Kubelet 垃圾回收机制揭秘：本地镜像离奇消失之谜](https://mp.weixin.qq.com/s?__biz=Mzk0NzE5OTQyOQ==&mid=2247483849&idx=1&sn=17d3a278a8a11b3def5ba635f03fcea4&chksm=c37bcd63f40c4475078fdfb4ff9dd85df9225866b8aa7044bb7c39229b286dfd3e2570075746&token=2095298004&lang=zh_CN#rd)

对于磁盘资源可以通过驱逐 Pod 以外的方式进行回收：

如果节点有一个专用的 `imagefs` 文件系统供容器运行时使用，kubelet 会执行以下操作：

- 如果 `nodefs` 文件系统满足驱逐条件，kubelet 垃圾收集死亡 Pod 和容器。
- 如果 `imagefs` 文件系统满足驱逐条件，kubelet 将删除所有未使用的镜像。

如果节点只有一个满足驱逐条件的 `nodefs` 文件系统， kubelet 按以下顺序释放磁盘空间：

1. 对死亡的 Pod 和容器进行垃圾收集
2. 删除未使用的镜像

对于 CPU、内存资源则是只能驱逐 Pod 方式进行回收。

### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E9%A9%B1%E9%80%90%E5%AF%B9%E8%B1%A1%E5%93%AA%E4%BA%9B-pod-%E4%BC%9A%E8%A2%AB%E9%A9%B1%E9%80%90)驱逐对象：哪些 Pod 会被驱逐

如果进行垃圾回收后，节点资源也满足驱逐条件，那么为了保证当前节点不被压垮， kubelet 只能驱逐 Pod 了。

根据触发驱逐的资源不同，驱逐目标筛选逻辑也有不同。

#### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E5%86%85%E5%AD%98%E8%B5%84%E6%BA%90%E5%AF%BC%E8%87%B4%E7%9A%84%E9%A9%B1%E9%80%90)内存资源导致的驱逐

对于内存资源 kubelet 使用以下参数来确定 Pod 驱逐顺序：

- 1）**Pod** **使用的资源是否超过请求值**：即当前占用资源是否高于 yaml 中指定的 request 值
- 2）**Pod** **的优先级**：这里不是 QoS 优先级，而是 [priority-class](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/pod-priority-preemption/) 这个优先级
- 3）**Pod** **使用资源相对于请求资源的百分比**：百分比越高则越容易别驱逐，比如请求 100Mi，使用了 60Mi 则占用了 60%，如果没有 limit 导致 Pod 使用了 200Mi，那就是 200%，

因此，虽然没有严格按照 QoS 来排序，但是整个驱逐顺序和 Pod 的 QoS 是有很大关系的：

- 首先考虑资源使用量超过其请求的 Qos 级别为 `BestEffort` 或 `Burstable` 的 Pod。 这些 Pod 会根据它们的优先级以及它们的资源使用级别超过其请求的程度被驱逐。
- 资源使用量少于请求量的 `Guaranteed` 级别的 Pod 和 `Burstable` Pod 根据其优先级被最后驱逐。

#### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#inode--pid-%E5%AF%BC%E8%87%B4%E7%9A%84%E9%A9%B1%E9%80%90)inode & pid 导致的驱逐

当 kubelet 因 **inode** 或 **进程标识符(pid)** 不足而驱逐 Pod 时， 它使用 Pod 的相对优先级来确定驱逐顺序，因为 inode 和 PID 没有对应的请求字段。

相对优先级排序方式如下：

节点有 `imagefs`

- 如果 `nodefs` 触发驱逐， kubelet 会根据 `nodefs` 使用情况（`本地卷 + 所有容器的日志`）对 Pod 进行排序。
- 如果 `imagefs` 触发驱逐，kubelet 会根据所有容器的可写层使用情况对 Pod 进行排序。

节点没有 `imagefs`

- 如果 `nodefs` 触发驱逐， kubelet 会根据磁盘总用量（`本地卷 + 日志和所有容器的可写层`）对 Pod 进行排序。

即：inode、pidk 导致的驱逐，Kubelet 会优先驱逐磁盘消耗大的 Pod，而不是根据 Pod Qos 来。

#### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E5%B0%8F%E7%BB%93)小结

Kubelet 并没有使用 QoS 来作为驱逐顺序，但是对于**内存资源回收**的场景，驱逐顺序和 QoS 是相差不大的。不过对于 **磁盘和 PID** 资源的回收则完全不一样的，会优先考虑驱逐磁盘占用多的 Pod，即使 Pod QoS 等级为 `Guaranteed`。

> 毕竟不是所有资源都有 request 和 limit，只能先驱逐占用量大的

#### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E6%9C%80%E5%B0%91%E8%B5%84%E6%BA%90%E5%9B%9E%E6%94%B6%E9%87%8F)最少资源回收量

为了保证尽量少的 Pod，kubelet 每次只会驱逐一个 Pod，驱逐后就会判断一次资源是否充足。

这样可能导致该节点资源一直处于驱逐阈值，反复达到驱逐条件从而触发多次驱逐，kubelet 也提供了参数 `--eviction-minimum-reclaim` 来指定每次驱逐最低回收资源，达到该值后才停止驱逐。从而减少触发驱逐的次数。

### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#oom-killer)OOM Killer

这里也顺带提一下和资源相关的另一个功能：`OOM Killer`。

> 注意：OOM Killer 实际上是一个 Linux 内核特性，并不是 kubernetes 的一部分。

OOM Killer 内核进程会监控系统里的所有进程，并根据公式`有效 oom_score = oom_score + oom_score_adj` 计算出每个进程的的 OOM 得分，随后内存不足时直接 Kill 掉得分最高的进程。

**其中 oom_score_adj 就是内核为用户提供的灵活性，k8s 中则是根据** **Pod** **的** **QoS** **为 Pod 中的容器进程设置了不同的 oom_score_adj 来调整优先级，**具体 oom_score_adj 计算公式如下：

|服务质量|oom_score_adj|
|---|---|
|Guaranteed|-997|
|BestEffort|1000|
|Burstable|min(max(2, 1000 - (1000 * memoryRequestBytes) / machineMemoryCapacityBytes), 999)|

> 可以看到 Pod 的 QoS 对这个还是有影响的，Guaranteed 级别的 Pod oom_score_adj 固定为 -997，保证了该级别 Pod 中的容器不会被轻易 Kill 掉。

然后根据 Pod 在节点上使用的内存百分比计算出一个 `oom_score`。

最后 `oom_score`加上 `oom_score_adj` 得到每个容器有效的 `oom_score`。 内存快消耗完时，OOM Killer 会根据该算法找出得分最高的容器并 Kill 掉。

这意味着低 QoS Pod 中相对于其调度请求消耗内存较多的容器，将首先被杀死。

当进程被 OOM Killer Kill 掉之后，会生成内核事件，然后 kubelet 会识别该事件并将对应 Pod 设置为 OOMKilled 状态。

> 注意：容器最终是被 OOM Killer 这个内核进程 Kill 的，而不是 kubelet。

就像这样：

|   |
|---|
|```Bash<br>State:          Running<br>       Started:      Thu, 10 Oct 2019 11:14:13 +0200<br>       Last State:   Terminated<br>       Reason:       OOMKilled<br>       Exit Code:    137<br>       ...<br>```|

注意：与 Pod 驱逐不同，如果容器被 OOM 杀死， kubelet 可以根据其 `restartPolicy` 重新启动它。

## [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#4-kube-controller-manger-%E8%8A%82%E7%82%B9%E7%A6%BB%E7%BA%BF%E9%A9%B1%E9%80%90)4. Kube-controller-manger 节点离线驱逐

除了 kubelet 会根据节点压力进行驱逐外，Kube-controller-manger 中也有驱逐逻辑。

**Kube-controller-manager 周期性检查节点状态，每当节点状态为 NotReady，并且超出 podEvictionTimeout 时间后，就把该节点上的 pod 全部驱逐到其它节点。**

> k8s node NotReady 的检测逻辑：节点的状态 `Ready` 或 `NotReady` 是依据节点的 `conditions` 字段中的 `Ready` 条目。如果 `Ready` 条目的 `Status` 为 `False` 或 `Unknown`，该节点被认为是 `NotReady`。

其中具体驱逐速度还受驱逐速度参数，集群大小等的影响。提供了以下启动参数控制 eviction。

比如 kubelet 来不及驱逐，Node 上的资源就被恶意 Pod 耗尽导致 Node 最终宕机，或者别的原因导致 Node 进入 NotReady 状态。

此时 control-plane 无法正常和该节点通信，拿不到节点上的 Pod 的状态，因此会做最坏的打算，认为这些 Pod 都挂了，因此会将该 Node 上的所有 Pod 都驱逐到其他节点重新跑起来，以保证高可用。

### [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#%E7%9B%B8%E5%85%B3%E5%8F%82%E6%95%B0)相关参数

- **pod-eviction-timeout**：即当节点宕机该事件间隔后，开始 eviction 机制，驱赶宕机节点上的 Pod，默认为 5min
- **node-eviction-rate:** 驱赶速率，即驱赶 Node 的速率，由令牌桶流控算法实现，默认为 0.1，即每秒驱赶 0.1 个节点，注意这里不是驱赶 Pod 的速率，而是驱赶节点的速率。相当于每隔 10s，清空一个节点
- **secondary-node-eviction-rate**: 二级驱赶速率，当集群中宕机节点过多时，相应的驱赶速率也降低，默认为 0.01
- **unhealthy-zone-threshold**：不健康 zone 阈值，会影响什么时候开启二级驱赶速率，默认为 0.55，即当该 zone 中节点宕机数目超过 55%，而认为该 zone 不健康
- **large-cluster-size-threshold**：大集群阈值，当该 zone 的节点多于该阈值时，则认为该 zone 是一个大集群。大集群节点宕机数目超过 55% 时，则将驱赶速率降为 0.0.1，假如是小集群，则将速率直接降为 0

## [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#5-kube-scheduler-%E8%B0%83%E5%BA%A6%E6%8A%A2%E5%8D%A0%E9%A9%B1%E9%80%90)5. kube-scheduler 调度抢占驱逐

在高优先级 Pod 无法调度时，kube-scheduler 也会驱逐低优先级 Pod 以为高优先级 Pod 腾出空间完成调度。

我们可以通过 PriorityClass 来设置 Pod 的优先级，首先是创建 PriorityClass 对象：
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "此优先级类应仅用于 XYZ 服务 Pod。"
```

然后在 Pod 对象上 **priorityClassName** 参数指定，当然更多情况下是在 Deployment 的 Pod 模板中指定该字段。

> 类似于 pvc 指定 storageclass

其中`preemptionPolicy` 字段控制抢占策略，默认为 `PreemptLowerPriority`，即：允许抢占低优先级 Pod，如果不希望抢占则可以配置为 `Never`。

Pod 创建后就会进入等待调度队列，Scheduler 从队列中取出 Pod 并尝试将其调度到一个合适的节点上。

如果没有找到一个满足 Pod 所有条件的节点，就会触发抢占逻辑。

假设等待调度的 Pod 为 P，抢占逻辑试图找到一个节点， 在该节点中删除一个或多个优先级低于 P 的 Pod，则可以将 P 调度到该节点上。

如果找到这样的节点，一个或多个优先级较低的 Pod 会被从节点中驱逐。 被驱逐的 Pod 消失后，P 可以被调度到该节点上。

## [](https://www.lixueduan.com/posts/kubernetes/20-pod-eviction/#6-%E5%B0%8F%E7%BB%93)6. 小结

k8s 中的驱逐包含 4 种触发方式：

- 1）**API-initiated** 驱逐：手动调用 API 方式驱逐，一般是节点下线维护时使用，通过 `kubectl drain` 方式触发
- 2）**kubelet 节点压力驱逐**：节点资源不足时 kubelet 自动触发，驱逐 Pod 以保证节点资源不被耗尽，提升节点可用性。
- 3）**kube-controller-manger 节点离线驱逐**：节点宕机时将 Pod 驱逐到其他正常节点重新拉起，保证业务高可用
- 4）**kube-scheduler 抢占式调度驱逐**：驱逐低优先级 Pod 给高优先级&抢占式 Pod 腾出空间，保证高优先级 Pod 的高可用。

可以看到整个驱逐功能主要有两个用途：

- 1）节点下线维护时进行节点排水，避免节点突然下线所有 Pod 挂掉对业务造成影响
- 2）保证 Node 以及 Pod 的高可用

总的来说这个功能就是为了保证 Kubernetes 集群和上面运行的 Pod 的高可用。

最后，用一个表格来回答一下开篇的问题：

| \|驱逐场景                             | 驱逐目的                   | 驱逐对象                                        |                                                                                                                    |
| ---------------------------------- | ---------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **API-initiated 驱逐**               | 节点下线维护                 | 提前将 pod 驱逐到其他节点，避免节点突然下线导致所有 Pod 挂掉，对业务造成影响 | 目标节点上的所有 Pod，其中 DaemonSet、使用 LocalStorage 的 Pod 以及没有被 controller（deploy、StatefulSet 等）管理的，单独启动的 Pod 这些比较特殊，默认不会驱逐。 |
| **Kubelet 节点压力驱逐**                 | 节点资源不足时                | 驱逐 Pod 到其他节点以释放资源，保证当前节点不会因为资源耗尽而宕机         | 按照规则排序，内存资源不足时一般按照 QoS 等级驱逐，inodes 和 pid 资源不足时则按照资源占用量进行驱逐(忽略 Pod QoS)                                             |
| **kube-controller-manager 节点离线驱逐** | 节点处于 NotReady 状态超过一定时间 | 将 Pod 从 NotReady 节点驱逐到其他正常节点，保证 Pod 能够正常运行  | NotReady 节点上的所有 Pod                                                                                                |
| **kube-scheduler 调度抢占驱逐**          | 抢占式高可用 Pod 无法调度时       | 驱逐低优先级 Pod 给高优先级抢占式 Pod 腾出空间，使其能够正常调度       | 低优先级 Pod（注：这里的优先级不是 QoS 而是 priorityClass）。                                                                         |

## 7. pod 被驱逐后的行为

StatefulSet 管理的 pod 如果被驱逐后，pod 不会一直保持 Evicted 状态，会由 StatefulSet 控制器自动删除被驱逐的 pod。

Deployment 管理的 pod 行为，会看到 Deployment 管理的 pod 会经常出现 Evicted 类型的？
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/20240716222538.png)
