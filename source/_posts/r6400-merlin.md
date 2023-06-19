---
title: 网件R6400V2刷梅林教程
date: 2019-02-21 21:17:32
tags:
---

最近突发奇想，想折腾一下路由器。经过研究半天后，锁定了网件R6400这款路由器，原因是可选择的系统较多，跟华硕的路由器架构一致，且支持较为强大的梅林系统。

整个刷机的过程还是非常简单的，虽然花了我不少时间，本文简单记录一下。

拿到手后，R6400比我印象中的要大不少，可以通过下图中的苹果手机进行对比。

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-11.jpeg)

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-12.jpeg)

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-13.jpeg)

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-14.jpeg)

R6400有v1和v2两个版本，其中v1版本的CPU频率为800MHz，v2版本的CPU频率为1GHz。v1版本的刷梅林系统和v2版本刷系统有所区别，v2版本需要先刷DD-WRT固件作为过渡固件，然后再刷梅林固件。

整个的过程最好用有线连接路由器操作，用无线会频繁掉线。

*.chk结尾的文件为过渡固件，*.trx为最终固件。

注意：下载的固件文件最好检验一下md5，确保固件的正确性。

## 刷dd-wrt固件

连接上路由器后，在chrome浏览器中输入http://www.routerlogin.net可跳转到网件管理系统，通过一堆路由器设置后会重启路由器，然后重新登录。

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-1.png)

选择“是”后会下载新网件固件，其实该步骤可以选择“否”即可。固件下载完成后，会升级固件，路由器会自动重启。

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-2.png)

下载DD-WRT固件文件“DD固件.chk”，并在路由器的管理界面“高级 -> 管理 -> 路由器升级”中上传固件。

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-3.png)

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-4.png)

这里选择是，然后开始升级固件，升级完成后路由器会重启。

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-5.png)

重启完成后，Wifi信号变成dd-wrt，没有密码，可直接连接。

在浏览器中输入192.168.1.1，会出现dd-wrt的界面。用户名和密码可以直接输入admin，因为该系统仅为中间过度系统。在dd-wrt这个偏工程师化的系统中有非常详细的信息，包括路由器的CPU和Memory等的硬件信息，甚至还有load average，多么熟悉的指标。

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-6.png)

## 升级梅林固件

在dd-wrt的固件升级中选择“R6400_380.70_0-X7.9.1-koolshare.trx”，刷入梅林固件。待路由器重启完成后，即完成梅林固件的刷入。此时路由器的Wifi SSID变为“NETGEAR”。

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-8.png)

访问192.168.1.1，会出现梅林系统的管理界面，依次设置即可。

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-9.png)

题外话：无线的密码修改完后，悲剧的事情发生了，路由器重启后居然连不上wifi，提示密码错误。不得不找来一台带有网口的笔记本用有线连接。在梅林管理系统中查看，未发现密码输入错误，明明输入的密码是对的，但SSID换一个密码居然奇迹般的可以无线连接了，怀疑是一个bug。

设置完成后路由器会重启，此时管理系统地址变更为192.168.50.1。

在“高级设置 -> 无线网络 -> 专业设置”中，调整区域为“United States”，据说可以加快速度。

要想使用软件中心，需要在系统设置中开启下图选项，并重启路由器。重启后，Format JFFS partition at next boot会自动设置为false。

![](https://kuring.oss-cn-beijing.aliyuncs.com/r6400/r6400-10.png)

## ASUS Router

下载app “ASUS Router”，可以直接连接到路由器，这是因为网件的路由器架构跟华硕完全一致。

## ref

- [网件Netgear R6400 v2 开箱 刷梅林固件](https://www.youtube.com/watch?v=sM3IJrXI7g0)
- [梅林系统官网](https://asuswrt.lostrealm.ca/)
- [梅林固件下载地址](http://koolshare.cn/thread-139324-1-1.html)
