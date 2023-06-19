title: k8s多集群管理方案 - clusternet
date: 2021-12-26 21:44:17
tags: 多集群
---
# 简介

腾讯云开源的k8s多集群管理方案，可发布应用到多个k8s集群。
[https://github.com/clusternet/clusternet](https://github.com/clusternet/clusternet)
GitHub Star：891

# 架构
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/clusternet1.png)
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/clusternet2.png)
包含clusternet-hub和clusternet-agent两个组件。

clusternet-hub部署在父集群，用途：

- 接收子集群的注册请求，并为每个自己群创建namespace、serviceaccount等资源
- 提供父集群跟各个子集群的长连接
- 提供restful api来访问各个子集群，同时支持子集群的互访

clusternet-agent部署在子集群，用途：

- 自动注册子集群到父集群
- 心跳报告到中心集群，包括k8s版本、运行平台、健康状态等
- 通过websocket协议跟父集群通讯

# CRD抽象

## ClusterRegistrationRequest
clusternet-agent在父集群中创建的CR，一个k8s集群一个
```powershell
apiVersion: clusters.clusternet.io/v1beta1
kind: ClusterRegistrationRequest
metadata:
  labels:
    clusters.clusternet.io/cluster-id: 2bcd48d2-0ead-45f7-938b-5af6af7da5fd
    clusters.clusternet.io/cluster-name: clusternet-cluster-pmlbp
    clusters.clusternet.io/registered-by: clusternet-agent
  name: clusternet-2bcd48d2-0ead-45f7-938b-5af6af7da5fd
spec:
  clusterId: 2bcd48d2-0ead-45f7-938b-5af6af7da5fd
  clusterName: clusternet-cluster-pmlbp # 集群名称
  clusterType: EdgeCluster # 集群类型
  syncMode: Dual
status:
  caCertificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1USXhNekV3TWpjeE5Wb1hEVE14TVRJeE1URXdNamN4TlZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTHVCCmJwdXNaZXM5RXFXaEw1WUVGb2c4eHRJai9jbFh2S1lTMnZYbGR4cUt1MUR3VHV3aUcxZUN4VGlHa2dLQXVXd1IKVFFheEh5aGY3U29lc0hqMG43NnFBKzhQT05lS1VGdERJOWVzejF2WTF5bXFoUHR0QVo0cGhkWmhmbXJjZTJLRQpuMS84MzNWbWlXd0pSZmNWcEJtTU52MjFYMVVwNWp6RGtncS9tY0JOOGN0ZU5PMEpKNkVVeTE2RXZLbjhyWG90ClErTW5PUHE4anFNMzJjRFFqYWVEdGxvM2kveXlRd1NMa2wzNFo4aElwZDZNWWVSTWpXcmhrazF4L0RYZjNJK3IKOGs5S1FBbEsvNWZRMXk5NHYwT25TK0FKZTliczZMT2srYVFWYm5SbExab2E2aVZWbUJNam03UjBjQ2w0Y1hpRwpyekRnN1ZLc3lsY1o3aGFRRTNFQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZPaGwxMHUwNnJvenZJUm9XVmpNYlVPMzFDbERNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBQytqMXpPQVZYVXpNUFJ0U29Kd3JhMU1FSkJOUTNMWitmWnZRYjdIbk53b00zaCthMgoyc25yUitSWTYrOFFDbXVKeis1eU5yYStEZDlnNzQ1Q0hDaFpVZzlsY3RjQTRzZVR5OXZqcUVQNVBNSzEvbi9PCnFEcDQyMUpqTjYvUXJmamIvbVM0elUvZXNGZGowQXRYQ0FLMWJsNmF0ai9jZXVBbzh1bTRPaUVlNnJhanBYTHUKWFlmQ1FVZFV5TWpQdEZHTDMwNytna0RpdlVsdXk3RkQ4aUpURTh2QWpxOVBXOW40SmxFMjdQWXR5QnNocy9XSApCZ2czQjZpTG10SjZlNzJiWnA3ZmptdmJWTC9VdkxzYXZqRXltZDhrMnN1bFFwQUpzeVJrMkEzM1g3NWJpS0RGCk1CU29DaHc3U2JMSkJrN0FNckRzTjd1Q2U3WWVIWGdZemdhRAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  dedicatedNamespace: clusternet-sdqrh
  managedClusterName: clusternet-cluster-pmlbp
  result: Approved
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkluZFpWMUV6UmxCbFdUUldWa1Y0UmxBeWMzaDJOV3RuUzBabVJsaG9jWG94TkhKM2JtUkxiRTlrYUZVaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUpqYkhWemRHVnlibVYwTFhOa2NYSm9JaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbU5zZFhOMFpYSnVaWFF0YmpneWFHc3RkRzlyWlc0dGN6SnpjRFVpTENKcmRXSmxjbTVsZEdWekxtbHZMM05sY25acFkyVmhZMk52ZFc1MEwzTmxjblpwWTJVdFlXTmpiM1Z1ZEM1dVlXMWxJam9pWTJ4MWMzUmxjbTVsZEMxdU9ESm9heUlzSW10MVltVnlibVYwWlhNdWFXOHZjMlZ5ZG1salpXRmpZMjkxYm5RdmMyVnlkbWxqWlMxaFkyTnZkVzUwTG5WcFpDSTZJamN3TWpRMk1tTTBMV0prTlRFdE5HVXpNeTA1WXpnekxUWmpPV1F6TUdGall6bG1aU0lzSW5OMVlpSTZJbk41YzNSbGJUcHpaWEoyYVdObFlXTmpiM1Z1ZERwamJIVnpkR1Z5Ym1WMExYTmtjWEpvT21Oc2RYTjBaWEp1WlhRdGJqZ3lhR3NpZlEuUjJCdEQ1YzQxRm1seC1hTEFKSWQzbGxvOThSY25TakVUak5pSnVuTXg1US1jYXhwc3VkamUxekVtUF9mTHR6TjU4d21Dd3RXcHpoaXhSMWFTUHE5LXJTRlBIYzk3aDlTT3daZGdicC10alFBSjA4dllfYWdiVFRKLU1WN1dpN0xQVzRIcmt1U3RlemUzVHh2RW11NWwySlpzbG5UTXdkR3NGRVlVZzVfeWFoLUQwQ2pnTkZlR1ljUjJ1TlJBWjdkNW00Q1c5VmRkdkNsanNqRE5WX1k0RkFEbGo2cHgzSlh0SDQ4U1ZUd254TS1sNHl0eXBENjZFbG1PYXpUMmRwSWd4eWNSZ0tJSUlacWNQNnZVc0ZOM1Zka21ZZ29ydy1NSUcwc25YdzFaZ1lwRk9SUDJRa3hjd2NOUVVOTjh6VUZqSUZSSmdSSFdjNlo4aXd2eURnZTVn
```
## ClusterRole和Role
ClusterRegistrationRequest被接收后会创建ClusterRole
```powershell
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    clusternet.io/autoupdate: "true"
  labels:
    clusternet.io/created-by: clusternet-hub
    clusters.clusternet.io/bootstrapping: rbac-defaults
    clusters.clusternet.io/cluster-id: 2bcd48d2-0ead-45f7-938b-5af6af7da5fd
  name: clusternet-2bcd48d2-0ead-45f7-938b-5af6af7da5fd
rules:
- apiGroups:
  - clusters.clusternet.io
  resources:
  - clusterregistrationrequests
  verbs:
  - create
  - get
- apiGroups:
  - proxies.clusternet.io
  resourceNames:
  - 2bcd48d2-0ead-45f7-938b-5af6af7da5fd
  resources:
  - sockets
  verbs:
  - '*'
```
并会在cluster对应的namespace下创建Role
```powershell
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  annotations:
    clusternet.io/autoupdate: "true"
  labels:
    clusternet.io/created-by: clusternet-hub
    clusters.clusternet.io/bootstrapping: rbac-defaults
  name: clusternet-managedcluster-role
  namespace: clusternet-sdqrh
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
```
## ManagedCluster
clusternet-hub在接受ClusterRegistrationRequest后，会创建一个ManagedCluster。一个k8s集群一个namespace
```powershell
apiVersion: clusters.clusternet.io/v1beta1
kind: ManagedCluster
metadata:
  labels:
    clusternet.io/created-by: clusternet-agent
    clusters.clusternet.io/cluster-id: 2bcd48d2-0ead-45f7-938b-5af6af7da5fd
    clusters.clusternet.io/cluster-name: clusternet-cluster-pmlbp
  name: clusternet-cluster-pmlbp
  namespace: clusternet-sdqrh
spec:
  clusterId: 2bcd48d2-0ead-45f7-938b-5af6af7da5fd
  clusterType: EdgeCluster
  syncMode: Dual
status:
  allocatable:
    cpu: 7600m
    memory: "30792789992"
  apiserverURL: https://10.233.0.1:443 # default/kubernetes的service clusterip
  appPusher: true
  capacity:
    cpu: "8"
    memory: 32192720Ki
  clusterCIDR: 10.233.0.0/18
  conditions:
  - lastTransitionTime: "2021-12-15T07:47:10Z"
    message: managed cluster is ready.
    reason: ManagedClusterReady
    status: "True"
    type: Ready
  healthz: true
  heartbeatFrequencySeconds: 180
  k8sVersion: v1.21.5
  lastObservedTime: "2021-12-15T07:47:10Z"
  livez: true
  nodeStatistics:
    readyNodes: 1
  parentAPIServerURL: https://172.21.115.160:6443  # 父集群地址
  platform: linux/amd64
  readyz: true
  serviceCIDR: 10.233.64.0/18
  useSocket: true
```
## Subscription
应用发布的抽象，人工提交到环境中。要发布的资源即可以是helm chart，也可以是原生的k8s对象。clusternet在部署的时候会查看部署的优先级，并且支持weight，优先部署cluster级别的资源，这点跟helm部署的逻辑一致。
```powershell
apiVersion: apps.clusternet.io/v1alpha1
kind: Subscription
metadata:
  name: app-demo
  namespace: default
spec:
  # 定义应用要发布到哪个k8s集群
  subscribers: # defines the clusters to be distributed to
    - clusterAffinity:
        matchLabels:
          clusters.clusternet.io/cluster-id: dc91021d-2361-4f6d-a404-7c33b9e01118 # PLEASE UPDATE THIS CLUSTER-ID TO YOURS!!!
	# 定义了要发布哪些资源
  feeds: # defines all the resources to be deployed with
    - apiVersion: apps.clusternet.io/v1alpha1
      kind: HelmChart
      name: mysql
      namespace: default
    - apiVersion: v1
      kind: Namespace
      name: foo
    - apiVersion: v1
      kind: Service
      name: my-nginx-svc
      namespace: foo
    - apiVersion: apps/v1
      kind: Deployment
      name: my-nginx
      namespace: foo
```
## HelmChart
Subscriber发布的一个子资源之一，对应一个helm chart的完整描述
```powershell
apiVersion: apps.clusternet.io/v1alpha1
kind: HelmChart
metadata:
  name: mysql
  namespace: default
spec:
  repo: https://charts.bitnami.com/bitnami
  chart: mysql
  version: 8.6.2
  targetNamespace: abc
```
## Localization
差异化不同于其他集群的配置，用来描述namespace级别的差异化配置。
## Globalization
用来描述不同集群的cluster级别的差异化配置。
## Base
## Description
跟Base对象根据Localization和Globalization渲染得到的最终要发布到集群中的最终对象。

