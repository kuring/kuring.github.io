title: 使用 kubeadm 安装 k8s 集群
date: 2022-12-07 00:07:20
tags:
author:
---
本文将通过 kubeadm 实现单 master 节点模式和集群模式两种部署方式。

## 所有节点均需初始化操作

所有节点均需做的操作。

### 主机准备

```
cat > /etc/sysctl.d/kubernets.conf <<EOF
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness=0
EOF
sysctl --system
modprobe br_netfilter
```


### 安装containerd

由于 dockerd 从 k8s 1.24 版本开始不再支持，这里选择 containerd。参考: [Getting started with containerd](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)

#### 手工安装

安装 containerd

```
wget https://github.com/containerd/containerd/releases/download/v1.6.11/containerd-1.6.11-linux-amd64.tar.gz
tar Cxzvf /usr/local containerd-1.6.11-linux-amd64.tar.gz
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -P /usr/local/lib/systemd/system/
systemctl daemon-reload
systemctl enable --now containerd
```

执行 `containerd config default > /etc/containerd/config.toml` 生成 containerd 的默认配置文件。

安装 runc

```
wget https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64
install -m 755 runc.amd64 /usr/local/sbin/runc
```

#### yum 源安装

```
yum install -y yum-utils
yum-config-manager --add-repo     https://download.docker.com/linux/centos/docker-ce.repo
yum install containerd.io -y
systemctl daemon-reload
systemctl enable --now containerd
```

通过 yum 安装的containerd 没有启用 cri，在其配置文件 /etc/containerd/config.toml 中包含了 `disabled_plugins = ["cri"]` 配置，需要将配置信息注释后并重启 containerd。

```
sed -i 's/disabled_plugins/#disabled_plugins/'  /etc/containerd/config.toml
systemctl restart containerd
```

### 安装 kubeadm/kubelet/kubectl

```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# Set SELinux in permissive mode (effectively disabling it)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

## 单 master 节点模式

| 节点                | 角色        |
| ------------------- | ----------- |
| 172.21.115.190<br/> | master 节点 |
| 172.21.115.191      | 普通节点    |

### kubeadm 初始化

创建文件 kubeadm-config.yaml

```
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.25.4
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```

kubeadm init --config kubeadm-config.yaml 

```
kubeadm config print init-defaults --component-configs KubeletConfiguration > cluster.yaml
kubeadm init --config cluster.yaml
```

接下来初始化 kubeconfig 文件，这样即可通过 kubectl 命令来访问 k8s 了。

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



### 安装网络插件

刚部署完成的节点处于NotReady 的状态，原因是因为还没有安装网络插件。本文直接使用 cilim 网络插件。


```
# 安装 cilium 客户端
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/master/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}

# 网络插件初始化
cilium install
```

在安装完网络插件后，node 节点即可变为 ready 状态。

查看环境中包含如下的 pod:

```
$ kubectl  get pod  -A
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   cilium-7zj7t                     1/1     Running   0          82s
kube-system   cilium-operator-bc4d5b54-kvqqx   1/1     Running   0          82s
kube-system   coredns-565d847f94-hrm9b         1/1     Running   0          14m
kube-system   coredns-565d847f94-z5kwr         1/1     Running   0          14m
kube-system   etcd-k8s002                      1/1     Running   0          14m
kube-system   kube-apiserver-k8s002            1/1     Running   0          14m
kube-system   kube-controller-manager-k8s002   1/1     Running   0          14m
kube-system   kube-proxy-bhpqr                 1/1     Running   0          14m
kube-system   kube-scheduler-k8s002            1/1     Running   0          14m
```

### 添加其他节点

```shell
kubeadm join 172.21.115.189:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:457fba2c4181a5b02d2a4f202dfe20f9ce5b9f2274bf40b6d25a8a8d4a7ce440 
```

此时即可以将节点添加到 k8s 集群中

```
$ kubectl  get node 
NAME     STATUS   ROLES           AGE   VERSION
k8s002   Ready    control-plane   79m   v1.25.4
k8s003   Ready    <none>          35s   v1.25.4
```

### 节点清理

#### 清理普通节点

```
kubectl drain <node name> --delete-emptydir-data --force --ignore-daemonsets
kubeadm reset 
# 清理 iptabels 规则
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
kubectl delete node <node name>
```

#### 清理 control plan 节点

```
kubeadm reset
```

## 集群模式部署

待补充，参考文档：[Creating Highly Available Clusters with kubeadm | Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)

## 资料

- [Bootstrapping clusters with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/)