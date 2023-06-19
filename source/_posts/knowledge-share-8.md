---
title: 知识分享第8期
date: 2018-12-20 21:51:10
tags:
---


![](https://kuring.oss-cn-beijing.aliyuncs.com/images/railway-museum.jpeg)

题图为中国铁道博物馆东郊馆中的毛泽东号列车

## 资源

1.[Hawkular](https://www.hawkular.org/)

Hawkular为RedHat开源的监控解决方案，实现语言为java，监控数据的底层存储引擎使用Cassandra，包含了告警功能。目前Github上的Star还较少。RedHat的OpenShift就使用了该监控方案。

2.[Kong](https://konghq.com/)

基于Nginx OpenResty的API网关，支持自定义插件，支持比原生nginx更多的功能。

3.[NuoDB](https://www.nuodb.com/)

<div align=center><img src="https://www.nuodb.com/sites/all/themes/nuodb/logo.svg" width="250" align=center /></div>

弹性可伸缩的关系型数据库，兼容SQL标准。将数据库中的事务和存储进行了分离，存储层支持多种存储系统，比如文件系统、Amazon S3和HDFS。因为存储层可以是外部的存储，意味着NuoDB的扩展性会大大增强，使其部署到Kubernetes成为了比较容易的事情。

4.Linux命令hping3

hping3是一个用于生成和解析tcp/ip协议的工具，能够对数据包进行定制，可用于端口扫描、DDOS攻击等，是一个比较常见的黑客工具。

5.[Firecracker](https://firecracker-microvm.github.io/)

<div align=center><img src="https://firecracker-microvm.github.io/img/logo-icon@3x.png" width="100" align=center /></div>

Amazon开源的轻量级的虚拟机软件，使用KVM来创建和管理虚拟机，整体架构类似Kata Container。容器采用cgroup和namespace来做资源隔离，但是在安全性方面却比较差，轻量级的虚拟机在做到隔离性的同时，又提供了不错的启动速度，是容器领域的一个发展方向。

6.[NginxConfig.io](https://nginxconfig.io/)

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/nginxconfig.png)

NginxConfig.io是一款在线生成nginx配置文件的工具，可以通过点点鼠标，在文本框中内容的方式轻松生成nginx的配置文件。

7.[Caddy](https://github.com/mholt/caddy)

一款实用Go语言编写的负载均衡工具，默认启用HTTPS服务，可以使用Let's Encrypt来自动签发证书。配置文件的写法也比nginx要简洁。

8.[loki](https://github.com/grafana/loki)

Grafana团队最新发布的基于Go语言开发的日志聚合系统，loki不会对日志进行全文索引，而是以压缩聚合的方式进行存储，可以对日志流通过打标签的方式进行分组，页面的展示直接使用grafana。对Kubernetes Pod中的log做了特别的支持，比较适合抓取和存储Kubernetes Pod中的log。

个人感觉该工具未来会很火爆，尤其是跟Grafana有着无缝的整合。很多公司会使用ES来作为日志中心的底层存储，但不见得所有的服务都有按照关键字进行匹配搜索的需求，ES作为日志中心就显得不够高效和经济。

9.[JSON-RPC](https://www.jsonrpc.org/specification)

json-rpc是rpc通讯中的一种json格式标准，该协议要求request和response的内容必须为json格式，且json有固定的格式。

10.[KSQL](https://github.com/confluentinc/ksql)

Apache Kafka的开源SQL引擎，可以使用SQL的形式查询kafka中的消息，该产品跟Kafka一样，同样为Confluent出品。

## 精彩文章

1.[北京五环外的真实中国](https://mp.weixin.qq.com/s/9XyayHJ_m_9Q-igED6jRRg)

朋友圈刷屏文章，文章以gif动画的形式描述了社会底层人士的艰辛生活，他们背上扛起的不仅是压得直不起腰来的砖头，而是面对困难努力生活的勇气，有些时候为了生计确实没得选择。

当我们在抱怨生活的同时，可以想想比我们更苦更累却默默承受生活之重的人们，或许心里会好受些。

## 书籍

1.[《深入解析Go》](https://tiancaiamao.gitbooks.io/go-internals/zh/)

从底层角度分析go语言实现，推荐所有golang开发者一看。

2.[深入浅出Serverless：技术原理与应用实践](http://product.china-pub.com/8054378)

![http://images.china-pub.com/ebook8050001-8055000/8054378/zcover.jpg](https://kuring.oss-cn-beijing.aliyuncs.com/images/serverless-zcover.jpg)

要想能够对Serverless技术的概念和现状有所了解，该书还是挺合适的。

该书介绍了公有云上的Serverless产品AWS Lambda、Azure Functions，开源项目OpenWhisk、Kubeless、Fission和OpenFasS，提供对这些技术的一站式了解。
