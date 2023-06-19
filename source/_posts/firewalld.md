title: firewalld
date: 2022-03-29 19:24:27
tags:
author:
---
在RHEL7系统中，firewalld和iptabels两个防火墙的服务并存，firewalld通过调用iptables命令来实现，firewalld和iptables均用来维护系统的netfilter规则，底层实现为内核的netfilter模块。

firewalld为systemd项目的一部分，为python语言实现，跟iptables相比，使用上更加人性化，提供了更高层次的抽象，用户不用关心netfilter的链和表的概念。

## 架构

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/firewalld-structure.png)

最上层为用户界面层，提供了firewall-cmd和firewall-offline-cmd两个命令行工具，其中firewall-cmd为最主要的命令行工具。firewall-config为GUI工具。用户界面层通过firewalld提供的D-Bus接口进行通讯。

firewalld为daemon进程，其中zone、service、ipset等为firewalld抽象的概念。在接收到用户界面层的命令后，一方面需要将操作保存到本地文件，另外还需要调用更底层的如iptables、ipset、ebtables等命令来产生规则，最终在内核层的netfilter模块生效。

## firewalld的管理

在RHEL7系统下，firewalld作为systemd家族的一员会默认安装。其service文件定义如下：

```
$ cat /usr/lib/systemd/system/firewalld.service 
[Unit]
Description=firewalld - dynamic firewall daemon
Before=network-pre.target
Wants=network-pre.target
After=dbus.service
After=polkit.service
Conflicts=iptables.service ip6tables.service ebtables.service ipset.service
Documentation=man:firewalld(1)

[Service]
EnvironmentFile=-/etc/sysconfig/firewalld
ExecStart=/usr/sbin/firewalld --nofork --nopid $FIREWALLD_ARGS
ExecReload=/bin/kill -HUP $MAINPID
# supress to log debug and error output also to /var/log/messages
StandardOutput=null
StandardError=null
Type=dbus
BusName=org.fedoraproject.FirewallD1
KillMode=mixed

[Install]
WantedBy=multi-user.target
Alias=dbus-org.fedoraproject.FirewallD1.service
```

查看firewall-cmd的运行状态

```
$ firewall-cmd --state
not running
```

在安装完firewalld后，会在/usr/lib/firewalld/目录下产生默认的配置，该部分配置不可修改。同时会在/etc/firewalld目录下产生firewalld的配置，该部分配置会覆盖/usr/lib/firewalld/下的配置。/etc/firewalld目录下的文件内容如下，其中firewalld.conf文件为最主要的配置文件。

```
$ ls 
firewalld.conf  helpers  icmptypes  ipsets  lockdown-whitelist.xml  services  zones
```

## 概念

firewalld抽象了zone、service、ipset、helper、icmptypes几个概念。

### zone

firewalld 将网络按照安全等级划分了不同的zone，zone的定义位于/usr/lib/firewalld/zones/目录下，文件格式为xml，包括了如下的zone。

1. drop：只允许出向，任何入向的网络数据包被丢弃，不会回复icmp报文。
2. block：任何入向的网络数据包均会被拒绝，会回复icmp报文。
3. public：公共区域网络流量。不信任网络上的流量，选择接收入向的网络流量。
4. external：不信任网络上的流量，选择接收入向的网络流量。
5. DMZ：隔离区域，内网和外网增加的一层网络，起到缓冲作用。选择接收入向的网络连接。
6. work：办公网络，信任网络流量，选择接收入向的网络流量。
7. home：家庭网络，信任网络流量，选择接收入向的网络流量。
8. internal：内部网络，信任网络流量，选择接收入向的网络流量。
9. trusted：信任区域。所有网络连接可接受。

firewalld的默认zone为public，zone目录下的public.xml的内容如下：
```
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
</zone>
```

可以看到Public zone关联了三个service对象 ssh、dhcpv6-client、cockpit。没有匹配到该zone的流量，默认情况下会被拒绝。

在配置文件/etc/firewalld/firewalld.conf中，通过DefaultZone字段指定的默认zone为public。即如果开启了firewalld规则，那么默认仅会放行访问上述三个服务的流量。执行 iptables-save 命令实际上并为看到任何的iptables规则，说明firewalld是直接调用内核的netfilter来实现的。

### service

service为firewalld对运行在宿主机上的进程的抽象，并在zone文件中跟zone进行绑定。比如ssh.xml的内容如下：

```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>SSH</short>
  <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
  <port protocol="tcp" port="22"/>
</service>
```

## 实践

### 向某个zone内增加端口号

执行 `firewall-cmd --permanent --zone=public --add-port=80/tcp` 即可向public zone内增加80端口号。同时可以在 /etc/firewalld/zones/public.xml 文件中看到新增加了80端口号。

```
$ cat public.xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <port port="80" protocol="tcp"/>
</zone>
```

参数 `--permanent` 为固化到文件中，如果不增加该参数重启firewalld进程后配置会失效。

### 获取当前zone信息

```
$ firewall-cmd --get-active-zones
public
  interfaces: eth0
```

每个网络设备可以属于不同的zone，可以根据网络设备名来查询所属的zone

```
$ firewall-cmd --get-zone-of-interface=eth0
public
```

### 设置当前默认zone

该命令会自动修改/etc/firewalld/firewalld.conf配置文件中的DefaultZone字段。

```
$ firewall-cmd --set-default-zone=trusted
```

### 查看当前配置规则

```
$ firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 80/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

## 相关链接

- [firewalld官方文档](https://firewalld.org)
- [RHEL7: How to get started with Firewalld](https://www.certdepot.net/rhel7-get-started-firewalld/)

