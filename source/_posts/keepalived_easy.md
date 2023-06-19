---
title: keepalived简易教程
Status: public
url: keepalived_easy
tags: keepalived
date: 2015-10-23
toc: yes
---

keepalived的作用为保持存活服务，服务启动后会在两台物理机器之间维护一个vip，但是仅有一台物理机器拥有该vip，这样就保证了两台机器之间是主备。

# 安装

在ubuntu下直接执行：`sudo apt-get install keepalived`.

# 使用

本例子两台机器的物理ip地址分别为10.101.185和10.101.1.186，要增加的虚拟ip地址为10.101.0.101、10.101.0.102、10.101.0.107和10.101.0.108，其中10.101.0.101和10.101.0.102在10.101.185上为主，10.101.0.107和10.101.0.108在10.101.1.186上为主。

keepalived的默认配置文件位于/etc/keepalived/keepalived.conf目录下，由于两台物理机器之间的主辅关系不同，配置文件也不相同。

10.101.185机器上的配置文件如下：

```
! Configuration File for keepalived

global_defs {
   # 报警邮箱配置
   notification_email {
     ops@yidian-inc.com
   }
   smtp_server 10.101.1.139
   smtp_connect_timeout 30
   router_id 101-1-185-lg-201-l10.yidian.com  // 运行机器的唯一标识，每个机器应该都不一样，可以直接使用hostname代替，具体用在什么地方暂时不是很清楚
}

vrrp_instance ha-internal-1 {
    state MASTER
    interface eth0
    virtual_router_id 1	// VRID标记，可以设置为0-255，对应VRRD协议中的Virtual Rtr Id
    priority 100  // 对应VRRD协议中的priority选项
    advert_int 1	// 检测间隔，默认为1s，对应VRRD协议中的adver int
    authentication {
        auth_type PASS // 认证方式，支持PASS和AH
        auth_pass 1-internal-ha  // 认证的密码，从抓取的包中看到
    }
    // 声明的虚拟ip地址，这些ip会在VRRP一些的一个包发送
    // 另外VRRP协议中还有一个Count IP Addrs用来指明需要声明多少个VIP
    virtual_ipaddress {
        10.101.0.101/22 dev eth0
        10.101.0.102/22 dev eth0
    }
}

vrrp_instance ha-internal-2 {
    state BACKUP
    interface eth0
    virtual_router_id 2
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 2-internal-ha
    }
    virtual_ipaddress {
        10.101.0.107/22 dev eth0
        10.101.0.108/22 dev eth0
    }
}
```

10.101.1.186上的配置文件如下：

```
! Configuration File for keepalived

global_defs {
   notification_email {
     ops@yidian-inc.com
   }
   smtp_server 10.101.1.139
   smtp_connect_timeout 30
   router_id 101-1-186-lg-201-l10.yidian.com
}

vrrp_instance ha-internal-1 {
    state BACKUP
    interface eth0
    virtual_router_id 1
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1-internal-ha
    }
    virtual_ipaddress {
        10.101.0.101/22 dev eth0
        10.101.0.102/22 dev eth0
    }
}

vrrp_instance ha-internal-2 {
    state MASTER
    interface eth0
    virtual_router_id 2
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 2-internal-ha
    }
    virtual_ipaddress {
        10.101.0.107/22 dev eth0
        10.101.0.108/22 dev eth0
    }
}
```

配置文件搭建完毕后，通过`sudo service keepalived start`即可启动服务，执行`ip addr`命令即可看到vip。需要注意的是，通过`ifconfig`命令是看不到vip的。

有了vip，其他服务就可以利用该vip做一些绑定vip的端口来作为主辅热备模式了。

# about vrrp

关于VRRP的详细说明可以查看[RFC3768](https://tools.ietf.org/html/rfc3768)，我这里记录几点说明。

协议中的以太网Destination Address的值必须为多播地址224.0.0.18。

当前正在使用的VRRP版本为version 2，认证功能已经取消，但为了向下兼容，仍然可用。在抓取的包中，仍在使用认证信息

# download

[这里提供两个vrrp协议的pcap包](http://pan.baidu.com/s/1jHreDyI)