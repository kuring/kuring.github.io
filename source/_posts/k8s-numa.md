title: k8s 中的 numa 亲和性
date: 2024-03-22 20:08:26
tags:
author:
---
默认的情况下，k8s 对于 pod 在单个节点的资源分配并不会考虑到 NUMA 架构。比如 cpu 默认会采用 cgroup CFS 来做资源的分配，并未考虑到 NUMA 架构。为了提升 pod 的性能，需要 pod 在分配资源时感知 NUMA 架构。<br />为此，k8s 在 kubelet 中通过 CPU Manager、Memory Manager、Device Manager、Topology Manager 等特性对 NUMA 做了支持，支持的 pod QoS 类型要求为 Granteed pod。<br />各特性的支持版本情况如下：

| 特性 | alpha | beta | stable |
| --- | --- | --- | --- |
| CPU Manager |  | 1.12 | 1.26 |
| Memory Manager | 1.21 | 1.22 | - |
| Topology Manager | 1.16 | 1.18 | - |


# CPU Manager
在 k8s 中使用 cgroup 的 CFS 配额来执行 pod 的 CPU 约束，在 CFS 的模式下，pod 可能会运行在不同的核上，会导致 pod 的缓存失效的问题。对于性能要求非常高的 pod，为了提升性能，可以通过 cgroup 中的 cpuset 绑核的特性来提升 pod 的性能。

在 k8s 中仅针对如下的 pod 类型做了绑核操作：

1. 必须为 guaranteed pod 类型。即 pod 需要满足如下两个条件：
   1. pod 中的每个容器都必须指定cpu 和 内存的 request 和 limit。
   2. pod 中的每个容器的 cpu 和内存的 request 和 limit 必须相等。
2. pod 的 cpu request 和 limit 必须为整数。

## kubelet 的参数配置
在 k8s 中仅通过 kubelet 来支持 pod 的绑核操作，跟其他组件无关。<br />kubelet 通过参数 `--cpu-manager-policy` 或者在 kubelet 的配置文件中参数`cpuManagerPolicy` 来配置，支持如下值：

1. none：默认策略。即不执行绑核操作。
2. static：允许为节点上的某些特征的 pod 赋予增强的 cpu 亲和性和独占性。

kubelet 通过参数 `--cpu-manager-reconcile-period` 来指定内存中的 cpu 分配跟 cgroupfs 一致。<br />kubelet 通过参数 `--cpu-manager-policy-options`来微调。该特性通过特性门控 CPUManagerPolicyOptions 来控制。

## 模式之间切换
默认的 kubelet `cpuManagerPolicy` 配置为 none，策略配置位于文件 `/var/lib/kubelet/cpu_manager_state`，文件内容如下：
```json
{"policyName":"none","defaultCpuSet":"","checksum":1353318690}
```
修改 kubelet 的`cpuManagerPolicy` 为 static，将模式从 none 切换为 static，重启 kubelet。发现 kubelet 会启动失败，kubelet 并不能支持仅修改参数就切换模式，报如下错误：
```
Mar 06 15:00:02 iZt4nd5yyw9vfuxn3q2g3tZ kubelet[102800]: E0306 15:00:02.463939  102800 cpu_manager.go:223] "Could not initialize checkpoint manager, please drain node and remove policy state file" err="could not restore state from checkpoint: configured policy \"static\" differs from state checkpoint policy \"none\", please drain this node and delete the CPU manager checkpoint file \"/var/lib/kubelet/cpu_manager_state\" before restarting Kubelet"
Mar 06 15:00:02 iZt4nd5yyw9vfuxn3q2g3tZ kubelet[102800]: E0306 15:00:02.463972  102800 kubelet.go:1392] "Failed to start ContainerManager" err="start cpu manager error: could not restore state from checkpoint: configured policy \"static\" differs from state checkpoint policy \"none\", please drain this node and delete the CPU manager checkpoint file \"/var/lib/kubelet/cpu_manager_state\" before restarting Kubelet"
```
将文件 `/var/lib/kubelet/cpu_manager_state` 删除后，kubelet 即可启动成功，新创建的 `/var/lib/kubelet/cpu_manager_state` 文件内容如下：
```
{"policyName":"static","defaultCpuSet":"0-3","checksum":611748604}
```
可以看到已经存在了绑核的 pod。

