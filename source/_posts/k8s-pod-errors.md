title: k8s pod异常状态分析
date: 2022-03-17 15:54:47
tags:
author:
---
# pod状态

## 1. 前置检查
在排查异常状态的pod错误之前，可以先检查一下node状态，执行`kubectl get node`查看是否所有的node状态都正常。

## 2. pod状态为CrashLoopBackOff
如果pod状态为CrashLoopBackOff状态，查看pod日志来定位原因，或者describe pod看一下。
## 3. pod状态为Pending
pod状态为Pending状态，说明调度失败，通常跟污点、标签、cpu、内存、磁盘等资源相关。

可以通过`kubectl describe pod -n {NAMESPACE} {POD_NAME}`，可以在最后的Event部分找到原因。
## 4. pod状态为Init:0/1
有些pod会有Init Containers，这些container是在pod的containers执行之前先执行。如果Init Container出现未执行完成的情况，此时pod处于Init状态。

通过`kubectl get pod -n {NAMESPACE} {POD_NAME} -o yaml` 找到pod的Init Containers，并找到其中的name字段。执行`kubectl logs -n {NAMESPACE} {POD_NAME} -c {INIT_CONTAINER_NAME}`可以查看Init Container的日志来分析原因。
## 5. pod状态为Terminating
pod处于此种状态的原因大致可分为：
1、pod或其控制器被删除。
解决方法：查看pod控制器类型和控制器名称，查看其控制器是否正常。如果正常pod将会被重建，如果pod没有被重建，查看controller-manager是否正常。
2、pod所在节点状态NotReady导致。
解决方法：检查该节点的网络，cpu，内存，磁盘空间等资源是否正常。检查该节点的kubelet、docker服务是否正常。检查该节点的网络插件pod是否正常。

最常见的pod处于Terminating状态的解决办法为强制删除 `kubectl delete pods -n ${namespace} ${name} --grace-period=0 --force`
## 6. pod状态为Evicted
pod处于Evicted的原因大致可分为：
1、kubelet服务启动时存在驱逐限制当节点资源可用量达到指定阈值（magefs.available<15%,memory.available<300Mi,nodefs.available<10%,nodefs.inodesFree<5%）
会优先驱逐Qos级别低的pod以保障Qos级别高的pod可用。
解决方法：增加节点资源或将被驱逐的pod迁移到其他空闲资源充足的节点上。
2、pod所在节点上被打上了NoExecute的污点，此种污点会将该节点上无法容忍此污点的pod进行驱逐。
解决方法：查看该节点上的NoExecute污点是否必要。或者pod是否可以迁移到其他节点。
3、pod所在的节点为NotReady状态

通常可以通过`kubectl describe pods ${pod} -n ${namespace}`的底部的Events信息来找到一些问题的原因。例如下面例子中可以看到DiskPressure信息，说明跟磁盘相关。
```yaml
Events:
  Type     Reason     Age        From                 Message
  ----     ------     ----       ----                 -------
  Warning  Evicted    61s        kubelet, acs.deploy  The node had condition: [DiskPressure].
  Normal   Scheduled  <invalid>  default-scheduler    Successfully assigned ark-system/bridge-console-bridge-console-554d57bb87-nh2vd to acs.deploy
```
或者根据pod的Message字段来找到原因
```
Name:           tiller-deploy-7f6456894f-22vgr
Namespace:      kube-system
Priority:       0
Node:           a36e04001.cloud.e04.amtest17/
Start Time:     Mon, 08 Jun 2020 12:17:26 +0800
Labels:         app=helm
                name=tiller
                pod-template-hash=7f6456894f
Annotations:    <none>
Status:         Failed
Reason:         Evicted
Message:        Pod The node had condition: [DiskPressure].
```
由于pod驱逐的原因排查跟时间点相关，需要根据pod被驱逐的时间来分析当时的状态。
4、批量删除状态Evicted的pod，此操作会删除集群里面所有状态为Evicted的pod
```shell
ns=`kubectl get ns | awk 'NR>1 {print $1}'`
for i in $ns;do  kubectl get pods -n $i  | grep Evicted| awk '{print $1}' | xargs  kubectl delete pods -n $i ;done
```
## 7. pod状态为Unknown
通常该状态为pod对应节点的为NotReady，通过查看 kubectl get node 来查看node是否为NotReady。
## 8. pod为running，但是Not Ready状态
```yaml
argo-ui-56f4d67b69-8gshr   0/1     Running     0          10h
```
类似上面这种状态，此时说明pod的readiness健康检查没过导致的，需要先从pod的健康检查本身来排查问题。可以通过 `kubectl get pods -n ${namespace} ${name} -o yaml` 找到pod的健康检查部分，关键字为readiness，然后进入pod中执行对应的健康检查命令来测试健康检查的准确性。

例如readiness的配置如下，需要进入pod中执行`curl http://127.0.0.1/api/status`，也可以在pod对应的node节点上执行`curl [http://${pod_ip}/api/status](http://127.0.0.1/api/status)`
```yaml
    livenessProbe:
      failureThreshold: 3
      httpGet:
        path: /api/status
        port: http
        scheme: HTTP
      initialDelaySeconds: 120
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
```
## 9. pod为ContainerCreating状态
通过`kubectl describe pods -n ${namespace} ${name}`的Events部分来分析原因，可能的原因如下：

- 网络分配ip地址失败

## 10. Init:CrashLoopBackOff

1. 通过`kubectl get pod -n {NAMESPACE} {POD_NAME} -o yaml` 找到pod的Init Containers，并找到其中的name字段。
2. 执行`kubectl logs -n {NAMESPACE} {POD_NAME} -c {INIT_CONTAINER_NAME}`可以查看Init Container的日志来分析原因。

## 11. PodInitializing
需要查看initContainer的日志

## 12. MatchNodeSelector

查看pod的status信息可以看到如下信息：
```
status:
  message: Pod Predicate MatchNodeSelector failed
  phase: Failed
  reason: MatchNodeSelector
  startTime: "2022-03-15T05:07:57Z"
```

说明该pod没有调度成功，在predicate的MatchNodeSelector阶段失败了，没有匹配上node节点。

在k8s 1.21之前的版本，存在bug，节点重启后可能遇到过问题，将pod delete后重新调度可以解决。https://github.com/kubernetes/kubernetes/issues/92067

# 参考
- [常见的Pod异常状态及处理方式](https://help.aliyun.com/document_detail/412618.html#section-7gv-3tf-paf)