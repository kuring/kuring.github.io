title: k8s ipv4/ipv6双栈
date: 2022-01-16 23:41:33
tags:
---
# ipv6在k8s中的支持情况

ipv6特性在k8s中作为一个特定的feature IPv6DualStack来管理。在1.15版本到1.20版本该feature为alpha版本，默认不开启。从1.21版本开始，该feature为beta版本，默认开启，支持pod和service网络的双栈。1.23版本变为稳定版本。

## 配置双栈
alpha版本需要通过kube-apiserver、kube-controller-manager、kubelet和kube-proxy的--feature-gates="IPv6DualStack=true"命令来开启该特性，beta版本该特性默认开启。<br />​

kube-apiserver：
- `--service-cluster-ip-range=<IPv4 CIDR>,<IPv6 CIDR>`

kube-controller-manager:
- `--cluster-cidr=<IPv4 CIDR>,<IPv6 CIDR>`
- `--service-cluster-ip-range=<IPv4 CIDR>,<IPv6 CIDR>`
- `--node-cidr-mask-size-ipv4|--node-cidr-mask-size-ipv6` 对于 IPv4 默认为 /24，对于 IPv6 默认为 /64

kube-proxy:
- `--cluster-cidr=<IPv4 CIDR>,<IPv6 CIDR>`

# 使用kind创建k8s集群

由于ipv6的环境并不容易找到，可以使用kind快速在本地拉起一个k8s的单集群环境，同时kind支持ipv6的配置。

检查当前系统是否支持ipv6，如果输出结果为0，说明当前系统支持ipv6.

```powershell
$ sysctl -a | grep net.ipv6.conf.all.disable_ipv6
net.ipv6.conf.all.disable_ipv6 = 0
```
创建ipv6单栈k8s集群执行如下命令：
```powershell
cat > kind-ipv6.conf <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: ipv6-only
networking:
  ipFamily: ipv6
# networking:
  # apiServerAddress: "127.0.0.1"
  # apiServerPort: 6443
EOF
kind create cluster --config kind-ipv6.conf
```
创建ipv4/ipv6双栈集群执行如下命令：
```powershell
cat > kind-dual-stack.conf <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: dual-stack
networking:
  ipFamily: dual
# networking:
  # apiServerAddress: "127.0.0.1"
  # apiServerPort: 6443
EOF
kind create cluster --config kind-dual-stack.conf
```
# service网络

## spec定义
k8s的service网络为了支持双栈，新增加了几个字段。

.spec.ipFamilies用来设置地址族，service一旦场景后该值不可认为修改，但可以通过修改.spec.ipFamilyPolicy来简介修改该字段的值。为数组格式，支持如下值：
- ["IPv4"]
- ["IPv6"]
- ["IPv4","IPv6"] （双栈）
- ["IPv6","IPv4"] （双栈）

上述数组中的第一个元素会决定.spec.ClusterIP中的字段。<br />​

.spec.ClusterIPs：由于.spec.ClusterIP的value为字符串，不支持同时设置ipv4和ipv6两个ip地址。因此又扩展出来了一个.spec.ClusterIPs字段，该字段的value为宿主元祖。在Headless类型的Service情况下，该字段的值为None。<br />​

.spec.ClusterIP：该字段的值跟.spec.ClusterIPs中的第一个元素的值保持一致。<br />​

.spec.ipFamilyPolicy支持如下值：
- SingleStack：默认值。会使用.spec.ipFamilies数组中配置的第一个协议来分配cluster ip。如果没有指定.spec.ipFamilies，会使用service-cluster-ip-range配置中第一个cidr中来配置地址。
- PreferDualStack：如果 .spec.ipFamilies 没有设置，使用 k8s 集群默认的 ipFamily。
- RequireDualStack：同时分配ipv4和ipv6地址。.spec.ClusterIP的值会从.spec.ClusterIPs选择第一个元素。

