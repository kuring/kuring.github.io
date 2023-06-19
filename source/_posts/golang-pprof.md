---
title: golang使用pprof分析程序性能瓶颈
date: 2018-11-16 22:07:49
tags:
---

golang中的pprof工具可以分析系统问题，开启pprof功能非常简单，即在import中增加`_ "net/http/pprof"`的导入即可，然后通过http调用`/debug/pprof`接口即可在web界面上看到pprof的相关信息。

golang 1.11版本中已经自带了火焰图功能，火焰图为性能分析的利器，可以快速找到程序性能的瓶颈。不再需要使用[go-torch](https://github.com/uber/go-torch)项目。

查看火焰图需要用到Graphviz工具，该工具需要单独安装。

Graphviz工具运行的服务器系统为CentOS，使用下载[源码包](https://graphviz.gitlab.io/pub/graphviz/stable/SOURCES/graphviz.tar.gz)的方式进行安装。

依次执行下面命令：

```shell
./configure
make
make install
```

## 例子

本文的使用环境：服务器程序为transfer，运行在linux系统中。执行`go tool pprof`命令运行在linux服务器10.103.17.184上。

程序运行后调用程序的`/debug/pprof/profile`http接口，可获取到cpu profile数据文件。例如，在服务器上可以执行运行`wget http://10.103.34.138:3300/debug/pprof/profile`命令，将profile文件下载到另外一台分析的服务器上。

在分析服务器上执行`go tool pprof -http="10.103.17.184:20000" transfer profile`

在浏览器中打开`10.103.17.184:20000`即可得到性能分析的结果。

![调用关系图](https://kuring.oss-cn-beijing.aliyuncs.com/images/pprof_graph.png)

![火焰图](https://kuring.oss-cn-beijing.aliyuncs.com/images/pprof_flame.png)

同样也可以在本机访问远程的程序暴露的pprof数据，使用命令如：go tool pprof -http :9090  http://10.66.161.43:10245/debug/pprof/heap
