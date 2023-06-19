title: 根据pid获取到pod名称
date: 2022-05-06 14:45:20
tags:
author:
---
在k8s的运维过程中，经常会有根据pid获取到pod名称的需求。比如某个pid占用cpu特别高，想知道是哪个pod里面的进程。

# 操作步骤

查看进程的cgroup信息

```
$ cat /proc/5760/cgroup 
11:rdma:/
10:hugetlb:/apsara_k8s/kubepods/burstable/podf87c35b4-c170-4e1c-a726-d839e2fe6bea/17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3
9:freezer:/apsara_k8s/kubepods/burstable/podf87c35b4-c170-4e1c-a726-d839e2fe6bea/17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3
8:pids:/apsara_k8s/kubepods/burstable/podf87c35b4-c170-4e1c-a726-d839e2fe6bea/17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3
7:perf_event:/apsara_k8s/kubepods/burstable/podf87c35b4-c170-4e1c-a726-d839e2fe6bea/17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3
6:blkio:/apsara_k8s/kubepods/burstable/podf87c35b4-c170-4e1c-a726-d839e2fe6bea/17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3
5:devices:/apsara_k8s/kubepods/burstable/podf87c35b4-c170-4e1c-a726-d839e2fe6bea/17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3
4:memory:/apsara_k8s/kubepods/burstable/podf87c35b4-c170-4e1c-a726-d839e2fe6bea/17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3
3:cpuset,cpu,cpuacct:/apsara_k8s/kubepods/burstable/podf87c35b4-c170-4e1c-a726-d839e2fe6bea/17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3
2:net_cls,net_prio:/apsara_k8s/kubepods/burstable/podf87c35b4-c170-4e1c-a726-d839e2fe6bea/17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3
1:name=systemd:/apsara_k8s/kubepods/burstable/podf87c35b4-c170-4e1c-a726-d839e2fe6bea/17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3

```

其中的 `17e15f78a43b2ddc7caff93bcc03d5ca7f5249fbee4c2ade5c3c96a7279af0a3` 即为容器的id，可以如下的命令直接获取到容器的pid

```
CID=$(cat /proc/5760/cgroup | awk -F '/' '{print $6}')
echo ${CID:0:8}
```

继续执行如下命令查看docker的label

```
docker  inspect 17e15f --format "{{json .Config.Labels}}" 
```

其中io.kubernetes.pod.namespace为pod所在的namespace，io.kubernetes.pod.name为pod的name。

# 参考资料
- [Kubernetes 教程：根据 PID 获取 Pod 名称](https://icloudnative.io/posts/find-kubernetes-pod-info-from-process-id/)