## service配置场景
先创建如下的deployment，以便于后面试验的service可以关联到pod。
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: MyApp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: MyApp
  template:
    metadata:
      labels:
        app: MyApp
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### 未指定任何协议栈信息
在没有指定协议栈信息的Service，创建出来的service .spec.ipFamilyPolicy为SingleStack。同时会使用service-cluster-ip-range配置中第一个cidr中来配置地址。如果第一个cidr为ipv6，则service分配的clusterip为ipv6地址。创建如下的service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app: MyApp
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
```
会生成如下的service
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: MyApp
  name: my-service
  namespace: default
spec:
  clusterIP: 10.96.80.114
  clusterIPs:
  - 10.96.80.114
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: MyApp
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```

### 指定.spec.ipFamilyPolicy为PreferDualStack
创建如下的service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-2
  labels:
    app: MyApp
spec:
  ipFamilyPolicy: PreferDualStack
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
```
提交到环境后会生成如下的service，可以看到.spec.clusterIPs中的ip地址跟ipFamilies中的顺序一致。

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: MyApp
  name: my-service-2
  namespace: default
spec:
  clusterIP: 10.96.221.70
  clusterIPs:
  - 10.96.221.70
  - fd00:10:96::7d1
  ipFamilies:
  - IPv4
  - IPv6
  ipFamilyPolicy: PreferDualStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: MyApp
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```
查看Endpoints，可以看到subsets中的地址为pod的ipv4协议地址。Endpoints中的地址跟service的.spec.ipFamilies数组中的第一个协议的值保持一致。
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  labels:
    app: MyApp
  name: my-service-2
  namespace: default
subsets:
- addresses:
  - ip: 10.244.0.5
    nodeName: dual-stack-control-plane
    targetRef:
      kind: Pod
      name: nginx-deployment-65b5dd4c68-vrfps
      namespace: default
      resourceVersion: "16875"
      uid: 30a1d787-f799-4250-8c56-c96564ca9239
  - ip: 10.244.0.6
    nodeName: dual-stack-control-plane
    targetRef:
      kind: Pod
      name: nginx-deployment-65b5dd4c68-wgz72
      namespace: default
      resourceVersion: "16917"
      uid: 8166d43e-2702-45c6-839e-b3005f44f647
  - ip: 10.244.0.7
    nodeName: dual-stack-control-plane
    targetRef:
      kind: Pod
      name: nginx-deployment-65b5dd4c68-x4lt5
      namespace: default
      resourceVersion: "16896"
      uid: f9c2968f-ca59-4ba9-a69f-358c202a964b
  ports:
  - port: 80
    protocol: TCP
```
接下来指定.spec.ipFamilies的顺序再看一下执行的结果，创建如下的service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-3
  labels:
    app: MyApp
spec:
  ipFamilyPolicy: PreferDualStack
  ipFamilies:
  - IPv6
  - IPv4
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
```
在环境中生成的service如下，可以看到.spec.clusterIPs中的顺序第一个为ipv6地址，.spec.clusterIP同样为ipv6地址。
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: MyApp
  name: my-service-3
  namespace: default
