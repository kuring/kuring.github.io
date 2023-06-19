---
title: 通过rsync来绕过relay同步文件
date: 2017-06-07 11:58:54
tags:
---

由于不允许通过ssh直接连接服务器，即服务器的22端口是不开放的，但是其他端口号可以访问。这就造成了往服务器上传输文件会特别麻烦，需要通过relay中转一下。

rsync命令有shell模式和daemon模式，为了解决该问题，可以通过rsync的daemon模式，rysnc的daemon模式会默认使用873端口，不使用ssh协议，以此来绕过ssh的22端口限制。

最终可以实现在本地通过rsync一条命令直接同步文件或文件夹到服务器的指定目录下。

首先在服务器上搭建rsync的服务端，rsync的安装不再介绍。

修改服务器的rsync配置文件/etc/rsyncd.conf如下：

```
[worker]
path = /home/worker
list = true
uid = worker
gid = worker
read only = false
```

这里为了简便，并未设置rsync的用户名和密码。

客户端同步文件的命令如下：

```
rsync -avz $SRC worker@$HOST::worker --exclude=target --exclude=.git --exclude=.idea --delete
```

命令中的第一个worker为HOST的登录用户名，第二个worker为rysncd配置文件中配置的组名。--exclude选项可以用来屏蔽需要同步的文件夹。--delete选项用来同步删除的文件或文件夹。

daemon模式跟ssh模式相比，无法指定服务器的具体某一个路径，使用不够灵活，但也基本可以满足需求。只能通过daemon配置文件中配置的组中的path参数，同步时仅能通过`::组名`的形式来指定。
