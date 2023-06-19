---
title: nsenter的用法
date: 2018-11-15 23:51:53
tags:
---

nsenter是一个命令行工具，用来进入到进程的linux namespace中。

docker提供了exec命令可以进入到容器中，nsenter具有跟`docker exec`差不多的执行效果，但是更底层，特别是docker daemon进程异常的时候，nsenter的作用就显示出来了，因此可以用于排查线上的docker问题。

CentOS用户可以直接使用`yum install util-linux`来进行安装。

启动要进入的容器：`docker run -d ubuntu /bin/bash -c "sleep 1000"`

获取容器的pid可以使用`

要进入容器执行如下命令：

```
# 获取容器的pid
docker inspect 9f7f7a7f0f26 -f '{{.State.Pid}}'
# 进入pid对应的namespace
sudo nsenter --target $PID --mount --uts --ipc --net --pid
```

## ref

- [nsenter man page](http://man7.org/linux/man-pages/man1/nsenter.1.html)
- [nsenter GitHub](https://github.com/jpetazzo/nsenter)
