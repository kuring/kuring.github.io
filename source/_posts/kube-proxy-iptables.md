---
title: kube-proxy iptables规则分析
date: 2020-02-08 12:10:33
tags:
---

kube-proxy默认使用iptables规则来做k8s集群内部的负载均衡，本文通过例子来分析创建的iptabels规则。

主要的自定义链涉及到：

- KUBE-SERVICES： 访问集群内服务的CLusterIP数据包入口，根据匹配到的目标ip+port将数据包分发到相应的KUBE-SVC-xxx链上。一个Service对应一条规则。由OUTPUT链调用。
- KUBE-NODEPORTS: 用来匹配nodeport端口号，并将规则转发到KUBE-SVC-xxx。一个NodePort类型的Service一条。在KUBE-SERVICES链的最后被调用
- KUBE-SVC-xxx：相当于是负载均衡，将流量利用random模块均分到KUBE-SEP-xxx链上。
- KUBE-SEP-xxx：通过dnat规则将连接的目的地址和端口号做dnat，从Service的ClusterIP或者NodePort转换为后端的pod ip
- KUBE-MARK-MASQ: 使用mark命令，对数据包设置标记0x4000/0x4000。在KUBE-POSTROUTING链上有MARK标记的数据包进行一次MASQUERADE，即SNAT，会用节点ip替换源ip地址。

## 环境准备

创建nginx deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-svc
    version: nginx
  name: nginx
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-svc
  template:
    metadata:
      labels:
        app: nginx-svc
        version: nginx
    spec:
      containers:
        - image: 'nginx:1.9.0'
          name: nginx
          ports:
            - containerPort: 443
              protocol: TCP
            - containerPort: 80
              protocol: TCP
```

创建service对象

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: default
spec:
  ports:
    - name: '80'
      port: 8000
      protocol: TCP
      targetPort: 80
      nodePort: 30080
  selector:
    app: nginx-svc
  sessionAffinity: None
  type: NodePort
```

环境信息如下：
- 容器网段：172.20.0.0/16
- Service ClusterIP cidr: 192.168.0.0/20
- k8s版本：

提交后创建出来的信息如下：
- Service ClusterIP：192.168.103.148
- nginx pod的两个ip地址：172.16.3.3 172.16.4.4

## 从宿主机上访问ClusterIP

从本机请求ClusterIP的数据包会经过iptables的链：OUTPUT -> POSTROUTING

要想详细知道iptabels的执行情况，可以通过iptables的trace功能。如何开启trace功能可以参考：http://kuring.me/post/iptables/。