# 安装
本文使用kind进行测试，使用kind创建两个k8s集群host和member1，两个集群的apiserver均监听在宿主机的端口，确保从一个集群可以访问到另外一个集群的apiserver。

在父集群执行如下命令安装clusternet管控组件：
```powershell
helm repo add clusternet https://clusternet.github.io/charts
helm install clusternet-hub -n clusternet-system --create-namespace clusternet/clusternet-hub
kubectl apply -f https://raw.githubusercontent.com/clusternet/clusternet/main/manifests/samples/cluster_bootstrap_token.yaml
```
会在clusternet-system namespace下创建deployment clusternet-hub。

最后一条命令，会创建一个secret，其中的token信息为07401b.f395accd246ae52d，在子集群注册的时候需要用到。
```powershell
apiVersion: v1
kind: Secret
metadata:
  # Name MUST be of form "bootstrap-token-<token id>"
  name: bootstrap-token-07401b
  namespace: kube-system

# Type MUST be 'bootstrap.kubernetes.io/token'
type: bootstrap.kubernetes.io/token
stringData:
  # Human readable description. Optional.
  description: "The bootstrap token used by clusternet cluster registration."

  # Token ID and secret. Required.
  token-id: 07401b
  token-secret: f395accd246ae52d

  # Expiration. Optional.
  expiration: 2025-05-10T03:22:11Z

  # Allowed usages.
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"

  # Extra groups to authenticate the token as. Must start with "system:bootstrappers:"
  auth-extra-groups: system:bootstrappers:clusternet:register-cluster-token
```

