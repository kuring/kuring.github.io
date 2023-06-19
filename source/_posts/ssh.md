title: ssh协议
tags:
  - 效率
categories: []
date: 2022-03-12 17:55:00
author:
---
## ssh命令常用参数

指定私钥连接：`ssh -i ~/.ssh/id_rsa ido@192.168.1.111 -p 7744`

## 免密登录

用来两台主机之间的ssh免密操作，步骤比较简单，主要实现如下两个操作：
1. 生成公钥和私钥
2. 将公钥copy到要免密登录的服务器


### 生成公钥和私钥

执行 `ssh-keygen -b 4096 -t rsa` 即可在 ~/.ssh/目录下生成两个文件id_rsa和id_rsa.pub，其中id_rsa为私钥文件，id_rsa.pub为公钥文件。

### 将公钥copy到要免密登录的服务器

执行 `ssh-copy-id $user@$ip` 即可将本地的公钥文件放到放到要免密登录服务器的 $HOME/.ssh/authorized_keys 文件中。至此，免密登录的配置就完成了。


## ssh隧道

动态端口转发：执行 `ssh root@xxxx -ND 127.0.0.1:1080` 即可在本机的1080端口开启一个ssh隧道。


## rsync

rsync的常用命令：

```
rsync -avzP --delete $local_idr  $user@$remote:$remote_dir
```