![image](https://kuring.oss-cn-beijing.aliyuncs.com/common/kube-proxy-clusterip.png)


执行 `iptables -nvL OUTPUT -t nat` 可以看到如下的iptables规则命令

```
pkts bytes target         prot opt in     out     source               destination         
17M  1150M KUBE-SERVICES  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */
```

执行 `iptables -nvL KUBE-SERVICES -t nat` 可以查看自定义链的具体内容，里面包含了多条规则，其中跟当前Service相关的规则如下。

```
pkts bytes target                     prot opt in     out     source               destination
1    60    KUBE-SVC-Y5VDFIEGM3DY2PZE  tcp  --  *      *       0.0.0.0/0            192.168.103.148      /* default/nginx-svc:80 cluster IP */ tcp dpt:8000
```

执行 `iptables -nvL KUBE-SVC-Y5VDFIEGM3DY2PZE -t nat` 查看自定义链的具体规则

```
pkts  bytes target                     prot opt in     out     source               destination
0     0     KUBE-SEP-IFV44I3EMZAL3LH3  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/nginx-svc:80 */ statistic mode random probability 0.50000000000
1    60     KUBE-SEP-6PNQETFAD2JPG53P  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/nginx-svc:80 */
```

上述规则会按照特定的概率将流量均等的执行自定义链的规则，两个自定义的链的规则跟endpoint相关，执行 `iptables -nvL  KUBE-SEP-IFV44I3EMZAL3LH3 -t nat`可查看endpoint级别的iptabels规则。dnat操作会修改数据包的目的地址和端口，从clusterip+service port修改为访问pod ip+pod端口。

```
pkts bytes target          prot opt in     out     source               destination
0     0    KUBE-MARK-MASQ  all  --  *      *       172.16.3.3           0.0.0.0/0            /* default/nginx-svc:80 */
0     0    DNAT            tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/nginx-svc:80 */ tcp to:172.16.3.3:80
```

会在dnat操作之前为对数据包执行打标签操作。KUBE-MARK-MASQ 自定义链为对数据包打标记的自定义规则，执行 `iptables -nvL  KUBE-MARK-MASQ -t nat`

```
pkts bytes target     prot opt in     out     source               destination         
 1    60   MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            MARK or 0x4000
```

接下来看一下POSTROUTING链上的规则，`iptables -nvL  POSTROUTING -t nat`。

```
pkts bytes target            prot opt in     out     source               destination         
205K   13M KUBE-POSTROUTING  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes postrouting rules */
```

继续看一下KUBE-POSTROUTING链的内容，`iptables -nvL  KUBE-POSTROUTING -t nat`，其中最后一条的MASQUERADE指令的操作实际上为SNAT操作。

```
Chain KUBE-POSTROUTING (1 references)
pkts bytes target     prot opt in     out     source               destination         
6499  398K RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0            mark match ! 0x4000/0x4000
   1    60 MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            MARK xor 0x4000
   1    60 MASQUERADE  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service traffic requiring SNAT */
```

即从本机访问service clusterip的数据包，在output链上经过了dnat操作，在postrouting链上经过了snat操作后，最终会发往目标pod。pod在处理完请求后，回的数据包最终会经过nat的逆过程返回到本机。

## 外部访问nodeport

从外部访问本机的nodeport数据包会经过iptables的链：PREROUTING -> FORWARD -> POSTROUTING

![image](https://kuring.oss-cn-beijing.aliyuncs.com/common/kube-proxy-clusterip.png)

nodeport都是被外部访问的情况，入口位于PREROUTING链上。执行 `iptables -nvL PREROUTING  -t nat`：

```
pkts bytes target         prot opt in     out     source               destination         
349K   21M KUBE-SERVICES  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */
```

在KUBE-SERVICES链的最后一条规则为跳转到KUBE-NODEPORTS链

```
 4079  246K KUBE-NODEPORTS  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service nodeports; NOTE: this must be the last rule in this chain */ ADDRTYPE match dst-type LOCAL
```

执行`iptables -nvL KUBE-NODEPORTS -t nat`， 查看KUBE-NODEPORTS链 
```
pkts bytes target                     prot opt in     out     source               destination         
0     0    KUBE-MARK-MASQ             tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/nginx-svc:80 */ tcp dpt:30080
0     0    KUBE-SVC-Y5VDFIEGM3DY2PZE  tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/nginx-svc:80 */ tcp dpt:30080
```

其中KUBE-MARK-MASQ链只有一条规则，即打上0x4000的标签。

```
pkts bytes target     prot opt in     out     source               destination         
0     0    MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            MARK or 0x4000
```

自定义链KUBE-SVC-Y5VDFIEGM3DY2PZE的内容如下，跟clusterip的规则是重叠的：
```
pkts bytes target     prot opt in     out     source               destination         
0    0     KUBE-SEP-IFV44I3EMZAL3LH3  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/nginx-svc:80 */ statistic mode random probability 0.50000000000
0    0     KUBE-SEP-6PNQETFAD2JPG53P  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/nginx-svc:80 */
```

KUBE-SEP-IFV44I3EMZAL3LH3的内容为，会经过一次DNAT操作:
```
pkts bytes target          prot opt in     out     source               destination         
0    0     KUBE-MARK-MASQ  all  --  *      *       172.16.3.3           0.0.0.0/0            /* default/nginx-svc:80 */
0    0     DNAT            tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/nginx-svc:80 */ tcp to:172.16.3.3:80
```

在经过了PREROUTING链后，接下来会判断目的ip地址不是本机的ip地址，接下来会经过FORWARD链。在FORWARD链上，仅做了一件事情，就是将前面大了0x4000的数据包允许转发。

```
pkts bytes target              prot opt in     out     source               destination         
0    0 KUBE-FORWARD            all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding rules */
0    0 KUBE-SERVICES           all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes service portals */
0    0 KUBE-EXTERNAL-SERVICES  all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate NEW /* kubernetes externally-visible service portals */
```

KUBE-FORWARD的内容如下：

```
pkts bytes target     prot opt in     out     source               destination         
0     0    DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate INVALID
0     0    ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding rules */ mark match 0x4000/0x4000
0     0    ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding conntrack pod source rule */ ctstate RELATED,ESTABLISHED
0     0    ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding conntrack pod destination rule */ ctstate RELATED,ESTABLISHED
```

跟clusterip一样，会在POSTROUTING阶段匹配mark为0x4000/0x4000的数据包，并进行一次MASQUERADE转换，将ip包替换为宿主上的ip地址。

加入这里不做MASQUERADE，流量发到目的的pod后，pod回包时目的地址为发起端的源地址，而发起端的源地址很可能是在k8s集群外部的，此时pod发回的包是不能回到发起端的。NodePort跟ClusterIP的最大不同就是NodePort的发起端很可能是在集群外部的，从而这里必须做一层SNAT转换。

在上述分析中，访问NodePort类型的Service会经过snat，从而服务端的pod不能获取到正确的客户端ip。可以设置Service的spec.externalTrafficPolicy为Local，此时iptables规则只会将ip包转发给运行在这台宿主机上的pod，而不需要经过snat。pod回包时，直接回复源ip地址即可，此时源ip地址是可达的，因为源ip地址跟宿主机是可达的。如果所在的宿主机上没有pod，那么此时流量就不可以转发，此为限制。

## 使用LoadBalancer类型访问的情况

### externalTrafficPolicy为local

```
-A KUBE-SERVICES -d 10.149.30.186/32 -p tcp -m comment --comment "acs-system/nginx-ingress-lb-cloudbiz:http loadbalancer IP" -m tcp --dport 80 -j KUBE-FW-76HLDRT5IPNSMPF5
-A KUBE-FW-76HLDRT5IPNSMPF5 -m comment --comment "acs-system/nginx-ingress-lb-cloudbiz:http loadbalancer IP" -j KUBE-XLB-76HLDRT5IPNSMPF5
-A KUBE-FW-76HLDRT5IPNSMPF5 -m comment --comment "acs-system/nginx-ingress-lb-cloudbiz:http loadbalancer IP" -j KUBE-MARK-DROP

# 10.149.112.0/23为pod网段
-A KUBE-XLB-76HLDRT5IPNSMPF5 -s 10.149.112.0/23 -m comment --comment "Redirect pods trying to reach external loadbalancer VIP to clusterIP" -j KUBE-SVC-76HLDRT5IPNSMPF5
-A KUBE-XLB-76HLDRT5IPNSMPF5 -m comment --comment "Balancing rule 0 for acs-system/nginx-ingress-lb-cloudbiz:http" -j KUBE-SEP-XZXLBWOKJBSJBGVU

-A KUBE-SVC-76HLDRT5IPNSMPF5 -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-XZXLBWOKJBSJBGVU
-A KUBE-SVC-76HLDRT5IPNSMPF5 -j KUBE-SEP-GP4UCOZEF3X7PGLR

-A KUBE-SEP-XZXLBWOKJBSJBGVU -s 10.149.112.45/32 -j KUBE-MARK-MASQ
-A KUBE-SEP-XZXLBWOKJBSJBGVU -p tcp -m tcp -j DNAT --to-destination 10.149.112.45:80
-A KUBE-SEP-GP4UCOZEF3X7PGLR -s 10.149.112.46/32 -j KUBE-MARK-MASQ
-A KUBE-SEP-GP4UCOZEF3X7PGLR -p tcp -m tcp -j DNAT --to-destination 10.149.112.46:80
```

## 缺点

1. iptables规则特别乱，一旦出现问题非常难以排查
2. 由于iptables规则是串行执行，算法复杂度为O(n)，一旦iptables规则多了后，性能将非常差。
3. iptables规则提供的负载均衡功能非常有限，不支持较为复杂的负载均衡算法。