如果节点上已经存在符合绑核条件的 pod，在修改配置并重启 kubelet 后即可绑核生效。

## 绑核实践
配置 kubelet 的`cpuManagerPolicy`值为 static，创建 guaranteed pod：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "200Mi"
            cpu: "2"
          requests:
            memory: "200Mi"
            cpu: "2"
```
在 pod 调度的节点上，进入到 /sys/fs/cgroup/cpuset/kubepods.slice 目录下，跟绑核相关的设置均在该目录下，该目录的结构如下：
```
$ ll /sys/fs/cgroup/cpuset/kubepods.slice
-rw-r--r--  1 root root 0 Mar  6 10:31 cgroup.clone_children
-rw-r--r--  1 root root 0 Mar  6 10:31 cgroup.procs
-rw-r--r--  1 root root 0 Mar  6 10:31 cpuset.cpu_exclusive
-rw-r--r--  1 root root 0 Mar  6 10:31 cpuset.cpus
-r--r--r--  1 root root 0 Mar  6 10:31 cpuset.effective_cpus
-r--r--r--  1 root root 0 Mar  6 10:31 cpuset.effective_mems
-rw-r--r--  1 root root 0 Mar  6 10:31 cpuset.mem_exclusive
-rw-r--r--  1 root root 0 Mar  6 10:31 cpuset.mem_hardwall
-rw-r--r--  1 root root 0 Mar  6 10:31 cpuset.memory_migrate
-r--r--r--  1 root root 0 Mar  6 10:31 cpuset.memory_pressure
-rw-r--r--  1 root root 0 Mar  6 10:31 cpuset.memory_spread_page
-rw-r--r--  1 root root 0 Mar  6 10:31 cpuset.memory_spread_slab
-rw-r--r--  1 root root 0 Mar  6 10:31 cpuset.mems
-rw-r--r--  1 root root 0 Mar  6 10:31 cpuset.sched_load_balance
-rw-r--r--  1 root root 0 Mar  6 10:31 cpuset.sched_relax_domain_level
drwxr-xr-x  2 root root 0 Mar  6 10:31 kubepods-besteffort.slice
drwxr-xr-x 10 root root 0 Mar  6 10:31 kubepods-burstable.slice
drwxr-xr-x  4 root root 0 Mar  6 15:09 kubepods-pod4dc3ad18_5bad_4728_9f79_59f2378de46e.slice
-rw-r--r--  1 root root 0 Mar  6 10:31 notify_on_release
-rw-r--r--  1 root root 0 Mar  6 10:31 pool_size
-rw-r--r--  1 root root 0 Mar  6 10:31 tasks
```
其中 kubepods-besteffort.slice 和 kubepods-burstable.slice 分别对应的 besteffort 和 burstable 类型的 pod 配置，因为这两种类型的 pod 并不执行绑核操作，所有的子目录下的 cpuset.cpus 文件均绑定的 cpu 核。<br />kubepods-pod4dc3ad18_5bad_4728_9f79_59f2378de46e.slice 目录为要绑核的 pod 目录，其中 4dc3ad18_5bad_4728_9f79_59f2378de46e 根据 pod 的 uid 转换而来，pod 的 `metadata.uid` 字段的值为 4dc3ad18-5bad-4728-9f79-59f2378de46e，即目录结构中将`-`转换为`_`。<br />目录结构如下：
```
$ ll /sys/fs/cgroup/cpuset/kubepods.slice/kubepods-pod4dc3ad18_5bad_4728_9f79_59f2378de46e.slice
-rw-r--r-- 1 root root 0 Mar  6 15:09 cgroup.clone_children
-rw-r--r-- 1 root root 0 Mar  6 15:09 cgroup.procs
-rw-r--r-- 1 root root 0 Mar  6 15:09 cpuset.cpu_exclusive
-rw-r--r-- 1 root root 0 Mar  6 15:09 cpuset.cpus
-r--r--r-- 1 root root 0 Mar  6 15:09 cpuset.effective_cpus
-r--r--r-- 1 root root 0 Mar  6 15:09 cpuset.effective_mems
-rw-r--r-- 1 root root 0 Mar  6 15:09 cpuset.mem_exclusive
-rw-r--r-- 1 root root 0 Mar  6 15:09 cpuset.mem_hardwall
-rw-r--r-- 1 root root 0 Mar  6 15:09 cpuset.memory_migrate
-r--r--r-- 1 root root 0 Mar  6 15:09 cpuset.memory_pressure
-rw-r--r-- 1 root root 0 Mar  6 15:09 cpuset.memory_spread_page
-rw-r--r-- 1 root root 0 Mar  6 15:09 cpuset.memory_spread_slab
-rw-r--r-- 1 root root 0 Mar  6 15:09 cpuset.mems
-rw-r--r-- 1 root root 0 Mar  6 15:09 cpuset.sched_load_balance
-rw-r--r-- 1 root root 0 Mar  6 15:09 cpuset.sched_relax_domain_level
drwxr-xr-x 2 root root 0 Mar  6 15:09 cri-containerd-75f8e4b4185f673869604d300629ec2de934cf253244bf91c577f9fc0ba0f14a.scope
drwxr-xr-x 2 root root 0 Mar  6 15:09 cri-containerd-cce0a338829921419407fcdc726d1a4bd5d4489da712b49a4834a020131ce718.scope
-rw-r--r-- 1 root root 0 Mar  6 15:09 notify_on_release
-rw-r--r-- 1 root root 0 Mar  6 15:09 pool_size
-rw-r--r-- 1 root root 0 Mar  6 15:09 tasks
```
其中 cri-containerd-75f8e4b4185f673869604d300629ec2de934cf253244bf91c577f9fc0ba0f14a.scope 和 cri-containerd-cce0a338829921419407fcdc726d1a4bd5d4489da712b49a4834a020131ce718.scope 为 pod 的两个容器，其中一个为 pause 容器，另外一个为 nginx 容器。

```
$ crictl pods | grep nginx-deploy
cce0a33882992       6 hours ago         Ready               nginx-deployment-67778646bb-mgcpg                          default             0                   (default)
```
其中第一列的 cce0a33882992 为 pod id，cri-containerd-cce0a338829921419407fcdc726d1a4bd5d4489da712b49a4834a020131ce718.scope 对应的为 pause 容器，查看该目录下的 cpuset.cpus 文件，绑定了所有的核，并未做绑核操作。

```
$ crictl ps | grep nginx-deploy
75f8e4b4185f6       e4720093a3c13       6 hours ago         Running             nginx                      0                   cce0a33882992       nginx-deployment-67778646bb-mgcpg
```
其中第一列的 75f8e4b4185f6 为 container id，cri-containerd-75f8e4b4185f673869604d300629ec2de934cf253244bf91c577f9fc0ba0f14a.scope 对应的为 nginx 容器。查看 /sys/fs/cgroup/cpuset/kubepods.slice/kubepods-pod4dc3ad18_5bad_4728_9f79_59f2378de46e.slice/cri-containerd-75f8e4b4185f673869604d300629ec2de934cf253244bf91c577f9fc0ba0f14a.scope/cpuset.cpus 对应的值为`2-3`，说明绑核成功。

`/var/lib/kubelet/cpu_manager_state` 文件内容如下，跟 cgroup cpuset 实际配置可以完全对应：
```
{"policyName":"static","defaultCpuSet":"0-1","entries":{"4dc3ad18-5bad-4728-9f79-59f2378de46e":{"nginx":"2-3"}},"checksum":3689800814}
```

## kubelet 内部实现
在 kubelet 内部，采用 [cpu manager](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/cm/cpumanager/cpu_manager.go) 模式实现绑核功能。

## 总结

1. 只能 by 节点配置，不能按照 pod 来灵活的配置。
2. 无法适用于所有类型的 pod，pod 必须为 Guaranteed 时才允许开启。
3. 修复配置较为麻烦，一旦配置变更后，需要删除文件 `/var/lib/kubelet/cpu_manager_state` 后才可生效，而且对现有的 pod 均有影响。

# Memory Manager
## kubelet 中参数配置
k8s 1.21 版本引入，需要使用 featuregate 开启，参数 `--feature-gates=MemoryManager=true`。在k8s 1.22 版本为 beta 版本，默认开启。

kubelet 通过参数 `--memory-manager-policy` 来配置内存管理策略，支持如下值：

1. none：默认策略，不执行任何内存分配的策略。
2. static：仅针对 Guaranteed pod 生效，对于 Guaranteed pod，会返回跟 NUMA 相关的 topology hint 信息。在该模式下，会修改 cgroup cpuset.mems 的配置为对应的 cpu core。

kubelet 将已经分配的 pod 的内存绑定信息位于文件 `/var/lib/kubelet/memory_manager_state` 中，文件格式为 json。

# Topology Manager
Topology Manager 特性理解起来比较抽象。举个例子说明：上述的 CPU Manager 和 Memory Manager 的特性，在 kubelet 的实现中是完全独立的，可能会导致 cpu 和内存被分配到了不同的 numa 节点上，CPU Manager 通过 cgroup 的 cpuset.cpus 来控制容器要绑定的 cpu，而 Memory Manager 则通过 cgroup 的 cpuset.mems 来控制容器要使用的 NUMA node 内存。如果两者的信息不匹配，则会导致跨 NUMA Node 的内存访问，从而会对于性能要求高的应用产生影响。<br />Topology Manager 是 kubelet 中的一部分功能，通过 Hint Providers 来发送和接收各个模块的 NUMA 拓扑信息，比如接收 CPU Manager 和 Memory Manager 的 NUMA Node以及 NUMA Node 分配的优先级。
## kubelet 的参数配置
kubelet 通过参数`--topology-manager-scope`来指定作用域：

1. container：默认值，按照容器级别分配到共同的 NUMA node 集合上。
2. pod：将 pod 内的所有 container 分配到共同的 NUMA node 集合上。

kubelet 通过参数 `--topology-manager-policy` 来设置 NUMA 的分配策略：

1. none：默认值，不执行任何的拓扑对齐。
- best-effort：优先选择首选亲和性的 NUMA node，如果亲和性不满足，pod 仍然可以调度成功。
- restricted：选择首选亲和性的 NUMA node，如果亲和性不满足，pod 调度失败。
- single-numa-node：通过 Hint Provider 返回的结果，判断单 NUMA 节点的亲和性是否，如果不满足，则 pod 调度失败。
## kubelet 中的实现

在 kubelet 中定义了接口：
```go
// TopologyHint is a struct containing the NUMANodeAffinity for a Container
type TopologyHint struct {
    // 记录了 NUMA Node 满足资源请求的位掩码
	NUMANodeAffinity bitmask.BitMask
	// Preferred is set to true when the NUMANodeAffinity encodes a preferred
	// allocation for the Container. It is set to false otherwise.
    // 亲和性的结果是否为首选的
	Preferred bool
}

