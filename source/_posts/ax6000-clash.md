title: 红米路由器 AX6000 科学上网
date: 2024-02-11 01:12:10
tags:
author:
---
在上篇文章《红米路由器 AX6000 解锁 SSH》中，解锁了红米路由器 AX6000 的 SSH 功能，本文在此基础上安装 clash，用来科学上网。
# 安装 clash
通过 ssh 命令连接到红米路由器，执行如下 shell 命令，即可下载并安装 clash 软件：
```
mkdir -p /data/openclash && cd /data/openclash
export url='https://raw.fastgit.org/juewuy/ShellClash/master'
curl -kfsSl $url/install.sh > install.sh
sh install.sh
```

会出现如下的提示：
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/ax6000/clash-1.png)
install.sh 会将 ShellCrash 安装到 /data/ShellCrash 目录下。

# clash 配置
在安装完 clash 后，输入 clash 命令即可对 clash 进行管理，该操作有点类似于打客户电话的操作。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/ax6000/clash-2.png)

# clash UI
通过上述 clash 命令可以安装 clash UI，可以通过网址 [http://192.168.31.1:9999/ui](http://192.168.31.1:9999/ui) 访问。UI 的功能较 clash 的客户端更为简单，但也可以弥补上述 clash 命令的很多不足之处。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/ax6000/clash-ui.png)

# 资料
[https://www.youtube.com/watch?v=e90UWDpAlxA](https://www.youtube.com/watch?v=e90UWDpAlxA)