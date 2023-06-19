---
title: 网件R6400V2重新刷回官方系统教程
date: 2021-02-16 02:59:38
tags:
---

之前购买的网件R6400V2路由器刷到了梅林系统，但一直以来信号都特别差，甚至都比不过最便宜的水星路由器，想重新刷回官方系统看下是否是梅林系统的问题。本文记录下重新刷回梅林系统的操作步骤。


## 刷机之路


从如下的地址下载固件。
```
链接：https://pan.baidu.com/s/1EBvUBlXozo_4zkXaeUMQng 密码：pqqv
```


在梅林系统的界面上找到固件升级的地方，将下载的固件上传。


![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/reset-1.png)
结果悲剧的事情发生了，路由器出现了不断重启的状态，并且没有无线信号。猜测可能是因为固件用的是R6400，而非R6400V2导致的。或者是因为是用无线网络升级的原因，导致升级到一半网断掉了，从而导致失败了。


## 救砖之路


按照[网件通用救砖，超详细教程](https://koolshare.cn/thread-142232-1-1.html)文档中的网件通用救砖方法，将路由器的一个LAN口跟电脑的网卡用网线直接相连，并设置网卡的ip地址为192.168.1.3，网关为255.255.255.0。


在路由器启动的状态下，在命令行执行ping -t 192.168.1.1来验证路由器是否可以ping通，如果不通，可能是因为路由器的LAN段不是192.168.1.0，可能是192.168.50.0的。


在windows电脑上下载对应的文件，并保存到本地的F盘下。


手工安装winpcap，主要是给nmrpflash来使用。


以管理员身份运行命令行工具，执行nmrpflash.exe -L后找到本机的网卡net1。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/reset-2.png)
执行nmrpflash命令后，立即重启路由器，如果第一次提示Timeout，可以立即执行该命令，如果出现下图的提示，说明命令执行成功。
![image.png](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/reset-3.png)
# 参考文档

- [万能网件R6400刷回最近官方固件的方法](http://mr.mw/share/136.html)（不适用于R6400V2）
- [网件通用救砖，超详细教程](https://koolshare.cn/thread-142232-1-1.html)
- [Netgear 网件系列路由器救砖工具](https://roov.org/2019/07/netgear-unbrick-utility/#comment-37190)
