---
title: Linux下debug内核coredump
date: 2019-10-02 02:09:32
tags:
---

Linux内核会存在一些严重的bug，导致内核crash，会在/var/crash目录下产生类似”127.0.0.1-2019-09-30-21:33:38“这种的文件夹，里面包含了vmcore文件，该文件对于debug 内核crash的原因非常有帮助。

本文在CentOS 7下操作。

执行`yum install crash`来安装crash

另外还需要两个rpm包：kernel-debuginfo-3.10.0-957.el7.x86_64.rpm 和 kernel-debuginfo-common-x86_64-3.10.0-957.el7.x86_64.rpm，需要关注下操作系统的内核版本，这两个rpm包可以通过搜索引擎找到。

下到包后即可执行`rpm -ivh *.rpm`的方式来安装rpm包。

在机器上执行`crash /usr/lib/debug/lib/modules/3.10.0-957.el7.x86_64/vmlinux /var/crash/xx/vmcore`进行debug，可以输入`bt`命令来查看栈信息。
