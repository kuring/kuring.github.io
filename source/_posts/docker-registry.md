---
title: docker私有仓库搭建
date: 2017-12-07 18:44:59
tags:
---

为了其他主机可访问docker registry，必须采用https协议。

```
mkdir -p ~/docker_registry/certs
signdomain=103-17-184-lg-201-k08
openssl req -nodes -subj "/C=CN/ST=BeiJing/L=BeiJing/CN=$signdomain" -newkey rsa:4096 -keyout ~/docker_registry/certs/$signdomain.key -out ~/docker_registry/certs/$signdomain.csr
openssl x509 -req -days 3650 -in ~/docker_registry/certs/$signdomain.csr -signkey ~/docker_registry/certs/$signdomain.key -out ~/docker_registry/certs/$signdomain.crt
```

从docker hub拉取registry镜像，并启动镜像

```
docker run -d -p 5000:5000 --restart=always --name registry \
  -v /data/docker_registry:/var/lib/registry \
  -v /home/worker/docker_registry/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/103-17-184-lg-201-k08.yidian.com.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/103-17-184-lg-201-k08.yidian.com.key \
  registry:2
```

停止registry镜像并删除的命令为

```
docker stop registry && docker rm -v registry
```

下载最新的centos7镜像

```
 docker pull centos:7.3.1611
```

将centos7镜像增加tag

```
docker tag centos:7.3.1611 103-17-184-lg-201-k08.yidian.com:5000/centos:7.3

# 可以看到列表中会多出一个镜像
[root@103-17-184-lg-201-k08 data]# docker images
REPOSITORY                                     TAG                 IMAGE ID            CREATED             SIZE
docker.io/registry                             2                   047218491f8c        4 weeks ago         33.17 MB
103-17-184-lg-201-k08.yidian.com:5000/centos   7.3                 67591570dd29        3 months ago        191.8 MB
docker.io/centos                               7.3.1611            67591570dd29        3 months ago        191.8 MB
```

docker push命令仅支持https协议，签名已经启动了自签名的https协议的registry，为了能够让docker能够信任registry，需要在/etc/docker/certs.d/目录下增加相应的crt文件，增加后的目录结构为/etc/docker/certs.d/103-17-184-lg-201-k08.yidian.com:5000/103-17-184-lg-201-k08.yidian.com.crt，添加完成后需要重启docker服务。

将image push到registry

```
docker push 103-17-184-lg-201-k08.yidian.com:5000/centos:7.3 
```

# api

* 列出images：https://10.103.17.184:5000/v2/_catalog
* 列出image的tags：https://10.103.17.184:5000/v2/centos/tags/list

可以直接通过curl命令来访问api：`curl --cacert 103-17-184-lg-201-k08.yidian.com.crt -v https://103-17-184-lg-201-k08.yidian.com:5000/v2`

# ref

* [registry](https://hub.docker.com/_/registry/)
* [docker创建私有仓库](http://www.cnblogs.com/fengzheng/p/5168951.html)
* [Docker Registry](https://docs.docker.com/registry/)（官方教程）
* [Registry API](https://docs.docker.com/registry/spec/api/)
