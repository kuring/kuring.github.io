title: PXE
date: 2023-10-12 20:12:55
tags:
author:
---
重装过电脑操作系统的同学大概知道操作系统的安装流程如下：
1. 在 BIOS 中将系统设置为光驱/USB开机优先模式
2. 以DVD或者 U 盘中的操作系统开机，进入到装机界面
3. 完成一系列的装机初始化，比如磁盘分区、语言选择等
4. 重启进入新安装的操作系统

以上过程必须要手工才能完成，安装一台电脑还可以，但如果要大批量安装一批机器就不适用了。为此，Intel 公司研发了 PXE(Pre-boot Execution Environment) 技术，可以通过网络的方式批量安装操作系统。

PXE 基于 C/S 架构，分为PXE client 和PXE server，其中 PXE client 为要安装操作系统的机器，PXE server 用来提供安装操作系统必须的镜像等信息。要想实现从网络上安装操作系统，必须要解决如下几个问题：
1. 因为还没有安装操作系统，此时并不存在 ip 地址，在装机之前必须要获取到一个 ip 地址。
2. 安装操作系统需要的 boot loader 和操作系统镜像如何获取。

为了解决 PXE client 的 ip 地址问题，PXE 中采用了 DHCP 协议来给 client 分配 ip 地址，这就要求 PXE server 必须要运行 dhcp server。为了解决 PXE server 可以提供 boot loader 和操作系统基线，PXE server 通过 tftp 协议的方式对 client 提供服务。

client 端需要 DHCP client 和 tftp client 的功能，为此 PXE 协议中将该功能以硬件的方式内置在网卡 ROM 中。当启动时，BIOS 会加载内置在网卡中的 ROM，从而该机器具备了 DHCP client 和 tftp client 的功能。

优点：
1. 规模化：可以批量实现多台服务器的安装
2. 自动化：可以自动化安装
3. 远程实现：不用本地的光盘来安装 OS

客户机的前提条件：
1. 网络必须要支持 PXE 协议
2. 主板支持网络引导，一般在 BIOS 中可以配置

服务端：
1. DHCP 服务，用来给客户机分配 ip 地址
2. TFTP 服务：用来提供操作系统文件的下载