spec:
  clusterIP: fd00:10:96::c306
  clusterIPs:
  - fd00:10:96::c306
  - 10.96.147.82
  ipFamilies:
  - IPv6
  - IPv4
  ipFamilyPolicy: PreferDualStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: MyApp
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```
查看Endpoints，可以看到subsets中的地址为pod的ipv6协议地址。Endpoints中的地址跟service的.spec.ipFamilies数组中的第一个协议的值保持一致。
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  labels:
    app: MyApp
  name: my-service-3
  namespace: default
subsets:
- addresses:
  - ip: fd00:10:244::5
    nodeName: dual-stack-control-plane
    targetRef:
      kind: Pod
      name: nginx-deployment-65b5dd4c68-vrfps
      namespace: default
      resourceVersion: "16875"
      uid: 30a1d787-f799-4250-8c56-c96564ca9239
  - ip: fd00:10:244::6
    nodeName: dual-stack-control-plane
    targetRef:
      kind: Pod
      name: nginx-deployment-65b5dd4c68-wgz72
      namespace: default
      resourceVersion: "16917"
      uid: 8166d43e-2702-45c6-839e-b3005f44f647
  - ip: fd00:10:244::7
    nodeName: dual-stack-control-plane
    targetRef:
      kind: Pod
      name: nginx-deployment-65b5dd4c68-x4lt5
      namespace: default
      resourceVersion: "16896"
      uid: f9c2968f-ca59-4ba9-a69f-358c202a964b
  ports:
  - port: 80
    protocol: TCP
```
## 单栈和双栈之间的切换
虽然.spec.ipFamilies字段不允许直接修改，但.spec.ipFamilyPolicy字段允许修改，但并不影响单栈和双栈之间的切换。<br />单栈变双栈只需要修改.spec.ipFamilyPolicy从SingleStack变为PreferDualStack或者RequireDualStack即可。<br />双栈同样可以变为单栈，只需要修改.spec.ipFamilyPolicy从PreferDualStack或者RequireDualStack变为SingleStack即可。此时.spec.ipFamilies会自动变更为一个元素，.spec.clusterIPs同样会变更为一个元素。
## LoadBalancer类型的Service
对于LoadBalancer类型的Service，单栈的情况下，会在.status.loadBalancer.ingress设置vip地址。如果是ingress中的ip地址为双栈，此时应该是将双栈的vip地址同时写到.status.loadBalancer.ingress中，并且要保证其顺序跟serivce的.spec.ipFamilies中的顺序一致。
# pod网络
pod网络要支持ipv6，需要容器网络插件的支持。为了支持ipv6特性，新增加了.status.podIPs字段，用来展示pod上分配的ipv4和ipv6的信息。.status.podIP字段的值跟.status.podIPs数组的第一个元素的值保持一致。
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    k8s-app: kube-dns
    pod-template-hash: 558bd4d5db
  name: coredns-558bd4d5db-b2zbj
  namespace: kube-system
spec:
  containers:
  - args:
    - -conf
    - /etc/coredns/Corefile
    image: k8s.gcr.io/coredns/coredns:v1.8.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 5
      httpGet:
        path: /health
        port: 8080
        scheme: HTTP
      initialDelaySeconds: 60
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 5
    name: coredns
    ports:
    - containerPort: 53
      name: dns
      protocol: UDP
    - containerPort: 53
      name: dns-tcp
      protocol: TCP
    - containerPort: 9153
      name: metrics
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        path: /ready
        port: 8181
        scheme: HTTP
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    resources:
      limits:
        memory: 170Mi
      requests:
        cpu: 100m
        memory: 70Mi
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        add:
        - NET_BIND_SERVICE
        drop:
        - all
      readOnlyRootFilesystem: true
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /etc/coredns
      name: config-volume
      readOnly: true
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-hgxnc
      readOnly: true
  dnsPolicy: Default
  enableServiceLinks: true
  nodeName: dual-stack-control-plane
  nodeSelector:
    kubernetes.io/os: linux
  preemptionPolicy: PreemptLowerPriority
  priority: 2000000000
  priorityClassName: system-cluster-critical
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: coredns
  serviceAccountName: coredns
  terminationGracePeriodSeconds: 30
  tolerations:
  - key: CriticalAddonsOnly
    operator: Exists
    ...
  volumes:
  - configMap:
      defaultMode: 420
      items:
      - key: Corefile
        path: Corefile
      name: coredns
    name: config-volume
  - name: kube-api-access-hgxnc
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2022-01-16T12:51:24Z"
    status: "True"
    type: Initialized
    ...
  containerStatuses:
  - containerID: containerd://6da36ab908291ca1b4141a86d70f8c2bb150a933336d852bcabe2118aa1a3439
    image: k8s.gcr.io/coredns/coredns:v1.8.0
    imageID: sha256:296a6d5035e2d6919249e02709a488d680ddca91357602bd65e605eac967b899
    lastState: {}
    name: coredns
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2022-01-16T12:51:27Z"
  hostIP: 172.18.0.2
  phase: Running
  podIP: 10.244.0.3
  podIPs:
  - ip: 10.244.0.3
  - ip: fd00:10:244::3
  qosClass: Burstable
  startTime: "2022-01-16T12:51:24Z"
```
pod中要想获取到ipv4、ipv6地址，可以通过downward api的形式将.status.podIPs以环境变量的形式传递到容器中，在pod中通过环境变量获取到的格式为: `10.244.1.4,a00:100::4`。
# k8s node
在宿主机上可以通过ip addr show eth0的方式来查看网卡上的ip地址。
```yaml
# ip addr show eth0
23: eth0@if24: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fc00:f853:ccd:e793::2/64 scope global nodad
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe12:2/64 scope link
       valid_lft forever preferred_lft forever
