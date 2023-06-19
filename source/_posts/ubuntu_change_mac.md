---
title: 在ubuntu中更改mac地址的方法
Status: public
url: ubuntu_change_mac
date: 2013-11-12
---

本文提供简易shell脚本来更改mac地址，在其他linux发行版中去掉sudo即可。脚本内容如下：
```
#!/bin/sh
sudo ifconfig eth0 down
sudo ifconfig eth0 hw ether 08:00:27:DF:B3:7B
sudo ifconfig eth0 up
```
