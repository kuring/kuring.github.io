title: 阿里云容器服务ack技术调研（个人笔记）
date: 2022-03-08 19:54:17
tags:
author:
---
阿里云容器服务是阿里云公有云基于Kubernetes企业级服务，在社区的Kubernetes版本基础上有能力增强， 本文用于调研社区增强功能，记录解决的问题、以及实现方法。ACK集群的增强功能有一部分是基于Kubernetes的api的prodiver实现，另外一部分是基于Kubernetes增加的额外组件，其中很多都已经开源，可以看到[开源软件](https://help.aliyun.com/document_detail/196039.html)列表。

ACK的一些组件列表可以参见：[组件概述](https://help.aliyun.com/document_detail/277412.html)

# 产品形态

- 专有版Kubernetes：master和worker阶段均需要创建
- 托管版Kubernetes：只需要创建worker节点，master节点通过ack托管
- Serverless Kubernetes：master节点和worker节点均不需要自己创建

# 节点管理
| 大类 | 特性 | 描述 |
| --- | --- | --- |
| 节点 | 节点自动扩缩容 | 完全利用k8s的autoscaler实现，提供了白屏的配置功能。[https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md?spm=a2c4g.11186623.0.0.5e09135f2AAa2u&file=FAQ.md](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md?spm=a2c4g.11186623.0.0.5e09135f2AAa2u&file=FAQ.md) |
|  | 节点资源变配 | master和worker节点的资源变配，通过调用ecs的变配规格接口来实现。 |
| 节点池 |  | 将节点进行了分组，同一个分组内的节点可以统一来管理。比如，可以统一设置标签污点、设置期望节点数。通过节点池实现，节点池跟弹性伸缩组为一对一关系。 |

# 弹性伸缩
| 类型 | 特性 | 描述 |
| --- | --- | --- |
| 调度层 | hpa | 基于k8s的hpa功能实现 |
|  | 自定义指标的hpa | 基于[prometheus-adapter](https://github.com/kubernetes-sigs/prometheus-adapter)实现的自定义指标监控 |
|  | vpa | 适用于大型单体应用。k8s的vpa功能实现，基于cluster-autoscaler实现 |
|  | CronHPA | 定期对pod进行伸缩，组件已开源：[kubernetes-cronhpa-controller](https://github.com/AliyunContainerService/kubernetes-cronhpa-controller) |
|  | ElasticWorkload |  |
| 资源层 | 节点自动伸缩 | k8s的node自动伸缩，基于社区的cluster-autoscaler实现 |
|  | 使用ECI弹性调度 | 通过virtual-kubelet实现，将ECI抽象为k8s的node节点 |

# 安全
| 大类 | 特性 | 详细描述 |
| --- | --- | --- |
| 操作系统 | 基于Alibaba Cloud Linux 2支持等保2.0三级加固 |  |
|  | 基于Alibaba Cloud Linux 2支持CIS安全加固 |  |
| 基础设施 | 使用阿里云KMS提供Secret的落盘加密功能 | 在默认情况下，k8s的Secret是基于base64转码后明文存储的，存在一定的安全风险。[k8s提供了使用外部的KMS provider来对Secret的数据进行加密的功能](https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/#ensuring-all-secrets-are-encrypted)，apiserver和provider之间的通讯协议采用[grpc接口](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apiserver/pkg/storage/value/encrypt/envelope/v1beta1/service.proto)。Secret中的数据在经过KMS provider加密后存储在etcd中。ACK借助阿里云的秘钥管理服务（KMS）提供了provider实现，该[provider完全开源](https://github.com/AliyunContainerService/ack-kms-plugin?spm=a2c4g.11186623.0.0.1cf429ecKKZx1J)。 |
|  | 为pod动态配置阿里云产品白名单 | ack-kubernetes-webhook-injector组件会自动将pod ip添加到阿里云产品的白名单中。 |
|  | k8s审计日志白屏化展示 |  |
|  | 安全巡检功能 | 基于CIS Kubernetes基线的实现，用来校验k8s的安全性 |
| 容器 | 配置容器安全策略 | 基于opa实现的策略引擎，可以根据预置的规则对k8s中创建的资源进行校验，校验不通过会返回失败 |
|  | 通过巡检来检查集群中存在的安全隐患的pod | 无 |

# 可观测性
| 大类 | 功能项 | 描述 |
| --- | --- | --- |
| 日志 | 日志采集功能基于logtail实现，可以采集容器日志 | 类似于开源组件log-pilot，仅需要配置环境变量，即可对日志进行收集。也可以通过AliyunLogConfig CR旁路的对日志采集进行配置。 |
| 日志 | coredns日志 | 收集coredns日志 |
| 监控 | 基于arms产品支持应用性能监控 |  |
| 监控 | 基于ahas产品实现的架构感知监控 |  |
| 监控 | node节点异常监控 | npd将节点异常信息产生k8s的event |
| 监控 | k8s event监控 | kube-eventer收集k8s的event |

# 操作系统
Container OS：为容器场景而生的操作系统，操作系统镜像大大精简，提供了安全加固能力，不支持单个软件包的升级，软件包只能跟操作系统一起原子升级。
安全容器katacontainer
# 容器&镜像
| 大类 | 特性 | 描述 |
| --- | --- | --- |
| 镜像 | 容器镜像服务ACR | 使用公有云的容器镜像服务ACR |
|  | 验证容器镜像 | 基于开源组件kritis的kritis-validation-hook组件通过webhook的方式对镜像进行验证，确保镜像安全 |

# 调度
| 大类 | 特性 | 解决问题 | 适用场景 | 具体实现 |
| --- | --- | --- | --- | --- |
| pod调度 | 使用Descheduler组件对Pod进行调度优化 | k8s的调度为静态的，可以确保在pod调度时为最优的，但当集群运行一段时间后，集群中资源水位会发生变化，此处无法保证整个集群是最优的。 |  | 使用社区的开源组件Descheduler对pod进行重新调度 |
| cpu调度 | cpu拓扑感知调度 | k8s的cpu manager特性解决的是pod调度到同一个节点上后，通过cpuset来隔离pod的cpu争抢。但却缺乏集群级别的资源视角，从而无法做到全局最优。该特性解决cpu密集型的pod调度到同一个节点上后的争抢问题，允许不是Guaranteed级别的pod实现cpuset特性。 | cpu敏感型应用 | 通过调度器扩展来实现pod的cpu优化调度。ack-slo-manager agent负责实现每台机器上的绑核策略。 |
|  | cpu Brust策略优化容器性能 | k8s的cpu limit机制会在特定的时间段内将进程可以使用的时间片进行限制。但该特性不太适合一些突发类的应用，会导致这类应用在想要cpu的时间不够用，但不想用的时候却有空闲的时间片。 | cpu突发型应用 | ack通过slo-manager实现cpu brust的优化，该组件会监控容器的cpu throtteld状态，并且动态调整容器中的cgroup cfs quota限制。每个节点上会部署一个resource-controller的组件来动态修改pod的cgroup配置。 |
|  | 动态调整pod的资源上限 | k8s中要想修改pod的limit配置，只能修改pod的yaml，此时一定会导致pod重建。对于想调整pod的limit限制，却又不想重启pod的场景。 |  | 通过一个Cgroups的CR来对pod使用的cpu、内存以及磁盘的上限进行动态调整，但并不会修改pod的yaml配置，因此不会导致pod的重建。每个节点上会部署一个resource-controller的组件来动态修改pod的cgroup配置。 |
|  | 通过控制L3 cache和MBA提高不同优先级任务的隔离能力 | 不同优先级的任务调度到同一台机器上存在L3 Cache和内存带宽的资源争抢的问题 | cpu敏感型应用 | 底层利用Intel的[RDT技术](https://www.intel.cn/content/www/cn/zh/architecture-and-technology/resource-director-technology.html?spm=a2c4g.11186623.0.0.77dc7ccaJFCP94)来实现，该技术可以跟踪和控制同一台机器上同时运行的多个应用程序的共享资源情况，比如L3 Cache、内存带宽。每个节点上会部署一个resource-controller的组件来应用RDT的控制。 |
|  | 控制动态水位提高不同优先级pod的资源利用效率 | 为了充分利用机器资源，通常将不同优先级的pod部署在同一个节点上，pod的优先级往往随着时间段而有所变化。在白天的时候，在线业务pod优先级高；晚上离线任务优先级高。该特定可以调整一个节点上pod可以使用的资源上限。 | 在线业务pod和离线业务pod混部 | 通过ConfigMap来配置一台机器上的在线业务和离线业务可以使用的资源比例。通过Cronjob的方式来修改ConfigMap的配置，从而达到变配的效果。 |
|  | 动态资源超卖 | 一台机器上的pod在设计request和limit的时候总会预留一部分buffer，如果将所有pod的buffer加起来就非常多，从而导致机器的资源利用率很难上去。 | pod预留buffer | 引入了reclaimed资源来解决，未完全理解实现。文档：[动态资源超卖](https://help.aliyun.com/document_detail/412172.html) |
| gpu调度 | GPU拓扑感知调度 |  |  |  |
|  | 共享GPU调度 | GPU核共享 |  |  |
| FPGA调度 | 暂未深入研究 |  |  |  |
| 任务调度 | Gang scheduling | Coscheduling将N个pod调度到M个节点上同时运行，需要部分pod同时启动该批处理任务即可运行。如果允许的部分pod为N时，则退化为Gang Scheduling。该场景要求pod要同时创建。 | 批处理任务 | 基于Scheduling Framework实现的自定义调度器，核心机制是借助了Permit插件的pod延迟绑定功能，等到同一个group下的pod都创建后再调度。参考文章：[支持批任务的Coscheduling/Gang scheduling](https://developer.aliyun.com/article/766275) |
|  | Binpack scheduling | k8s默认的调度策略会将pod优先分配到空闲的节点上，但这样集群中的节点会存在资源碎片化的问题。Binpack调度策略会优先将节点的资源用完。 | 减少资源碎片化，尤其是GPU场景 | 基于Scheduling Framework实现的自定义调度器。参考文章：[支持批任务的Binpack Scheduling](https://developer.aliyun.com/article/770336) |
|  | Capacity Scheduling | k8s支持的namespace ResourceQuota特性可以设置一个namespace下的pod可以使用的资源上限，但不够灵活。比如一个namespace下资源耗尽，另外一个namespace还有额外的资源，但这部分资源却不能给资源耗尽的namespace使用。 | namespace ResourceQuota特性增强 | 使用ElasticQuotaTree的方式来定义每个namespace下可以使用的资源最小值和资源上限，配合Scheduling Framework扩展来实现。 |
| 弹性调度 | ECI弹性调度 | 充分利用ECI的弹性功能，解决k8s的node资源不够灵活的问题。 | 将pod调度到ECI | virtual kubelet技术将ECI抽象为k8s的node，并将pod指向调度到该node。调度到该node的pod最终会在ECI拉起容器。 |
|  | 自定义资源的优先级调度 | k8s的调度器的策略采用固定的算法，会将所有node一视同仁，并不能针对某种类型的节点采用不同的策略。 | 自定义基于node调度策略 | 引入CRD ResourcePolicy用来定义节点调度的优先级。 |
| 负载感知调度 | 负载感知调度 | 在pod调度时，参考node节点历史的负载信息，优先将pod调度到负载较低的节点上，避免出现单个节点负载过高的情况。 | 避免node的资源使用率不均 | 通过调度器扩展实现，pod要开启该特性需要增加特性的annotation。 |

# 网络
| 大类 | 特性 | 描述 |
| --- | --- | --- |
| 容器网络 | terway网络 | 基于ENI实现，支持ipvlan模式，基于ipvlan和eBPF实现。NetworkPolicy基于eBPF实现。 |
|  | terway网络的Hubble组件 | 基于eBPF实现的网络流量可视化 |
|  | 为pod挂载独立公网ip | pod声明annotation k8s.aliyun.com/pod-with-eip:"true" |
| Service网络 | ccm跨集群部署服务 | 同一个vip可以挂载到两个k8s集群内部的Service |
| Ingress | 基于Nginx Ingress实现灰度发布 | 扩展Nginx Ingress实现 |
|  | 通过AHAS支持流控特性 |  |
|  | 基于Nginx Ingress实现流量复制 |  |
|  | 基于ALB实现了ALB Ingress | 数据面直接使用ALB的七层负载均衡功能 |
| DNS | 引入kubernetes项目中的addon组件NodeLocal DNSCache来增加cache层 | [NodeLocal DNS Cache](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/nodelocaldns) nodelocal dns cache位于kubernetes项目中，以DaemonSet的方式运行在k8s集群中 |
|  | ExternalDNS服务 | 用来将DNS注册到外部的公共域名服务器 |
|  | 使用DNSTAP Analyser诊断异常 | s使用CoreDNS DNSTAP Analyser组件来接收coredns的DNS解析报文格式dnstap协议，并最终可以输出到sls对异常的DNS解析报文进行分析。 |

# 存储
存储为ACK的一大亮点，借助云的丰富存储类型，通过k8s的CSI插件机制提供了块存储、文件存储、对象存储OSS和本地存储的支持。

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/220839/1645347493244-2af51b21-0150-48e4-a15e-0585d5a59852.png#clientId=ub8a97252-9d7b-4&from=paste&height=916&id=ufabccb6c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=916&originWidth=2458&originalType=binary&ratio=1&size=143128&status=done&style=none&taskId=uf542f250-6209-4734-8564-306faaaaca5&width=2458)

## 本地存储

1. LVM数据卷功能，基于lvm来动态创建pv
1. QuotaPath，基于ext4的quota特性实现的本地存储的quota隔离功能
1. 内存数据卷
1. 持久化内存技术

## 云盘存储卷

1. 支持云盘的在线扩容，k8s 1.16版本之前可以手工扩容磁盘，但是pvc保持必变。在k8s 1.16之后，在pvc修改后，可以自动完成云盘的扩容。
1. 可根据磁盘的使用水位支持云盘的自动扩容，该功能通过额外的组件storage-operator来实现，策略存放到了额外的CRD StorageAutoScalerPolicy。跟k8s的hpa和vpa功能相比，该功能没有自动缩容的功能。
1. 使用k8s的存储快照功能实现了存储快照。k8s定义了VolumeSnapshotContent（类似pv）、VolumeSnapshot（类似pvc）和VolumeSnapshotClass（类似StorageClass）三个类型来实现打快照功能，快照的恢复则借助pvc的spec.dataSource字段实现。
1. 加密云盘功能，同样借助云上能力实现。使用时，仅需要在StorageClass的parameters参数中指定加密参数即可。

# Serverless Kubernetes（ASK）

适用场景：
1. 业务要求高弹性，如互联网在线服务存在波峰波谷特别明显

ACK提供了k8s以及k8s的管理功能，其中k8s的master节点需要单独创建和维护，每个k8s集群使用的资源是完全独立的。在ASK中，用户仅需要创建k8s集群，而不需要关心k8s集群的核心组件具体是怎么创建的。

在实际上，k8s的核心组件如etcd、kube-apiserver、kube-scheduler、kube-controller-manager可能是以pod的形式运行在另外的k8s集群之上，即所谓的k8s on k8s（KOK）的方案。而且这部分组件是多个k8s集群混部的，对用户完全屏蔽了实现细节，用户仅需要聚焦在如何使用k8s即可。

对于k8s的node节点，用户同样不需要创建，可以认为k8s的节点资源是无限多的。ask采用了virtual kubelet的技术创建了一个虚拟的k8s node节点 `virtual-kubelet-${region}-${zone}`，整个k8s集群仅有一个节点。因为只有一个k8s节点，实际上k8s的管控作用会大大弱化，尤其是kube-scheduler，因为只有一个k8s节点，不存在pod调度的问题。

用户创建的pod资源，实际上会部署在阿里云产品弹性容器实例ECI上。新创建ask集群完成后，再到ECI上即可看到有默认的容器创建出来。ECI了创建容器的功能，给用户暴露的功能是跟k8s无关的，而ask相当于是给用户提供了以k8s的方式来创建容器。

ask还集成了knative，用户可以使用knative的Serving和Eventing的功能来实现社区通用的serverless服务。

相关链接：
- [全新的网关能力增强](https://mp.weixin.qq.com/s/3zxbdvx2LmeAjt74xqhVbw)

# 分布式云容器平台ACK One

ack one提供了两个相对独立的功能，一个是第三方k8s集群的注册，另外一个是k8s多集群的管理和应用发布。两个功能的入口也未统一，其中一个是在ack界面，另外一个是在分布式云容器平台ack one。

第三方k8s集群的注册，允许用户将自己的k8s集群注册到ack上，可以从ack上来管理用户的k8s集群，而且可以部署一些ack自己的组件到用户的k8s集群上面。需要在用户的k8s集群部署一个ack-cluster-agent的服务，在用户的k8s集群跟ack可达的情况下，即可完成用户k8s集群的托管。

k8s多集群的管理功能，底层直接使用了kubevela项目，实现了k8s多集群的管理、多集群的应用发布功能。

# 其他
集群成本分析