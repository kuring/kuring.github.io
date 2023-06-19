---
title: Dockerfile中的ENTRYPOINT与CMD
date: 2019-01-04 22:00:56
tags:
---

在Dockerfile中ENTRYPOINT与CMD的功能类似，同时再加上`docker run`后面追加的容器启动参数，是极其容易混淆的。而且又掺杂着exec模式和shell模式。

这里先说几个结论，有了结论再跟进下面的例子来理解会更容易一些：

1. 实际上docker容器进程的完整启动参数为`ENTRYPOINT CMD`，如果没有指定ENTRYPOINT，docker会提供一个隐式的值`/bin/sh -c`。
2. `docker run`后面跟的容器启动参数仅会覆盖CMD部分。

## exec模式与shell模式

CMD和ENTRYPOINT两个命令均支持exec模式和shell模式。

exec模式格式类似`CMD [ "top" ]`，当容器启动时，top命令的进程号为1。

为了能够获取到环境变量，通常的写法为`CMD [ "sh", "-c", "echo $HOME" ]`，此时1号进程为sh。

shell模式的写法为`CMD top`，docker会以`/bin/sh -c top`的方式来执行命令，此时容器的1号进程为sh。

如果需要容器进程处理外部信号的情况下，shell模式下信号实际上时发送给了sh，而不是容器中的应用进程。

因此比较推荐使用exec模式，shell模式实际使用较少。

## CMD

- CMD ["param1", "param2"] 为ENTRYPOINT提供默认参数，需要指定ENTRYPOINT
- CMD ["executable","param1","param2"] exec模式
- CMD command param1 param2 shell模式

CMD为容器提供默认的启动命令，如果在启动容器时通过命令行指定了的启动参数，则该启动参数会覆盖CMD默认的启动参数。

## ENTRYPOINT

不能被`docker run`增加的参数覆盖，启动时要执行ENTRYPOINT的参数。

- ENTRYPOINT ["executable", "param1", "param2"] exec模式
- ENTRYPOINT command param1 param2 shell模式

### exec模式

当为exec模式时，容器启动时，在命令行上添加的参数会被追加到ENTRYPOINT的参数列表中。

例如：

```
FROM ubuntu:latest
ENTRYPOINT [ "echo", "hello" ]
```

执行`docker run --rm 0d89e8d4425a world`，会输出`hello world`

### shell模式

当ENTRYPOINT为shell模式时，docker run启动后追加的参数会被忽略。

例如：

```
FROM ubuntu:latest

ENTRYPOINT echo hello
```

执行`docker run --rm 0841e19b4d2e world`仅输出`hello`。

### ENTRYPOINT命令的覆盖

ENTRYPOINT的命令可以通过`docker run`中增加`--entrypoint`选项来使用命令行中指定的参数覆盖ENTRYPOINT的参数。

## ENTRYPOINT与CMD的组合使用

当同时指定CMD和ENTRYPOINT模式时，实际上为`ENTRYPOINT CMD`

```
FROM ubuntu:latest

ENTRYPOINT [ "echo", "hello" ]
CMD [ "world" ]
```

`docker run --rm 7edf658370d9`会输出`hello world`，而`docker run --rm 7edf658370d9 kitty`会输出`hello kitty`。

更复杂的情况可以参照下图：

![https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact](https://kuring.oss-cn-beijing.aliyuncs.com/images/entrypoint-cmd.png)

## 如何查看ENTRYPOINT和CMD

可以通过`docker history ${image} --no-trunc`来查生成镜像的所有Dockerfile命令

## ref

- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#entrypoint)
- [Dockerfile 中的 CMD 与 ENTRYPOINT](https://www.cnblogs.com/sparkdev/p/8461576.html)
