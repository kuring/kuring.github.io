title: k8s多集群管理方案 - karmada
date: 2022-02-22 20:16:25
tags: 多集群

author:
---
# 简介
华为云开源的多云容器编排项目，目前为CNCF沙箱项目。Karmada是基于[kubefed v2](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fkubernetes-sigs%2Fkubefed)进行修改的一个项目，因此里面很多概念都是取自kubefed v2。

Karmada相对于kubefed v2的最大优点：**完全兼容k8s的API**。karmada提供了一个一套独立的karmada-apiserver，跟kube-apiserver是兼容的，使用方在调用的时候只需要访问karmada-apiserver就可以了。

# 架构
![](https://kuring.oss-cn-beijing.aliyuncs.com/common/karmada-arch.png)
有三个组件构建：

- karmada api server，用来给其他的组件提供rest api
- karmada controller manager
- karmada scheduler

karmada会新建一个ETCD，用来存储karmada的API对象。

karmada controller manager内部包含了多个controller的功能，会通过karmada apiserver来watch karmada对象，并通过调用各个子集群的apiserver来创建标准的k8s对象，包含了如下的controller对象：

1. Cluster Controller：用来管理子集群的声明周期
1. Policy Controller：监听PropagationPolicy对象，通过resourceSelector找到匹配中的资源，并创建ResourceBinding对象。
1. Binding Controller：监听ResourceBinding对象，并创建每个集群的Work对象。
1. Execution Controller：监听Work对象，一旦Work对象场景后，会在子集群中创建Work关联的k8s对象。

# 组件
## karmada-aggregated-apiserver
在karmada-apiserver上注册的api信息
## etcd
用来存放karmada的元数据信息
# CRD
## Cluster
需要使用kamada-apiserver来查询

<details>
<summary>unfold me to see the yaml</summary>

```yaml
apiVersion: cluster.karmada.io/v1alpha1
kind: Cluster
metadata:
  finalizers:
  - karmada.io/cluster-controller
  name: member1
spec:
  apiEndpoint: https://172.18.0.3:6443
  impersonatorSecretRef:
    name: member1-impersonator
    namespace: karmada-cluster
  secretRef:
    name: member1
    namespace: karmada-cluster
  syncMode: Push
status:
	# 支持的api列表
  apiEnablements:
  - groupVersion: admissionregistration.k8s.io/v1
    resources:
    - kind: MutatingWebhookConfiguration
      name: mutatingwebhookconfigurations
    - kind: ValidatingWebhookConfiguration
      name: validatingwebhookconfigurations
  - groupVersion: apiextensions.k8s.io/v1
    resources:
    - kind: CustomResourceDefinition
      name: customresourcedefinitions
  - groupVersion: apiregistration.k8s.io/v1
    resources:
    - kind: APIService
      name: apiservices
  - groupVersion: apps/v1
    resources:
    - kind: ControllerRevision
      name: controllerrevisions
    - kind: DaemonSet
      name: daemonsets
    - kind: Deployment
      name: deployments
    - kind: ReplicaSet
      name: replicasets
    - kind: StatefulSet
      name: statefulsets
  - groupVersion: authentication.k8s.io/v1
    resources:
    - kind: TokenReview
      name: tokenreviews
  - groupVersion: authorization.k8s.io/v1
    resources:
    - kind: LocalSubjectAccessReview
      name: localsubjectaccessreviews
    - kind: SelfSubjectAccessReview
      name: selfsubjectaccessreviews
    - kind: SelfSubjectRulesReview
      name: selfsubjectrulesreviews
    - kind: SubjectAccessReview
      name: subjectaccessreviews
  - groupVersion: autoscaling/v1
    resources:
    - kind: HorizontalPodAutoscaler
      name: horizontalpodautoscalers
  - groupVersion: autoscaling/v2beta1
    resources:
    - kind: HorizontalPodAutoscaler
      name: horizontalpodautoscalers
  - groupVersion: autoscaling/v2beta2
    resources:
    - kind: HorizontalPodAutoscaler
      name: horizontalpodautoscalers
  - groupVersion: batch/v1
    resources:
    - kind: CronJob
      name: cronjobs
    - kind: Job
      name: jobs
  - groupVersion: batch/v1beta1
    resources:
    - kind: CronJob
      name: cronjobs
  - groupVersion: certificates.k8s.io/v1
    resources:
    - kind: CertificateSigningRequest
      name: certificatesigningrequests
  - groupVersion: coordination.k8s.io/v1
    resources:
    - kind: Lease
      name: leases
  - groupVersion: discovery.k8s.io/v1
    resources:
    - kind: EndpointSlice
      name: endpointslices
  - groupVersion: discovery.k8s.io/v1beta1
    resources:
    - kind: EndpointSlice
      name: endpointslices
  - groupVersion: events.k8s.io/v1
    resources:
    - kind: Event
      name: events
  - groupVersion: events.k8s.io/v1beta1
    resources:
    - kind: Event
      name: events
  - groupVersion: flowcontrol.apiserver.k8s.io/v1beta1
    resources:
    - kind: FlowSchema
      name: flowschemas
    - kind: PriorityLevelConfiguration
      name: prioritylevelconfigurations
  - groupVersion: networking.k8s.io/v1
    resources:
    - kind: IngressClass
      name: ingressclasses
    - kind: Ingress
      name: ingresses
    - kind: NetworkPolicy
      name: networkpolicies
  - groupVersion: node.k8s.io/v1
    resources:
    - kind: RuntimeClass
      name: runtimeclasses
  - groupVersion: node.k8s.io/v1beta1
    resources:
    - kind: RuntimeClass
      name: runtimeclasses
  - groupVersion: policy/v1
    resources:
    - kind: PodDisruptionBudget
      name: poddisruptionbudgets
  - groupVersion: policy/v1beta1
    resources:
    - kind: PodDisruptionBudget
      name: poddisruptionbudgets
    - kind: PodSecurityPolicy
      name: podsecuritypolicies
  - groupVersion: rbac.authorization.k8s.io/v1
    resources:
    - kind: ClusterRoleBinding
      name: clusterrolebindings
    - kind: ClusterRole
      name: clusterroles
    - kind: RoleBinding
      name: rolebindings
    - kind: Role
      name: roles
  - groupVersion: scheduling.k8s.io/v1
    resources:
    - kind: PriorityClass
      name: priorityclasses
  - groupVersion: storage.k8s.io/v1
    resources:
    - kind: CSIDriver
      name: csidrivers
    - kind: CSINode
      name: csinodes
    - kind: StorageClass
      name: storageclasses
    - kind: VolumeAttachment
      name: volumeattachments
  - groupVersion: storage.k8s.io/v1beta1
    resources:
    - kind: CSIStorageCapacity
      name: csistoragecapacities
  - groupVersion: v1
    resources:
    - kind: Binding
      name: bindings
    - kind: ComponentStatus
      name: componentstatuses
    - kind: ConfigMap
      name: configmaps
    - kind: Endpoints
      name: endpoints
    - kind: Event
      name: events
    - kind: LimitRange
      name: limitranges
    - kind: Namespace
      name: namespaces
    - kind: Node
      name: nodes
    - kind: PersistentVolumeClaim
      name: persistentvolumeclaims
    - kind: PersistentVolume
      name: persistentvolumes
    - kind: Pod
      name: pods
    - kind: PodTemplate
      name: podtemplates
    - kind: ReplicationController
      name: replicationcontrollers
    - kind: ResourceQuota
      name: resourcequotas
    - kind: Secret
      name: secrets
    - kind: ServiceAccount
      name: serviceaccounts
    - kind: Service
      name: services
  conditions:
  - lastTransitionTime: "2022-02-21T08:27:56Z"
    message: cluster is healthy and ready to accept workloads
    reason: ClusterReady
    status: "True"
    type: Ready
  kubernetesVersion: v1.22.0
  nodeSummary:
    readyNum: 1
    totalNum: 1
  resourceSummary:
    allocatable:
      cpu: "8"
      ephemeral-storage: 41152812Ki
      hugepages-1Gi: "0"
      hugepages-2Mi: "0"
      memory: 32192720Ki
      pods: "110"
    allocated:
      cpu: 950m
      memory: 290Mi
      pods: "9"
```
</details>

## PropagationPolicy
应用发布策略
```powershell
apiVersion: policy.karmada.io/v1alpha1
kind: PropagationPolicy
metadata:
  name: nginx-propagation
  namespace: default
spec:
  placement:
    clusterAffinity:
      clusterNames:
      - member1
      - member2
    replicaScheduling:
      replicaDivisionPreference: Weighted·
      replicaSchedulingType: Divided
      weightPreference:
        staticWeightList:
        - targetCluster:
            clusterNames:
            - member1
          weight: 1
        - targetCluster:
            clusterNames:
            - member2
          weight: 1
  resourceSelectors:
  - apiVersion: apps/v1
    kind: Deployment
    name: nginx
    namespace: default
```
## ClusterPropagationPolicy
用来定义Cluster级别资源的发布策略
```yaml
apiVersion: policy.karmada.io/v1alpha1
kind: ClusterPropagationPolicy
metadata:
  name: serviceexport-policy
spec:
  resourceSelectors:
    - apiVersion: apiextensions.k8s.io/v1
      kind: CustomResourceDefinition
      name: serviceexports.multicluster.x-k8s.io
  placement:
    clusterAffinity:
      clusterNames:
        - member1
        - member2
```
## OverridePolicy
用于修改不同集群内的对象
```yaml
apiVersion: policy.karmada.io/v1alpha1
kind: OverridePolicy
metadata:
  name: example
spec:
  ...
  overriders:
    commandOverrider:
      - containerName: myapp
        operator: remove
        value:
          - --parameter1=foo
```

# 安装
karmada提供了三种安装方式：

- kubectl karmada插件的方式安装，较新的特性，v1.0版本才会提供，当前最新版本为v0.10.1
- helm chart方式安装
- 使用源码安装
## 安装kubectl插件
karmada提供了一个cli的工具，可以是单独的二进制工具kubectl-karmada，也可以是kubectl的插件karmada，本文以kubectl插件的方式进行安装。
```powershell
kubectl krew install karmada
```
接下来就可以执行 kubectl karmada命令了。
## helm chart的方式安装
使用kind插件一个k8s集群 host，此处步骤略
​

在源码目录下执行如下的命令
```powershell
helm install karmada -n karmada-system --create-namespace ./charts
```
会在karmada-system下部署如下管控的组件
```powershell
$ kubectl get deployments -n karmada-system
NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
karmada-apiserver                 1/1     1            1           83s
karmada-controller-manager        1/1     1            1           83s
karmada-kube-controller-manager   1/1     1            1           83s
karmada-scheduler                 1/1     1            1           83s
karmada-webhook                   1/1     1            1           83s

$ kubectl get statefulsets -n karmada-system
NAME   READY   AGE
etcd   1/1     2m1s
```

## 从源码安装一个本地测试集群
本方式仅用于本地测试，会自动使用kind拉起测试的k8s集群，包括了一个host集群和3个member集群。


执行如下命令，会执行如下的任务：

1. 使用kind启动一个新的k8s集群host
1. 构建karmada的控制平面，并部署控制平面到host集群
1. 创建3个member集群并加入到karmada中
```powershell
git clone https://github.com/karmada-io/karmada
cd karmada
hack/local-up-karmada.sh
```
local-up-karmada.sh有两个细节问题值得关注。
kind创建出来的集群名一定带有“kind-”的前缀，该脚本中为了去掉“kind-”前缀，使用了kubectl config rename-context 命令来rename context。[https://github.com/karmada-io/karmada/blob/master/hack/util.sh#L375](https://github.com/karmada-io/karmada/blob/master/hack/util.sh#L375)
另外，使用kind创建出来的集群，context中的apiserver地址为类似[https://127.0.0.1:45195](https://127.0.0.1:45195)这样的本地随机端口，只能在宿主机网络中访问。如果要想在两个k8s集群之间访问是不可行的。脚本中使用kubectl config set-cluster 命令将集群apiserver的地址替换为了docker网段的ip地址，以便于多个k8s集群之间的互访。[https://github.com/karmada-io/karmada/blob/master/hack/util.sh#L389](https://github.com/karmada-io/karmada/blob/master/hack/util.sh#L389)
​

可以看到创建了如下的4个k8s cluster

```powershell
$ kind get clusters
karmada-host
member1
member2
member3
```

但这四个集群会分为两个kubeconfig文件 ~/.kube/karmada.config 和 ~/.kube/members.config。karmada又分为了两个context karmada-apiserver和karmada-host，两者连接同一个k8s集群。karmada-apiserver为跟karmada控制平台交互使用的context，为容器中的karmada-apiserver。karmada-host连接容器的kube-apiserver。
​

如果要连接host集群设置环境变量：export KUBECONFIG="$HOME/.kube/karmada.config"
如果要连接member集群设置环境变量：export KUBECONFIG="$HOME/.kube/members.config"
```yaml
$ export KUBECONFIG="$HOME/.kube/karmada.config"
# 切换到karmada-host
$ kctx karmada-host
# 可以看到karmada部署的组件
$ k get deploy -n karmada-system   
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
karmada-aggregated-apiserver          2/2     2            2           17m
karmada-apiserver                     1/1     1            1           18m
karmada-controller-manager            2/2     2            2           17m
karmada-kube-controller-manager       1/1     1            1           17m
karmada-scheduler                     2/2     2            2           17m
karmada-scheduler-estimator-member1   2/2     2            2           17m
karmada-scheduler-estimator-member2   2/2     2            2           17m
karmada-scheduler-estimator-member3   2/2     2            2           17m
karmada-webhook                       2/2     2            2           17m
```
在host集群中可以看到会创建出如下的pod
```powershell
# k get pod -n karmada-system 
NAME                                                   READY   STATUS    RESTARTS   AGE
etcd-0                                                 1/1     Running   0          18h
karmada-aggregated-apiserver-7b88b8df99-95twq          1/1     Running   0          18h
karmada-apiserver-5746cf97bb-pspfg                     1/1     Running   0          18h
karmada-controller-manager-7d66968445-h4xsc            1/1     Running   0          18h
karmada-kube-controller-manager-869d9df85-f4bqj        1/1     Running   0          18h
karmada-scheduler-8677cdf96d-psnlw                     1/1     Running   0          18h
karmada-scheduler-estimator-member1-696b54fd56-jjg6b   1/1     Running   0          18h
karmada-scheduler-estimator-member2-774fb84c5d-fldhd   1/1     Running   0          18h
karmada-scheduler-estimator-member3-5c7d87f4b4-fk4lj   1/1     Running   0          18h
karmada-webhook-79b87f7c5f-lt8ps                       1/1     Running   0          18h
```
在karmada-apiserver集群会创建出如下的Cluster，同时可以看到有两个Push模式一个Pull模式的集群。
```yaml
$ kctx karmada-apiserver
Switched to context "karmada-apiserver".
$ kubectl get clusters
NAME      VERSION   MODE   READY   AGE
member1   v1.22.0   Push   True    61m
member2   v1.22.0   Push   True    61m
member3   v1.22.0   Pull   True    61m
```
# 应用发布
![](https://kuring.oss-cn-beijing.aliyuncs.com/common/karama-concepts.png)

将context切换到karmada-host，在host集群部署应用nginx
```powershell
$ export KUBECONFIG="$HOME/.kube/karmada.config"
$ kctx karmada-apiserver
$ kubectl create -f samples/nginx/propagationpolicy.yaml
$ cat samples/nginx/propagationpolicy.yaml
apiVersion: policy.karmada.io/v1alpha1
kind: PropagationPolicy
metadata:
  name: nginx-propagation
spec:
  resourceSelectors:
    - apiVersion: apps/v1
      kind: Deployment
      name: nginx
  placement:
    # 要提交到子集群1和2
    clusterAffinity:
      clusterNames:
        - member1
        - member2
    replicaScheduling:
      replicaDivisionPreference: Weighted
      replicaSchedulingType: Divided
      weightPreference:
        staticWeightList:
          - targetCluster:
              clusterNames:
                - member1
            weight: 1
          - targetCluster:
              clusterNames:
                - member2
            weight: 1
```
在karmada-apiserver中提交deployment。deployment提交后实际上仅为一个模板，只有PropagationPolicy跟deployment关联后，才会真正部署。deployment中指定的副本数为所有子集群的副本数综合。
```yaml
$ kubectl create -f samples/nginx/deployment.yaml
$ k get deploy 
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   2/2     2            2           7m37s
# 可以看到虽然deployment的副本数为2，但在karmada-apiserver集群中实际上并没有pod创建出来
$ k get pod
No resources found in default namespace.
```
通过karmadactl命令可以查询环境中的所有子集群的pod状态
```yaml
$ karmadactl get pod
The karmadactl get command now only supports Push mode, [ member3 ] are not push mode

NAME                     CLUSTER   READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-4w2gc   member1   1/1     Running   0          4m16s
nginx-6799fc88d8-j77f5   member2   1/1     Running   0          4m16s
```
# 元集群访问子集群
![](https://kuring.oss-cn-beijing.aliyuncs.com/common/karmada-aa.png)

可以看到在karmada-apiserver上注册了AA服务，group为cluster.karmada.io
```yaml
# kubectl --kubeconfig /root/.kube/karmada.config --context karmada-apiserver get  apiservice v1alpha1.cluster.karmada.io -o yaml 
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    apiserver: "true"
    app: karmada-aggregated-apiserver
  name: v1alpha1.cluster.karmada.io
spec:
  group: cluster.karmada.io
  groupPriorityMinimum: 2000
  insecureSkipTLSVerify: true
  service:
    name: karmada-aggregated-apiserver
    namespace: karmada-system
    port: 443
  version: v1alpha1
  versionPriority: 10
status:
  conditions:
  - lastTransitionTime: "2022-02-21T08:27:47Z"
    message: all checks passed
    reason: Passed
    status: "True"
    type: Available
```
如果直接使用kubectl访问karmada-apiserver服务，会提示存在权限问题
```yaml
￥ kubectl --kubeconfig ~/.kube/karmada-apiserver.config --context karmada-apiserver   get --raw /apis/cluster.karmada.io/v1alpha1/clusters/member1/proxy/api/v1/nodes
Error from server (Forbidden): users "system:admin" is forbidden: User "system:serviceaccount:karmada-cluster:karmada-impersonator" cannot impersonate resource "users" in API group "" at the cluster scope
```
给karmada-apiserver的Account system:admin授权，创建文件cluster-proxy-rbac.yaml，内容如下：
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-proxy-clusterrole
rules:
- apiGroups:
  - 'cluster.karmada.io'
  resources:
  - clusters/proxy
  resourceNames:
  - member1
  - member2
  - member3
  verbs:
  - '*'
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-proxy-clusterrolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-proxy-clusterrole
subjects:
  - kind: User
    name: "system:admin"
```
执行如下命令即可给karmada-apiserver的Account system:admin 授权可访问AA服务的member1-3
```yaml
kubectl --kubeconfig /root/.kube/karmada.config --context karmada-apiserver apply -f cluster-proxy-rbac.yaml   
```
通过url的形式访问AA服务，返回数据格式为json
```yaml
kubectl --kubeconfig ~/.kube/karmada-apiserver.config --context karmada-apiserver get --raw /apis/cluster.karmada.io/v1alpha1/clusters/member1/proxy/api/v1/nodes
```
如果要想使用 kubectl get node 的形式来访问，则需要在kubeconfig文件中server字段的url地址后追加url `/apis/cluster.karmada.io/v1alpha1/clusters/{clustername}/proxy` 
## 给特定的账号授权
在karmada-apiserver中创建账号 tom
```yaml
kubectl --kubeconfig /root/.kube/karmada.config --context karmada-apiserver create serviceaccount tom
```
在karmada-apiserver中提交如下的yaml文件，用来给tom账号增加访问member1集群的权限
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-proxy-clusterrole
rules:
- apiGroups:
  - 'cluster.karmada.io'
  resources:
  - clusters/proxy
  resourceNames:
  - member1
  verbs:
  - '*'
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-proxy-clusterrolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-proxy-clusterrole
subjects:
  - kind: ServiceAccount
    name: tom
    namespace: default
  # The token generated by the serviceaccount can parse the group information. Therefore, you need to specify the group information below.
  - kind: Group
    name: "system:serviceaccounts"
  - kind: Group
    name: "system:serviceaccounts:default"
```
执行如下命令提交
```yaml
kubectl --kubeconfig /root/.kube/karmada.config --context karmada-apiserver apply -f cluster-proxy-rbac.yaml
```
获取karmada-apiserver集群的tom账号的token信息
```yaml
kubectl --kubeconfig /root/.kube/karmada.config --context karmada-apiserver  get secret `kubectl --kubeconfig /root/.kube/karmada.config --context karmada-apiserver  get sa tom -oyaml | grep token | awk '{print $3}'` -oyaml | grep token: | awk '{print $2}' | base64 -d
```
 创建~/.kube/tom.config文件，其中token信息为上一步获取的token，server的地址可以查看 ~/.kube/karmada-apiserver.config文件
```yaml
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: {karmada-apiserver-address} # Replace {karmada-apiserver-address} with karmada-apiserver-address. You can find it in /root/.kube/karmada.config file.
  name: tom
contexts:
- context:
    cluster: tom
    user: tom
  name: tom
current-context: tom
kind: Config
users:
- name: tom
  user:
    token: {token} # Replace {token} with the token obtain above.
```
通过karmada-apiserver的tom用户访问member1集群
```yaml
# 预期可以正常访问
$ kubectl --kubeconfig ~/.kube/tom.config get --raw /apis/cluster.karmada.io/v1alpha1/clusters/member1/proxy/apis

# 预期不可以访问，因为tom在member1集群没有权限
$ kubectl --kubeconfig ~/.kube/tom.config get --raw /apis/cluster.karmada.io/v1alpha1/clusters/member1/proxy/api/v1/nodes
Error from server (Forbidden): nodes is forbidden: User "system:serviceaccount:default:tom" cannot list resource "nodes" in API group "" at the cluster scope
```
在member1集群将tom账号绑定权限，创建member1-rbac.yaml文件，内容如下：
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: tom
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tom
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tom
subjects:
  - kind: ServiceAccount
    name: tom
    namespace: default
```
权限在member1集群生效
```yaml
kubectl --kubeconfig /root/.kube/members.config --context member1 apply -f member1-rbac.yaml 
```
重新执行命令即可以访问子集群中的数据
```yaml
kubectl --kubeconfig ~/.kube/tom.config get --raw /apis/cluster.karmada.io/v1alpha1/clusters/member1/proxy/api/v1/nodes
```
# 集群注册
支持Push和Pull两种模式。
​

push模式karmada会直接访问成员集群的kuba-apiserver。

pull模式针对的场景是中心集群无法直接子集群的场景。每个子集群运行karmada-agent组件，一旦karmada-agent部署完成后就会自动向host集群注册，karmada-agent会watch host集群的karmada-es-<cluster name>下的cr，并在本集群部署。
​

