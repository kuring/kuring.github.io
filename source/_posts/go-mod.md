---
title: go mod使用
date: 2019-09-18 11:39:13
tags:
---

go mod从1.11开始已经成为了go的默认包管理工具，本文记录go mod的一些使用经验。

要想使用go mod，需要将go升级到1.11或者更高版本。

在没有go mod之前，项目源码必须是放在GOPATH目录下的，有了go mod之后项目即可以放在GOPATH目录下，也可以放在非GOPATH的目录下，在GOPATH目录下在执行时需要指定环境变量`GO111MODULE=on`,具体的写法可以是`GO111MODULE=on go mod init`

由于众所周知的原因，go的包相对还是比较难下载的，很多情况下还是需要vendor目录存在的，并将vendor目录中的包一并提交到代码库中。可以使用`go mod vendor`命令来完成，执行该命令后会将本地下载的包copy到vendor目录下。

## 坑1 提示unknown revision

```
# GO111MODULE=on go get gitlab.aa-inc.com/bb@v2
go: finding gitlab.aa-inc.com/bb v2
go: finding gitlab.aa-inc.com v2
go get gitlab.aa-inc.com/bb@v2: unknown revision v2
```

在获取单个包的时候提示`unknown revision`错误，后发现是go get默认是使用的https协议，而不是git协议，而git仓库的https协议不支持导致的，解决办法为:

```
git config --global url."git@gitlab.aa-inc.com:".insteadOf "https://gitlab.aa-inc.com/"
```

## 参考文档

- [Go 每日漫谈——Go Module 的一些坑](https://zhuanlan.zhihu.com/p/65916369)
