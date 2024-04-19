---
title: 技术分享第14期
date: 2022-02-09 19:35:26
tags:
---

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/zheli.jpg)

题图为一点资讯近期推出的一款定位线上社交的App啫喱，每个人可以给自己订制一个卡通形象，是国内厂商对于线上社交的一次新的尝试。

距离上期技术分享已经约有一年半的时间，这次不定期更新的时间有些久。2022年期望在博客方面增加时间投入，多关注开源技术，预计会大幅度提升更新的频率。

# 资源

## 1. [Katacoda](https://www.katacoda.com/courses/kubernetes/playground)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/katacoda.jpg)

可以一键创建一个k8s集群的工具，甚至无需登录，比上期推荐的工具[Play with Kubernetes](http://kuring.me/post/knowledage-share-13/)更为方便。

## 2. [OperatorHub](operatorhub.io)

k8s推出了CRD的机制后，大大增强了k8s的扩展能力，可以好不夸张的说，k8s之所以如此成功，跟CRD的扩展机制有很大关系。OperatorHub类似于DockerHub，收集了各种各样的operator实现。

## 3. [lazykube](https://github.com/TNK-Studio/lazykube)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/lazykube.gif)

一款可以通过命令行终端来展示和管理k8s资源的工具。

## 4. [LazyDocker](https://github.com/jesseduffield/lazydocker)

![](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/lazydocker.jpg)

同上面的lazykube工具，LazyDocker是一个可以在命令行上查看本机docker的工具，可以查看docker上容器以及运行状态、本地的镜像以及分层信息、volume信息。

## 5. [httpbin](http://httpbin.org)

一个非常简单的http服务，可以用来在测试服务的连通性，尤其是可以用curl测试api层面的连通性，再也不用访问curl http://www.baidu.com了。比如执行`curl http://httpbin.org/headers`可以以json的形式返回request的http header信息，执行`curl http://httpbin.org/status/200`可以返回状态码为200。

不过，在国内访问该网站连通性并不是太好。还提供了镜像版本，可以直接在本地以docker的方式运行该服务。

## 6. [Flux](https://github.com/fluxcd/flux)

一款基于k8s的GitOps工具，采用k8s的声明式api基于operator实现。

![https://github.com/fluxcd/flux/blob/master/docs/_files/flux-cd-diagram.png](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/flux-cd-diagram.png)

## 7. [OpenTelemetry](https://opentelemetry.io)

云原生领域的可观测性工具，用来制定规范、提供sdk和采集agent实现，实现了可观察性中的tracing和metric，并没有实现logging部分。不负责底层的后端存储，可以跟负责metric的prometheus集成，负责tracing的jager集成。

## 8. [kspan](https://github.com/weaveworks-experiments/kspan)

![https://github.com/weaveworks-experiments/kspan/blob/main/example-2pod.png](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/kspan.png)

该工具可以将k8s的event使用OpenTelemetry转换成tracing的形式，并将其存储在支持tracing的后端，比如jaeger中。

## 9. [skopeo](https://github.com/containers/skopeo)

skopeo是一款镜像操作工具，用来解决常用的镜像操作，但这些功能docker命令却不太具备，或者需要调用docker registry的api才可以完成操作。比如镜像从一个镜像仓库迁移到另外一个镜像仓库，从镜像仓库中删除镜像等。

## 10. [KubeOperator](https://fit2cloud.com/kubeoperator/index.html)

![https://camo.githubusercontent.com/1045b27ad1186b915281a6fd77b35979d2038d97e152c34f0431e74ac43d4145/68747470733a2f2f6b7562656f70657261746f722e696f2f696d616765732f73637265656e73686f742f30322e6a7067](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/kube-operator.jpeg)

KubeOperator一个开源的轻量级 Kubernetes 发行版，提供可视化的 Web UI，可以用来在IaaS 平台上自动创建主机，通过 Ansible 完成自动化部署和变更操作。

另外，还提供了商业版本，支持更多的企业级特性，这种模式也是类似Redhat的最为常见的开源软件的商业变现方式。

## 11. [Lens](https://k8slens.dev/)

![https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/kube-operator.jpeg](https://kuring.oss-cn-beijing.aliyuncs.com/knowledge/lens.png)

Kubernetes的客户端工具，非web端工具，可以通过本地访问远程的k8s集群，并且可以获取k8s集群的node、pod等信息，提供了mac、linux、windows三种客户端版本。

## 12. [Caddy](https://caddyserver.com)

一款使用Golang编写的七层负载均衡软件，Github上有3万+的star数量，相比与nginx有两个明显的优势：提供了默认的https服务，支持证书的自签发；使用Golang开发，只需要一个二进制配合caddyfile配置文件即可运行，更轻量。

## 13. [LVScare](https://github.com/sealyun/lvscare)

sealyun开源的一个轻量级的lvs管理工具，可以作为轻量级的负载均衡，基于Golang实现。输入这样一条指令 `lvscare care --run-once --vs 10.103.97.12:6443 --rs 192.168.0.2:6443 --rs 192.168.0.3:6443 --rs 192.168.0.4:6443` 即可在宿主机上创建对应的ipvs规则。自带了健康检查功能，支持四层和七层的健康检查，是该工具的最大优势。一般lvs会配合着keepalived来进行检查检查，一旦健康检查失败后会将rs的权重调整为0，但keepalived的配置相对复杂，该工具更轻量。

## 14. [HUGO](https://gohugo.io/)

非常火爆的使用Golang开发的开源博客系统，目前Github上的Star数量已经超过5万+，更老牌的开源博客系统Hexo的Star数量才3万+，基于Vue开发的VuePress Star数量还不到2万。

## 15. [sealer](https://github.com/alibaba/sealer)

阿里巴巴开源的一款云原生的应用发布工具。该工具的设计思路非常先进，提出了集群镜像的概念，可以将k8s集群和应用build成为一个集群镜像，类似于docker镜像，一旦集群镜像构建完成后即可像容器一样运行在不同的硬件上。
