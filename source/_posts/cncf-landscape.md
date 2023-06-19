title: CNCF Landscape解读
tags: []
categories: []
date: 2022-06-10 00:14:00
author:
---
# 什么是CNCF Landscape？

CNCF（Cloud Native Computing Foundation）为云原生计算基金会的英文缩写，致力于云原生技术的普及和推广。

CNCF Landscape为CNCF的一个重要项目，为了帮助企业和个人开发者快速了解云原生技术的全貌，该项目维护在[GitHub](https://github.com/cncf/landscape)。CNCF Landscape的最重要的两个产出物为路线图和愿景图。


# CNCF Landscape路线图
![](https://raw.githubusercontent.com/cncf/trailmap/master/CNCF_TrailMap_latest.png)

路线图的目的是指导用户使用云原生技术的路径和开源项目。其中包括了10个步骤。


# CNCF Landscape愿景图

将云原生技术进行了分层分类，可以非常清晰的将云原生技术展示给用户。类似于软件架构中，又称“方块图”、“砌砖图”。
![](https://landscape.cncf.io/images/landscape.png)

精简版图形如下：

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/cncf_landscape2.png)

## 供给层（provisioning）

愿景图的第一层，是云原生平台和云原生应用的基础。

### 自动化配置

关键词：IaC（基础设施即代码）、声明式配置

用来加速计算机资源的创建和配置，资源包括虚拟机、网络、防火墙规则、负载均衡等。只需要点击下按钮，即可自动化创建对应的资源，用来降低人工的维护。

典型工具代表：Terraform、Chef、Ansible、Puppet

CNCF项目列表：

| CNCF项目           | 项目阶段 | 项目介绍                 |
| ------------------ | -------- | ------------------------ |
| Akri               | 沙箱     |                          |
| CDK for Kubernetes | 沙箱     |                          |
| Cloud Custodian    | 沙箱     |                          |
| KubeDL             | 沙箱     |                          |
| KubeEdge           | 孵化     | 华为开源的边缘计算项目   |
| Metal3-io          | 沙箱     |                          |
| OpenYurt           | 沙箱     | 阿里云开源的边缘计算项目 |
| SuperEdge          | 沙箱     |                          |
| Tinkerbell         | 沙箱     |                          |

### 容器镜像仓库

用来存储和拉取容器镜像，最简单的项目如docker官方的单机版的docker registry，所有的公有云厂商也都有自己的工具。

CNCF项目包括：

| CNCF项目                       | 项目阶段 | 项目介绍                                                     |
| ------------------------------ | -------- | ------------------------------------------------------------ |
| [Harbor](https://goharbor.io/) | 毕业     | 由vmware打造的镜像仓库，提供了很多企业级的特性，广受企业内部使用。 |
| [Dragonfly](https://d7y.io/)   | 孵化     | 阿里开源的基于P2P技术做镜像分发项目，在大规模集群的场景下效果会比较明显。 |

### 安全合规

关键词：镜像扫描、镜像签名、策略管理、审计、证书管理、代码扫描、漏洞扫描、网络层安全

用来监控、增强应用和平台的安全性，包括从容器到k8s的运行时环境均会涉及到。

CNCF项目包括：

| CNCF项目                                                     | 项目阶段 | 项目介绍                                                     |
| ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| [cert-manager]([cert-manager](https://cert-manager.io/))     | 沙箱     | 证书签发工具，部署在k8s之上，通过抽象CRD Certificate来管理证书，也可以用来管理Ingress的证书 |
| Confidential Containers                                      | 沙箱     |                                                              |
| Curiefense                                                   | 沙箱     |                                                              |
| Dex                                                          | 沙箱     |                                                              |
| Falco                                                        | 孵化     |                                                              |
| in-toto                                                      | 孵化     |                                                              |
| Keylime                                                      | 沙箱     |                                                              |
| Kyverno                                                      | 沙箱     | 基于k8s的策略引擎工具，通过抽象CRD ClusterPolicy的方式来声明策略，在运行时通过webhook的技术来执行策略。相比于opa & gatekeeper，更加k8s化，但却没有编程语言的灵活性。 |
| Notary                                                       | 孵化     |                                                              |
| Open Policy Agent (OPA)                                      | 毕业     | 基于Rego语言的策略引擎，编程能力非常强大                     |
| Parsec                                                       | 沙箱     |                                                              |
| [The Update Framework (TUF)]([The Update Framework](https://theupdateframework.io/)) | 毕业     |                                                              |

### 秘钥和身份管理

关键词：秘钥、身份、Secret、访问控制、认证、授权

CNCF项目包括：

| CNCF项目 | 项目阶段 | 项目介绍 |
| -------- | -------- | -------- |
| Athenz   | 沙箱     |          |
| SPIFFE   | 孵化     |          |
| SPIRE    | 孵化     |          |
| Teller   | 沙箱     |          |

## 运行时层（Runtime）

### 云原生存储

关键词：PV、CSI、备份和恢复

云原生架构下，存储类的工具主要涉及到如下几个方面：

1. 为容器提供云原生的存储。由于容器具有灵活、弹性的特点，云原生的存储相比传统存储会更复杂。
2. 需要有统一的接口。这块基本都会使用k8s的CSI接口。另外，minio提供了S3协议的接口。
3. 备份和还原功能。例如Velero可以用来备份k8s本身和容器使用到的存储。

CNCF项目包括：

| CNCF项目          | 项目阶段 | 项目介绍 |
| ----------------- | -------- | -------- |
| CubeFS            | 孵化     |          |
| K8up              | 沙箱     |          |
| Longhorn          | 孵化     |          |
| OpenEBS           | 沙箱     |          |
| ORAS              | 沙箱     |          |
| Piraeus Datastore | 沙箱     |          |
| Rook              | 毕业     |          |
| Vineyard          | 沙箱     |          |

### 容器运行时

容器运行时的三个主要特征：标准化、安全、隔离性。Containerd和CRI-O为容器运行时的标准实现方案，业界类似KataContainer的方式为将VM作为容器运行时，gVisor方案则在OS和容器中间增加了额外的安全层。

发展趋势：

1. 基于 MicroVM 的安全容器技术，通过虚拟化和容器技术的结合，可以提升更高的安全性。比如 KataContainer。
2. 操作系统的虚拟化程度进一步增加。Linux 4.5 版本的 CGroup V2 技术逐渐成熟，进一步提升了容器的隔离能力。Docker 提供了 rootless 技术，可以以非 root 用户运行，提升了容器的安全性。
3. WebAssembly技术作为跨平台的容器技术可能会作为新的挑战者出现。

| CNCF项目                            | 项目阶段 | 项目介绍                                                     |
| ----------------------------------- | -------- | ------------------------------------------------------------ |
| Containerd                          | 毕业     |                                                              |
| [CRI-O]([cri-o](https://cri-o.io/)) | 孵化     | k8s的CRI的轻量级实现，支持runc和kata作为容器运行时           |
| Inclavare Containers                | 沙箱     |                                                              |
| rkt                                 | 归档     | CoreOS公司主导研发的容器引擎，昔日的Docker竞争对手，目前已经没落，已经被CNCF归档 |
| WasmEdge Runtime                    | 沙箱     |                                                              |

### 云原生网络

关键词：SDN、CNI、Overlay网络

云原生网络具体指的是容器网络，k8s定义了CNI的网络规范，开源项目只需要实现CNI的网络规范即可，社区有非常多的CNI项目可以选择，比如Calico、Flannel、Weave Net等。网络分为Underlay和Overlay网络，其中容器网络为SDN网络，以overlay网络居多。

| CNCF项目                                                     | 项目阶段 | 项目介绍                                                     |
| ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| Antrea                                                       | 沙箱     |                                                              |
| Cilium                                                       | 孵化     |                                                              |
| [CNI-Genie](https://github.com/cni-genie/CNI-Genie)          | 沙箱     | 华为云开源的多网络平面的项目，该项目并非具体的网络插件实现，而是k8s的CNI和具体网络插件实现的中间层，可以实现同一个节点上有多种网络插件的实现，支持同一个pod中有多个网卡。 |
| [CNI（Container Network Interface）](https://github.com/containernetworking/cni) | 孵化     | k8s的容器网络规范，指的一提的是在[plugins]([containernetworking/plugins: Some reference and example networking plugins, maintained by the CNI team. (github.com)](https://github.com/containernetworking/plugins))项目中，提供了很多k8s内置的简单容器插件，比如macvlan、bandwidth等 |
| Kube-OVN                                                     | 沙箱     |                                                              |
| Network Service Mesh                                         | 沙箱     |                                                              |
| Submariner                                                   | 沙箱     | Rancher公司开源的项目，用来解决k8s的多集群场景下的跨集群互通问题 |

## 编排和管理层（Orchestration & Management）

### 调度和编排

在单机的系统中，操作系统会来调度系统中运行的所有的进程，允许某个时间点调度某个进程到某个cpu上面。在集群的环境中，同样需要调度容器在某个时间点运行在某台主机的某个cpu上。

在云原生社区基本上形成以k8s为生态的调度和编排，主要的发展方向为：

- 扩展k8s自身的功能。
- k8s的多集群方向。

| CNCF项目                | 项目阶段 | 项目介绍                                                     |
| ----------------------- | -------- | ------------------------------------------------------------ |
| Crossplane              | 孵化     |                                                              |
| Fluid                   | 沙箱     |                                                              |
| Karmada                 | 沙箱     | 华为云开源的k8s多集群管理项目，用来管理多个k8s集群。自己实现了一套完整的apiserver、scheduler、controller-manager，用来多k8s集群的调度。 |
| kube-rs                 | 沙箱     |                                                              |
| Kubernetes              | 毕业     | CNCF的最重要项目，在容器编排领域具有绝对的统治地位。俗称“现代数据中心的操作系统” |
| Open Cluster Management | 沙箱     | Redhat主导的k8s多集群项目                                    |
| Volcano                 | 孵化     | 华为云开源的基于k8s的容器批量计算平台，常用于大数据、AI领域。k8s默认的Job设计较为简单，无法满足很多批处理场景。Volcano通过CRD扩展的方式定义了Queue、PodGroup、VolcanoJob等实现对批处理作业的抽象，并通过调度器扩展的方式来大幅提升pod的调度效率。 |
| wasmCloud               | 沙箱     |                                                              |

### 调协和服务发现

该领域主要包含两类工具：

1. 服务发现引擎。
2. 域名解析服务。比如CoreDNS。

| CNCF项目 | 项目阶段 | 项目介绍 |
| -------- | -------- | -------- |
| CoreDNS  | 毕业     |          |
| etcd     | 毕业     |          |
| k8gb     | 沙箱     |          |

### 远程过程调用（RPC）

用于进程间通讯的框架，主要解决的问题：

1. 提供了框架，使开发者编码更简单。
2. 提供了结构化的通讯协议。

| CNCF项目 | 项目阶段 | 项目介绍                  |
| -------- | -------- | ------------------------- |
| gRPC     | 孵化     | 业界使用较为广泛的RPC框架 |

### 服务代理

通常又称为负载均衡，从协议上来划分，可以分为四层负载均衡和七层负载均衡。

| CNCF项目 | 项目阶段 | 项目介绍 |
| -------- | -------- | -------- |
| BFE      | 沙箱     |          |
| Contour  | 孵化     |          |
| Envoy    | 毕业     |          |
| OpenELB  | 沙箱     |          |

### API网关

相比于七层负载均衡，API网关还提供了更多高级特性，比如认证、鉴权、限流等。

| CNCF项目         | 项目阶段 | 项目介绍 |
| ---------------- | -------- | -------- |
| Emissary-Ingress | 孵化     |          |

### 服务网格

值得注意的是，业内最为流行的istio项目并不在该范围内。

| CNCF项目                 | 项目阶段 | 项目介绍           |
| ------------------------ | -------- | ------------------ |
| Kuma                     | 沙箱     |                    |
| Linkerd                  | 毕业     | 流行的服务网格项目 |
| Meshery                  | 沙箱     |                    |
| Open Service Mesh        | 沙箱     |                    |
| Service Mesh Interface   | 沙箱     |                    |
| Service Mesh Performance | 沙箱     |                    |

## 应用定义和应用部署

### 数据库

| CNCF项目                        | 项目阶段 | 项目介绍                                                     |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| SchemaHero                      | 沙箱     |                                                              |
| TiKV                            | 毕业     | 国内公司PingCAP开源的分布式kv数据库                          |
| [Vitess](https://vitess.io/zh/) | 毕业     | 用来扩展mysql集群的数据库解决方案，突破单mysql集群的性能瓶颈 |

### 流式计算和消息

| CNCF项目                               | 项目阶段 | 项目介绍                               |
| -------------------------------------- | -------- | -------------------------------------- |
| [CloudEvents](https://cloudevents.io/) | 孵化     | 仅描述了事件的数据规范，并非具体的实现 |
| NATS                                   | 孵化     |                                        |
| Pravega                                | 沙箱     |                                        |
| Strimzi                                | 沙箱     |                                        |
| Tremor                                 | 沙箱     |                                        |

### 应用定义和镜像构建

| CNCF项目                                                     | 项目阶段 | 项目介绍                                                     |
| ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| Artifact Hub                                                 | 沙箱     |                                                              |
| Backstage                                                    | 孵化     |                                                              |
| Buildpacks                                                   | 孵化     |                                                              |
| Devfile                                                      | 沙箱     |                                                              |
| Helm                                                         | 毕业     | k8s的应用打包工具                                            |
| Krator                                                       | 沙箱     |                                                              |
| KubeVela                                                     | 沙箱     | 阿里云开源的基于OAM的应用模型的实现，用来做应用的发布，同时支持多集群 |
| KubeVirt                                                     | 孵化     |                                                              |
| KUDO                                                         | 沙箱     |                                                              |
| Nocalhost                                                    | 沙箱     |                                                              |
| [Operator Framework]([https://operatorframework.io](https://operatorframework.io/)) | 孵化     | 用来开发基于k8s CRD的operator框架，功能跟k8s亲生的kubebuilder非常相似 |
| Porter                                                       |          |                                                              |
| sealer                                                       | 沙箱     | 阿里云开源的集群部署工具，理念比较先进，通过升维的方式可以通过类似Dockerfile的方式来构建集群镜像，并可以通过类似docker run的方式一键拉起完成的一套基于k8s的集群环境 |
| Serverless Workflow                                          |          |                                                              |
| Telepresence                                                 |          |                                                              |

### 持续集成和持续交付

| CNCF项目                                | 项目阶段 | 项目介绍                                                     |
| --------------------------------------- | -------- | ------------------------------------------------------------ |
| Argo                                    | 孵化     | k8s上应用广泛的工作流引擎                                    |
| Brigade                                 | 沙箱     |                                                              |
| Flux                                    | 孵化     | gitops工具                                                   |
| Keptn                                   | 孵化     |                                                              |
| OpenGitOps                              | 沙箱     |                                                              |
| [OpenKruise](https://openkruise.io/zh/) | 沙箱     | 基于k8s能力扩展的组件，通过CRD的方式定义了很多对象，用来增强k8s的workload能力。该组件放到该领域下有些不合适。 |

## 可观测和分析

### 监控

| CNCF项目                                                     | 项目阶段 | 项目介绍                                                     |
| ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| Cortex                                                       | 孵化     |                                                              |
| Fonio                                                        | 沙箱     |                                                              |
| Kuberhealthy                                                 | 沙箱     | k8s的巡检工具，用来检查k8s的健康状态。支持以插件的方式接入巡检脚本。 |
| [OpenMetrics](https://github.com/OpenObservability/OpenMetrics/blob/main/specification/OpenMetrics.md) | 沙箱     | 从Prometheus项目中发展出来的监控数据格式标准，该项目仅定义标准，非实现。 |
| Pixie                                                        | 沙箱     |                                                              |
| Prometheus                                                   | 毕业     | 云原生领域事实上的监控标准                                   |
| Skooner                                                      | 沙箱     |                                                              |
| Thanos                                                       | 孵化     | prometheus的集群化方案                                       |
| Trickster                                                    | 沙箱     |                                                              |

### 日志

| CNCF项目                            | 项目阶段 | 项目介绍     |
| ----------------------------------- | -------- | ------------ |
| [Fluentd](https://www.fluentd.org/) | 毕业     | 日志收集工具 |

### 分布式会话跟踪

| CNCF项目                                   | 项目阶段 | 项目介绍                                                     |
| ------------------------------------------ | -------- | ------------------------------------------------------------ |
| Jaeger                                     | 毕业     | User开源的完整的分布式会话跟踪项目                           |
| [OpenTelemetry](https://opentelemetry.io/) | 孵化     | 同时集成了监控、日志和分布式会话跟踪三个领域的数据收集工具，大有一统可观察性领域的趋势。 |
| [OpenTracing](https://opentracing.io/)     | 归档     | 已经完全被OpenTelemetry取代                                  |

### 混沌引擎

| CNCF项目   | 项目阶段 | 项目介绍 |
| ---------- | -------- | -------- |
| Chaos Mesh | 孵化     |          |
| Chaosblade | 沙箱     |          |
| Litmus     | 沙箱     |          |

# 参考资料

- [Cloud Native Landscape](https://landscape.cncf.io/)
- [云原生全景图详解系列（一）：带你了解云原生技术图谱](https://mp.weixin.qq.com/s/RPuzsVOyFGtqPdcqZqF_1Q)
- [云原生全景图详解系列（二）：供应层](https://mp.weixin.qq.com/s/NrW9-cJ1Lg-VF0WK8kPb7w)
- [云原生全景图详解系列（三）：运行时层](https://mp.weixin.qq.com/s/Q3USUso_PqeTrpNW8mIDhA)
- [云原生全景图详解系列（四）：编排和管理层](https://mp.weixin.qq.com/s/2G4uvqo9_mEbRPh9MfnUmw)
- [云原生全景图详解系列（五）：应用程序定义和开发层](https://mp.weixin.qq.com/s/Zkj0jQC1QOXFLkhCc0zlug)
- [云原生全景图详解（六）｜托管 Kubernetes 和 PaaS 解决什么问题](https://mp.weixin.qq.com/s/73AjqjDG7VuFjtYgCBhHDg)
- [云原生全景图详解（七）：可观察性是什么，有哪些相关工具](https://mp.weixin.qq.com/s/vJwqf9S8l9KpymmAAWACxw)