```
宿主机的网卡上有ipv6地址，k8s node上的.status.addresses中有所体现，type为InternalIP即包含了ipv4地址，又包含了ipv6地址，但此处并没有字段标识当前地址为ipv4，还是ipv6。
```yaml
apiVersion: v1
kind: Node
metadata:
  annotations:
    kubeadm.alpha.kubernetes.io/cri-socket: unix:///run/containerd/containerd.sock
    node.alpha.kubernetes.io/ttl: "0"
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: dual-stack-control-plane
    kubernetes.io/os: linux
    node-role.kubernetes.io/control-plane: ""
    node-role.kubernetes.io/master: ""
    node.kubernetes.io/exclude-from-external-load-balancers: ""
  name: dual-stack-control-plane
spec:
  podCIDR: 10.244.0.0/24
  podCIDRs:
  - 10.244.0.0/24
  - fd00:10:244::/64
  providerID: kind://docker/dual-stack/dual-stack-control-plane
status:
  addresses:
  - address: 172.18.0.2
    type: InternalIP
  - address: fc00:f853:ccd:e793::2
    type: InternalIP
  - address: dual-stack-control-plane
    type: Hostname
  allocatable:
    cpu: "4"
    ephemeral-storage: 41152812Ki
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    memory: 15936568Ki
    pods: "110"
  capacity:
    cpu: "4"
    ephemeral-storage: 41152812Ki
    hugepages-1Gi: "0"
    hugepages-2Mi: "0"
    memory: 15936568Ki
    pods: "110"
  conditions:
  - lastHeartbeatTime: "2022-01-16T15:01:48Z"
    lastTransitionTime: "2022-01-16T12:50:54Z"
    message: kubelet has sufficient memory available
    reason: KubeletHasSufficientMemory
    status: "False"
    type: MemoryPressure
    ...
  daemonEndpoints:
    kubeletEndpoint:
      Port: 10250
  images:
  - names:
    - k8s.gcr.io/kube-proxy:v1.21.1
    sizeBytes: 132714699
    ...
  nodeInfo:
    architecture: amd64
    bootID: 7f95abb9-7731-4a8c-9258-4a91cdcfb2ca
    containerRuntimeVersion: containerd://1.5.2
    kernelVersion: 4.18.0-305.an8.x86_64
    kubeProxyVersion: v1.21.1
    kubeletVersion: v1.21.1
    machineID: 8f6a98bffc184893ab6bc260e705421b
    operatingSystem: linux
    osImage: Ubuntu 21.04
    systemUUID: f7928fdb-32be-4b6e-8dfd-260b6820f067
```
# Ingress

Ingress 可以通过开关 `disable-ipv6` 来控制是否开启 ipv6，默认 ipv6 开启。

在开启 ipv6 的情况下，如果 nginx ingress 的 pod 本身没有ipv6 的 ip地址，则在 nginx 的配置文件中并不会监听 ipv6 的端口号。

```
        listen 80  ;
        listen 443  ssl http2 ;
```

如果 nginx ingress 的 pod 本身包含 ipv6 地址，则 nginx 的配置文件如下：

```
        listen 80  ;
        listen [::]:80  ;
        listen 443  ssl http2 ;
        listen [::]:443  ssl http2 ;
```





参考资料：

- 阿里云ACK的Nginx Ingress支持ipv6特性：https://help.aliyun.com/document_detail/378167.html
- [Nginx Ingress 支持 ipv6]([ConfigMap - NGINX Ingress Controller (kubernetes.github.io)](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#disable-ipv6))


# 引用

- [https://kubernetes.io/docs/concepts/services-networking/dual-stack/](https://kubernetes.io/docs/concepts/services-networking/dual-stack/)
- [https://kubernetes.io/docs/tasks/network/validate-dual-stack/](https://kubernetes.io/docs/tasks/network/validate-dual-stack/)
- [https://kind.sigs.k8s.io/docs/user/configuration/](https://kind.sigs.k8s.io/docs/user/configuration/)

<br />
