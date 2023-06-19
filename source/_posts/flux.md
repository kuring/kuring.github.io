title: Flux（学习笔记）
date: 2022-05-06 23:42:46
tags:
author:
---
# 核心原理
Flux为一款GitOPS的发布工具，应用信息全部放到git仓库中，一旦git仓库中应用信息有新的提交，Flux即可在k8s中发布新的部署。支持kustomize和helm chart形式的应用。目前该项目为CNCF的孵化项目。
# 快速开始
## 前置条件

1. 已经存在k8s集群
2. 可以获取到Github的个人token

## 安装flux cli
执行如下命令安装flux cli
```yaml
curl -s https://fluxcd.io/install.sh | sudo bash
```
将如下内容追到到文件~/.bash_profile
```yaml
. <(flux completion bash)
```
## bootstrap
bootstrap操作会在k8s集群中安装flux到flux-system下，并且会通过git的方式来管理其自身。这里直接采用github，首先要获取到token，并设置为环境变量。
```yaml
export GITHUB_TOKEN=xxx
```
并执行如下命令开始bootstrap
```yaml
flux bootstrap github \
  --owner=kuring \
  --repository=flux-learn \
  --path=clusters/my-cluster \
  --personal
```
会自动创建github repo，并且会在main分支的clusters/my-cluster/flux-system下创建三个文件：
- gotk-components.yaml：flux的组件yaml
- gotk-sync.yaml
- kustomization.yaml：使用kustomization来个性化flux组件

会安装如下组件到k8s中

```yaml
$ k get deployment -n flux-system 
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
helm-controller           1/1     1            1           27m
kustomize-controller      1/1     1            1           27m
notification-controller   1/1     1            1           27m
source-controller         1/1     1            1           27m
```
也可以使用更简单的方式`flux install`来安装flux，该命令不会将flux的安装信息存放到git仓库中，而是会直接安装。

## 部署应用
将上述的github项目copy到本地，需要特别注意的是github不允许使用密码来认证，需要在输入密码的地方输入token。
```yaml
git clone https://github.com/kuring/flux-learn
cd fleet-infra
```
使用如下命令在git仓库中创建podinfo-source.yaml文件和kustomization文件
```yaml
flux create source git podinfo \
  --url=https://github.com/stefanprodan/podinfo \
  --branch=master \
  --interval=30s \
  --export > ./clusters/my-cluster/podinfo-source.yaml
  
flux create kustomization podinfo \
  --target-namespace=default \
  --source=podinfo \
  --path="./kustomize" \
  --prune=true \
  --interval=5m \
  --export > ./clusters/my-cluster/podinfo-kustomization.yaml
```
podinfo-source.yaml文件内容如下：
```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: podinfo
  namespace: flux-system
spec:
  interval: 30s
  ref:
    branch: master
  url: https://github.com/stefanprodan/podinfo
```
podinfo-kustomization.yaml文件内容如下：
```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: podinfo
  namespace: flux-system
spec:
  interval: 5m0s
  path: ./kustomize
  prune: true
  sourceRef:
    kind: GitRepository
    name: podinfo
  targetNamespace: default
```
执行如下命令将文件提供到git仓库
```yaml
git add -A && git commit -m "Add podinfo GitRepository"
git push
```
查看podinfo的部署状态，说明已经自动部署成功。
```yaml
$ flux get kustomizations --watch
NAME    REVISION        SUSPENDED       READY   MESSAGE                          
podinfo master/bf09377  False           True    Applied revision: master/bf09377
flux-system     main/81f2115    False   True    Applied revision: main/81f2115

```
修改podinfo-kustomization.yaml文件内容如下，并重新提交到git仓库
```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: podinfo
  namespace: flux-system
spec:
  interval: 5m0s
  path: ./kustomize
  prune: true
  sourceRef:
    kind: GitRepository
    name: podinfo
  targetNamespace: default
  patches:
    - patch: |-
        apiVersion: autoscaling/v2beta2
        kind: HorizontalPodAutoscaler
        metadata:
          name: podinfo
        spec:
          minReplicas: 3             
      target:
        name: podinfo
        kind: HorizontalPodAutoscaler
```
过会可以看到环境中的hpa最小副本数已经变更为3.

# 资料

- [https://fluxcd.io/](https://fluxcd.io/)
