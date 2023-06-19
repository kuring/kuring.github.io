---
title: Linux Bridge brctl命令
date: 2020-06-21 17:01:40
tags:
---

## 查看网桥设备以及端口

使用brctl show可以查看本地上的所有的网桥设备以及接到网桥设备上的所有网络设备。

## 查看网桥设备的mac地址表

执行brctl showmacs ${dev}，常用来排查一些包丢在网桥上的场景。
其中port no为网桥通过mac地址学习到的某个mac地址所在的网桥端口号。
```
$ brctl showmacs br0
port no mac addr                is local?       ageing timer
 1     02:50:89:59:ac:4b       no                 3.96
69     02:e2:14:78:d7:92       no                 0.57
 1     0a:1e:01:dc:67:87       no                10.23
 1     0a:60:3c:ca:a8:85       no                 6.04
 1     0e:01:ce:d6:fc:66       no                 8.36
 1     0e:0c:f8:6c:08:75       no                56.73
58     0e:49:85:f6:a1:40       no                 1.30
22     0e:c0:99:b0:d9:f9       no                 0.85
```

##  查看网桥设备的某个端口的挂载设备
在上文中中可以获取到某个mac地址对应的网桥设备的端口号，要想知道某个网桥设备的端口号对应的设备可以使用brctl showstp ${dev}命令。

```
brctl showstp br0
br0
 bridge id              8000.ae90501b5b47
 designated root        8000.ae90501b5b47
 root port                 0                    path cost                  0
 max age                  20.00                 bridge max age            20.00
 hello time                2.00                 bridge hello time          2.00
 forward delay            15.00                 bridge forward delay      15.00
 ageing time             300.00
 hello timer               0.03                 tcn timer                  0.00
 topology change timer     0.00                 gc timer                  62.37
 flags
bond0.11 (1)
 port id                8001                    state                forwarding
 designated root        8000.ae90501b5b47       path cost                100
 designated bridge      8000.ae90501b5b47       message age timer          0.00
 designated port        8001                    forward delay timer        0.00
 designated cost           0                    hold timer                 0.00
 flags
veth02b41ce8 (20)
 port id                8014                    state                forwarding
 designated root        8000.ae90501b5b47       path cost                  2
 designated bridge      8000.ae90501b5b47       message age timer          0.00
 designated port        8014                    forward delay timer        0.00
 designated cost           0                    hold timer                 0.00
 flags
 hairpin mode              1
```
