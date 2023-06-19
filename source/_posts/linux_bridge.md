---
title: Linux下搭建网桥及脚本编写
Status: public
url: linux_bridge
tags: 网络
date: 2013-11-14
---

网桥工作在数据链路层，将两个LAN连起来，根据MAC地址来转发帧。Linux下要配置网桥的方法有两种，一种是通过修改配置文件，另外一种是通过brctl工具。修改配置文件的方式是通过修改/etc/sysconfig/network-scripts/ifcgg-eth*文件来完成的，这种方式没有仔细研究。本文将编写两个脚本来完成网桥的创建和删除，脚本的功能为将机器上的网卡eth1和eth2桥接，而网桥本身未设置ip。

# 网桥创建脚本
本脚本利用brctl命令将网卡eth1和eth2桥接，可以通过`brctl show`命令查看结果。
```
#!/bin/bash
# 脚本作用为将两个网卡桥接

# 检测brctl命令是否存在
brctl > /dev/null
if [ $? != 1 ]; then
	echo Command brctl not exist, please setup it. The setup execute command is \"yum install bridge-utils\"
	exit 0
fi

# 检测网桥br0是否存在，如果存在首先删除
declare -i result=$(brctl show | grep eth0 | wc -l)
if [ $result > 0 ]; then
	echo detect the bridge br0 have already exist, first delete it	
	ifconfig br0 down
	brctl delbr br0
fi

ifconfig eth1 0.0.0.0
ifconfig eth2 0.0.0.0
brctl addbr br0
brctl addif br0 eth1
brctl addif br0 eth2
ifconfig br0 up
echo create bridge br0 success, you can use command : \"brctl show\" to check
```

# 网桥删除脚本
本脚本将桥接网卡br0删除
```
#!/bin/bash

# 检测是否存在网桥br0
declare -i result=$(brctl show | grep br0 | wc -l)
if [ $result == 0 ]; then
	echo "bridge br0 not exists, exit immediately"
	exit 0
fi

# 删除网桥br0
ifconfig br0 down
brctl delbr br0
echo "delete bridge br0 success"
```

# 注意事项
brctl命令创建的桥接网卡在机器重启后会删除，最好将创建桥接网卡的命令放入到linux的开机启动脚本中，这样每次开机的时候都可以自动创建桥接网卡了。

# 参考网址
[bridge命令介绍](http://www.linuxfoundation.org/collaborate/workgroups/networking/bridge)