// HintProvider is an interface for components that want to collaborate to
// achieve globally optimal concrete resource alignment with respect to
// NUMA locality.
type HintProvider interface {
    // GetTopologyHints returns a map of resource names to a list of possible
    // concrete resource allocations in terms of NUMA locality hints. Each hint
    // is optionally marked "preferred" and indicates the set of NUMA nodes
    // involved in the hypothetical allocation. The topology manager calls
    // this function for each hint provider, and merges the hints to produce
    // a consensus "best" hint. The hint providers may subsequently query the
    // topology manager to influence actual resource assignment.
    GetTopologyHints(pod *v1.Pod, container *v1.Container) map[string][]TopologyHint
    // GetPodTopologyHints returns a map of resource names to a list of possible
    // concrete resource allocations per Pod in terms of NUMA locality hints.
    GetPodTopologyHints(pod *v1.Pod) map[string][]TopologyHint
    // Allocate triggers resource allocation to occur on the HintProvider after
    // all hints have been gathered and the aggregated Hint is available via a
    // call to Store.GetAffinity().
    Allocate(pod *v1.Pod, container *v1.Container) error
}
```
CPU Manager、Memory Manager 和 Device Manager 均实现了该接口。在 Topology Manager 中根据各个 Manager 返回的 TopologyHint 数据，从而决定最终的 NUMA Node 分配，并调用各个 Manager 的 Allocate 来做最终的 NUMA Node 分配。

# 资料

- [ACK CPU拓扑感知调度](https://www.alibabacloud.com/help/zh/ack/ack-managed-and-ack-dedicated/user-guide/topology-aware-cpu-scheduling)
- [控制节点上的 CPU 管理策略](https://v1-25.docs.kubernetes.io/zh-cn/docs/tasks/administer-cluster/cpu-management-policies/)
- [配置 Pod 的服务质量](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/quality-service-pod/)
- [Kubelet之Topology Manager分析](https://developer.aliyun.com/article/784148)