title: docker镜像
date: 2022-06-16 22:03:09
tags:
author:
---
## 多架构镜像

### 查看镜像的多架构信息

可以使用 `docker manifest inspect $image` 命令来查看，manifest为docker的体验特性，在Linux系统下开启，需要在本地创建 ~/.docker/config.json 文件，内容如下：

```
{
    "experimental": "enabled"
}
```

最好的方式为开启docker daemon的特性，修改 /etc/docker/daemon.json 文件：

```json
{
  "experimental": true
}
```

例如执行 `docker manifest inspect golang:alpine` 可以看到golang 官方的docker镜像包含了多架构信息，每个架构下会对应一个sha256值。

```
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
   "manifests": [
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:725f8fd50191209a4c4a00def1d93c4193c4d0a1c2900139daf8f742480f3367",
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:5adcff3a3e757841a6c7b07f1986b2a36cb0afaf47025e78bb17358eda2d541a",
         "platform": {
            "architecture": "arm",
            "os": "linux",
            "variant": "v6"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:84a1e4174b934fbf8f1dfe9f7353a5be449096b6f2273d6af5a364ffd6bf8f15",
         "platform": {
            "architecture": "arm",
            "os": "linux",
            "variant": "v7"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:86cfea5046e196f5061324c93f25ef05e1df58ba96721e0c0b42cc6e0cf22e49",
         "platform": {
            "architecture": "arm64",
            "os": "linux",
            "variant": "v8"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:5ad072476cb8b51dddaf4142789f1528c7d48a3a0c31941a5ce21177c8e47259",
         "platform": {
            "architecture": "386",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:fca6cbe2f1fb9095eac2669c0be58b482135f9cf7196d51ac7338ea3e7c556c7",
         "platform": {
            "architecture": "ppc64le",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1365,
         "digest": "sha256:3f7ac24ca4b3ce61b51439cb59b57a8151ba60bd73a0e33cc06020dda6b692cb",
         "platform": {
            "architecture": "s390x",
            "os": "linux"
         }
      }
   ]
}
```

gcr.io 可以在 console 上直接看到信息，比如: [nginx镜像](https://console.cloud.google.com/gcr/images/k8s-artifacts-prod/us/ingress-nginx%2Fnginx@sha256:1ef404b5e8741fe49605a1f40c3fdd8ef657aecdb9526ea979d1672eeabd0cd9/details?tab=pull)

### 多架构镜像的构建

可以使用docker buildx命令，比如 `docker buildx build -t <image-name> --platform=linux/arm64,linux/amd64 . --push` 可以同时构建出arm64和amd64的镜像。



## 查看镜像的构建历史

可以使用 `docker history --no-trunc ${image}` 来查看镜像的每层构建命令

## 通过代理拉取镜像

创建或者修改/etc/docker/daemon.json文件，文件内容如下：

````
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
````

重启docker后通过`docker info`命令查看输出结果：

```
$ docker info
 Registry Mirrors:
  https://hub-mirror.c.163.com/
  https://mirror.baidubce.com/
 Live Restore Enabled: false
```

## 参考资料

- [Image Manifest V 2, Schema 2](https://docs.docker.com/registry/spec/manifest-v2-2/)