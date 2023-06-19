title: k8s api
date: 2022-03-13 21:06:23
tags:
author:
---
## ectd中的数据存储

etcd中存储的k8s对象，key的格式为：/registry/${k8s对象}/${namespace}/${name}，value为k8s的资源对象。apiserver启动的时候通过--storage-media-type参数指定来指定value的格式，默认为application/vnd.kubernetes.protobuf，即protobuf格式。

## 无损转换

在etcd中仅存储了资源的一个特定版本，如果该资源存在多个版本，如果客户端请求的资源跟etcd中存储的资源版本并不一致，那么apiserver怎么保证兼容性呢？

apiserver维护了一个internal版本，版本之间做转换时，源版本首先转换为internal版本，再由internal版本转换为目标版本。

如果版本A要转换为版本B，存在如下几种情况：
1. A和B中均存在的字段，可以之间做转换。
2. A存在B不存在的字段，将字段写入到版本B的annotation中。
3. A不存在而B存在的字段，如果存在于A的注解中，则直接从注解中读取该字段，否则将该字段置为空。