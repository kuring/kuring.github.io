title: ecs的Linux主机上快速创建测试k8s集群
date: 2021-12-15 21:32:00
tags:
---
经常有快速创建一个测试k8s集群的场景，为了能够快速完成，整理了如下的命令，即可在主机上快速启动一个k8s集群。部分命令需要外网访问，推荐直接使用海外的主机。

# 安装docker

下面命令可以安装最新版本的docker-ce

```shell
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io -y
systemctl enable docker && systemctl start docker
yum install vim git make -y
```
## 安装特定版本的docker

如果要安装特定版本的docker-ce，可以使用如下方法。

使用如下命令查询yum源中的docker-ce版本

```
yum list docker-ce --showduplicates | sort -r
Last metadata expiration check: 0:00:27 ago on Sat 09 Apr 2022 12:39:09 AM CST.
docker-ce.x86_64                3:20.10.9-3.el8                 docker-ce-stable
docker-ce.x86_64                3:20.10.8-3.el8                 docker-ce-stable
docker-ce.x86_64                3:20.10.7-3.el8                 docker-ce-stable
docker-ce.x86_64                3:20.10.6-3.el8                 docker-ce-stable
```

选择特定版本的docker-ce和docker-ce-cli，执行如下命令
```
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```


# 安装kubectl kind helm

```
# 安装kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
mv kubectl /usr/bin/
# 如果找不到rpm包，可以通过网络安全 rpm -ivh http://mirror.centos.org/centos/7/os/x86_64/Packages/bash-completion-2.1-8.el7.noarch.rpm
yum install -y bash-completion
echo -e '\n# kubectl' >> ~/.bash_profile
echo 'source <(kubectl completion bash)' >>~/.bash_profile
echo 'alias k=kubectl' >>~/.bash_profile
echo 'complete -F __start_kubectl k' >>~/.bash_profile
source ~/.bash_profile

# 安装helm
wget https://get.helm.sh/helm-v3.7.2-linux-amd64.tar.gz
tar zvxf helm-v3.7.2-linux-amd64.tar.gz
mv linux-amd64/helm /usr/bin/
rm -rf linux-amd64

# 安装kubectx kubens
git clone https://github.com/ahmetb/kubectx /tmp/kubectx
cp /tmp/kubectx/kubens /usr/bin/kns
cp /tmp/kubectx/kubectx /usr/bin/kctx
wget https://github.com/junegunn/fzf/releases/download/0.29.0/fzf-0.29.0-linux_amd64.tar.gz -P /tmp
tar zvxf /tmp/fzf-0.29.0-llinux_amd64.tar.gz -C /tmp/
mv /tmp/fzf /usr/local/bin/

# 安装kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
chmod +x ./kind
mv ./kind /usr/local/bin/
echo -e "\n# kind" >> ~/.bash_profile
echo 'source <(kind completion bash)' >>~/.bash_profile
```

# 创建集群

其中将apiServerAddress指定为了本机，即创建出来的k8s集群仅允许本集群内访问。如果要是需要多个k8s集群之间的互访场景，由于kind拉起的k8s运行在docker容器中，而docker容器使用的是容器网络，此时如果设置apiserver地址为127.0.0.1，那么集群之间就没法直接通讯了，此时需要指定一个可以在docker容器中访问的宿主机ip地址。

```
if=eth0
ip=`ifconfig $if|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:"`
cat > kind.conf <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind
nodes:
- role: control-plane
  image: kindest/node:v1.23.17 # 指定 k8s 版本
networking:
  apiServerAddress: "$ip"
  apiServerPort: 6443
EOF
kind create cluster --config kind.conf
```

# 其他周边工具

```
# 安装kustomize
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
mv kustomize /usr/local/bin/

# 安装golang
wget https://go.dev/dl/go1.17.5.linux-amd64.tar.gz -P /opt
tar zvxf /opt/go1.17.5.linux-amd64.tar.gz -C /opt/
mkdir /opt/gopath
echo -e '\n# golang' >> ~/.bash_profile
echo 'export GOROOT=/opt/go' >> ~/.bash_profile
echo 'export GOPATH=/opt/gopath' >> ~/.bash_profile
echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bash_profile
source ~/.bash_profile

# 安装controller-gen，会将controller-gen命令安装到GOPATH/bin目录下
go install sigs.k8s.io/controller-tools/cmd/controller-gen@latest

# 安装dlv工具
go install github.com/go-delve/delve/cmd/dlv@latest
```

## 安装krew

```
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)

echo -e '\n# kubectl krew' >> ~/.bash_profile
echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
```