在子集群执行如下命令，其中parentURL要修改为父集群的apiserver地址
```powershell
helm repo add clusternet https://clusternet.github.io/charts
helm install clusternet-agent -n clusternet-system --create-namespace \
  --set parentURL=https://10.0.248.96:8443 \
  --set registrationToken=07401b.f395accd246ae52d \
  clusternet/clusternet-agent
```
其中parentURL为父集群的apiserver地址，registrationToken为父集群注册的token信息。

# 访问子集群

## kubectl-clusternet命令行方式访问
安装kubectl krew插件，参考 [https://krew.sigs.k8s.io/docs/user-guide/setup/install/](https://krew.sigs.k8s.io/docs/user-guide/setup/install/)，执行如下命令：
```powershell
yum install git -y
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
echo 'export PATH="${PATH}:${HOME}/.krew/bin"' >>  ~/.bashrc
source ~/.bashrc
```
安装kubectl插件clusternet：kubectl krew install clusternet

使用kubectl clusternet get可以看到发布的应用，非发布的应用看不到。

## 代码层面访问子集群
可以参照例子：[https://github.com/clusternet/clusternet/blob/main/examples/clientgo/demo.go#L42-L45](https://github.com/clusternet/clusternet/blob/main/examples/clientgo/demo.go#L42-L45)

改动代码非常小，仅需要获取到对应集群的config信息即可。
```powershell
config, err := rest.InClusterConfig()
	if err != nil {
		klog.Error(err)
		os.Exit(1)
	}

	// This is the ONLY place you need to wrap for Clusternet
	config.Wrap(func(rt http.RoundTripper) http.RoundTripper {
		return clientgo.NewClusternetTransport(config.Host, rt)
	})

	// now we could create and visit all the resources
	client := kubernetes.NewForConfigOrDie(config)
	_, err = client.CoreV1().Namespaces().Create(context.TODO(), &corev1.Namespace{
		ObjectMeta: metav1.ObjectMeta{
			Name: "demo",
		},
	}, metav1.CreateOptions{})
```

# 应用发布
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/images/clusternet3.png)
在主集群提交如下的yaml文件
```powershell
apiVersion: apps.clusternet.io/v1alpha1
kind: Subscription
metadata:
  name: app-demo
  namespace: default
spec:
  subscribers: # defines the clusters to be distributed to
    - clusterAffinity:
        matchLabels:
          clusters.clusternet.io/cluster-id: dc91021d-2361-4f6d-a404-7c33b9e01118 # PLEASE UPDATE THIS CLUSTER-ID TO YOURS!!!
  feeds: # defines all the resources to be deployed with
    - apiVersion: apps.clusternet.io/v1alpha1
      kind: HelmChart
      name: mysql
      namespace: default
    - apiVersion: v1
      kind: Namespace
      name: foo
    - apiVersion: apps/v1
      kind: Service
      name: my-nginx-svc
      namespace: foo
    - apiVersion: apps/v1
      kind: Deployment
      name: my-nginx
      namespace: foo
```

# 实现分析
在主集群实现了两个aggregated apiservice
```powershell
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  annotations:
    meta.helm.sh/release-name: clusternet-hub
    meta.helm.sh/release-namespace: clusternet-system
  labels:
    app.kubernetes.io/instance: clusternet-hub
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: clusternet-hub
    helm.sh/chart: clusternet-hub-0.2.1
  name: v1alpha1.proxies.clusternet.io
spec:
  group: proxies.clusternet.io
  groupPriorityMinimum: 1000
  insecureSkipTLSVerify: true
  service:
    name: clusternet-hub
    namespace: clusternet-system
    port: 443
  version: v1alpha1
  versionPriority: 100
```
```powershell
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  annotations:
    meta.helm.sh/release-name: clusternet-hub
    meta.helm.sh/release-namespace: clusternet-system
  labels:
    app.kubernetes.io/instance: clusternet-hub
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: clusternet-hub
    helm.sh/chart: clusternet-hub-0.2.1
  name: v1alpha1.shadow
spec:
  group: shadow
  groupPriorityMinimum: 1
  insecureSkipTLSVerify: true
  service:
    name: clusternet-hub
    namespace: clusternet-system
    port: 443
  version: v1alpha1
  versionPriority: 1
```
# 相关链接

- [https://mp.weixin.qq.com/s/BSgb2uoAuHbxQOUvn8fEsA](https://mp.weixin.qq.com/s/BSgb2uoAuHbxQOUvn8fEsA)