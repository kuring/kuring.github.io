---
title: Linux更改网卡名称
Status: public
url: linux_change_netcardname
date: 2014-05-02
---

本实验为在虚拟机环境中实验，操作系统为Red Hat Enterprise6.0 32位，当前网卡列表如下：

```
[root@localhost ~]# ifconfig 
eth1      Link encap:Ethernet  HWaddr 00:0C:29:8C:58:06  
          inet addr:192.168.124.140  Bcast:192.168.124.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe8c:5806/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:123 errors:0 dropped:0 overruns:0 frame:0
          TX packets:57 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:18770 (18.3 KiB)  TX bytes:11684 (11.4 KiB)
          Interrupt:19 Base address:0x2024

eth2      Link encap:Ethernet  HWaddr 00:50:56:3F:B3:90
          inet addr:192.168.124.141  Bcast:192.168.124.255  Mask:255.255.255.0
          inet6 addr: fe80::250:56ff:fe3f:b390/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:111 errors:0 dropped:0 overruns:0 frame:0
          TX packets:43 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:16101 (15.7 KiB)  TX bytes:10343 (10.1 KiB)
          Interrupt:19 Base address:0x20a4
```

目的为将网卡eth1更改为eth0，将eth2更改为eth3。

# 修改grub.conf文件

在文件中内核启动时增加_biosdevname=0_选项。修改后的文件内容如下：

```
default=0
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title Red Hat Enterprise Linux (2.6.32-71.el6.i686)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-71.el6.i686 ro root=/dev/mapper/VolGroup-lv_root rd_LVM_LV=VolGroup/lv_root rd_LVM_LV=VolGroup/lv_swap rd_NO_LUKS rd_NO_MD rd_NO_DM LANG=zh_CN.UTF-8 KEYBOARDTYPE=pc KEYTABLE=us nomodeset crashkernel=auto rhgb quiet biosdevname=0
        initrd /initramfs-2.6.32-71.el6.i686.img
```

# 更改网卡配置文件内容和文件名称

在/etc/sysconfig/network-scripts目录中将原有的网卡配置文件ifcfg_Auto_eth1和ifcfg_Auto_eth2更改为ifcfg_eth0和ifcfg_eth3，同时修改文件的内容，将文件的内容中的网卡设备名称进行替换。替换后的文件ifcfg_eth0内容如下：

```
TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME="Auto eth0"
UUID=995d037e-3b65-4490-a1fa-f26f6abf066d
ONBOOT=yes
HWADDR=00:0C:29:8C:58:06
PEERDNS=yes
PEERROUTES=yes
DEVICE=eth0
```

# 删除70-persistent-net.rules文件

该文件存在于/etc/udev/rules.d目录下。该文件如果不存在，开始时会自动创建，里面包含了网卡名称的配置信息。

-----

在修改完上述内容后重新启动机器配置就修改过来了,修改完成之后的网卡配置如下：

```
[root@localhost rules.d]# ifconfig                                                                                                                  
eth0      Link encap:Ethernet  HWaddr 00:0C:29:8C:58:06                                                                                             
          inet addr:192.168.124.140  Bcast:192.168.124.255  Mask:255.255.255.0                                                                      
          inet6 addr: fe80::20c:29ff:fe8c:5806/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:87 errors:0 dropped:0 overruns:0 frame:0
          TX packets:75 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:17174 (16.7 KiB)  TX bytes:14520 (14.1 KiB)
          Interrupt:19 Base address:0x2024

eth3      Link encap:Ethernet  HWaddr 00:50:56:3F:B3:90
          inet addr:192.168.124.141  Bcast:192.168.124.255  Mask:255.255.255.0
          inet6 addr: fe80::250:56ff:fe3f:b390/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:72 errors:0 dropped:0 overruns:0 frame:0
          TX packets:76 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:14164 (13.8 KiB)  TX bytes:14955 (14.6 KiB)
          Interrupt:19 Base address:0x20a4
```
