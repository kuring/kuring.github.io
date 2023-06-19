title: 阿里巴巴开源云原生项目分析（持续更新）
tags: []
categories: []
date: 2022-02-08 21:33:00
author:
---
# [ackdistro](https://github.com/AliyunContainerService/ackdistro)

阿里云的k8s发行版，跟阿里云的ack采用了相同的源码。该项目采用[sealer](#sealer)来部署k8s集群，并通过sealer支持k8s集群的扩容、缩容节点等运维操作。该项目并没有将k8s相关的源码开源，而主要维护了安装k8s集群需要的yaml文件、helm chart。

ack的k8s发行版比较简洁，并没有公有云ack的丰富的组件。除了k8s原生的几个组件外，网络插件集成了[hybridnet](#hybridnet)，存储插件集成了[open-local](#open-local)。

# [hybridnet](https://github.com/alibaba/hybridnet)

容器网络插件，支持underlay网络和overlay网络，且可以支持一个k8s集群内的网络同时支持overlay网络和underlay网络。underlay网络模式下，可以支持vlan网络，也可以支持bgp网络。

# [image-syncer](https://github.com/AliyunContainerService/image-syncer)

镜像仓库同步工具，用于两个镜像仓库之间的数据同步。在公有云的产品ack one中有所应用。

# [kube-eventer](https://github.com/AliyunContainerService/kube-eventer)

![![https://user-images.githubusercontent.com/8912557/117400612-97cf3a00-af35-11eb-90b9-f5dc8e8117b5.png](https://kuring.oss-cn-beijing.aliyuncs.com/common/sealer.png)](https://kuring.oss-cn-beijing.aliyuncs.com/common/kube-eventer.png)

k8s的Event会存放在etcd中，并跟对象关联。Event有一定的有效时常，默认为1小时。业界通常做法是将event存放在单独的etcd集群中，并将event的生命周期设长。对于异常类型的Event，是一种非常理想的告警机制。

kube-eventer组件可以将k8s的event对象收集起来，可以发送到钉钉、kafka等数据sink端。

# [kubernetes-cronhpa-controller](https://github.com/AliyunContainerService/kubernetes-cronhpa-controller)

k8s提供了hpa机制，可以针对pod的监控信息对pod进行扩缩容。该组件提供了定期扩缩容的机制，可以定期对pod的数量进行设置。

相关资料：[容器定时伸缩（CronHPA）](https://help.aliyun.com/document_detail/151557.htm?spm=a2c4g.11186623.0.0.48b151bb3ZWte9#task-2391975)

# [log-pilot](https://github.com/AliyunContainerService/log-pilot)

针对容器类型的服务研发的日志收集agent，可以在k8s pod上通过简单配置环境变量的方式即可采集日志，使用起来非常简洁。

# [open-local](https://github.com/alibaba/open-local)
k8s对于本地磁盘设备的使用相对较弱，提供了emptyDir、hostPath和local pv的能力来使用本地磁盘设备，但这些功能并没有使用到k8s的动态创建pv的功能，即在pod在使用pvc之前，pv必须实现要创建出来。

该项目提供了本地存储设备的动态供给能力，可以将本地的一块完整磁盘作为一个pv来动态创建。也可以将本地的磁盘切分成多块，通过lvm的方式来分配本地的不同pod来使用，以满足磁盘的共享，同时又有完整的磁盘quota能力。

相关资料：[LVM数据卷](https://help.aliyun.com/document_detail/178476.html)


# [sealer](https://github.com/alibaba/sealer)

![https://user-images.githubusercontent.com/8912557/117400612-97cf3a00-af35-11eb-90b9-f5dc8e8117b5.png](https://kuring.oss-cn-beijing.aliyuncs.com/common/sealer.png)

当前的应用发布经历了三个阶段：
- 阶段一 裸部署在物理机或者vm上。直接裸部署在机器上的进程，存在操作系统、软件包的依赖问题，比如要部署一个python应用，那么需要机器上必须要包含对应版本的python运行环境以及应用依赖的所有包。
- 阶段二 通过镜像的方式部署在宿主机上。docker通过镜像的方式将应用以及依赖打包到一起，解决了单个应用的依赖问题。
- 阶段三 通过k8s的方式来标准化部署。在k8s时代，可以将应用直接部署到k8s集群上，将应用的发布标准化，实现应用的跨机器部署。

在阶段三中，应用发布到k8s集群后，应用会对k8s集群有依赖，比如k8s管控组件的配置、使用的网络插件、应用的部署yaml文件，对镜像仓库和dockerd的配置也有所依赖。当前绝大多数应用发布是跟k8s集群部署完全独立的，即先部署k8s集群，然后再部署应用，跟阶段一的发布单机应用模式比较类似，先安装python运行环境，然后再启动应用。

sealer项目是个非常有意思的开源项目，旨在解决k8s集群连同应用发布的自动化问题，可以实现类似docker镜像的方式将整个k8s集群连同应用一起打包成集群镜像，有了集群镜像后既可以标准化的发布到应用到各个地方。sealer深受docker的启发，提出的很多概念跟docker非常类似，支持docker常见的子命令run、inspect、save、load、build、login、push、pull等。

- Kubefile概念跟Dockerfile非常类似，且可以执行sealer build命令打包成集群镜像，语法也类似于Dockerfile。
- CloudImage：集群镜像，将Kubefile打包后的产物，类比与dockerimage。基础集群镜像通常为裸k8s集群，跟docker基础镜像为裸操作系统一致。
- Clusterfile：要想运行CloudImage，需要配合Clusterfile文件来启动，类似于Docker Compose。Clusterfile跟Docker Compose一致，并不是必须的，也可以通过sealer run的方式来启动集群镜像。

sealer要实现上述功能需要实现将k8s集群中的所有用到镜像全部打包到一个集群镜像中。

相关资料：[集群镜像：实现高效的分布式应用交付](https://mp.weixin.qq.com/s/0SBslzaMWtqn9H8Q57urNA)