---
title: 利用aws科学上网
Status: public
url: aws_net
tags: aws
date: 2016-02-17
toc: yes
---

曾经使用过多种科学上网方式，​最近尝试了使用aws的免费试用一年的功能搭建shadowsocks，访问google的速度非常不错，比很多收费的服务要好用，amazon真是良心企业！

本文用于记录在aws上搭建服务的步骤及其中的一些注意事项，步骤不会太详细，aws上关于主机的功能需要读者自己在试验的过程中去自己探索。

# 注册aws账号

为了能够搭建搭建aws服务，拥有一个amazon账号是必须的，在[aws免费套餐](https://aws.amazon.com/cn/free/)的页面点击『创建免费账号』按钮即可按照步骤创建aws账号。

值得一提的是，注册aws的账号需要一张信用卡。

# 开启EC2主机实例

该步骤的目的是开启aws上的主机实例。​

进入aws的控制面板，在左上角的服务中选择EC2，aws提供了多种类型的主机，这里选择EC2即可。

在EC2控制面板界面中需要选择右上角的区域，这个用于选择EC2主机所在的机房，不同机房之间主机是不可以共享的。我这里选择了『美国西部（俄勒冈）』，感觉速度还不错，没有试验过亚洲地区的，新加坡的速度是不是会更好些。后续经过验证，首尔的服务器确实速度更快一些。
​
下面即可创建EC2的实例了，点击界面上的『启动实例』按钮即可按照步骤创建EC2实例了，创建实例的时候一定要选择免费的EC2主机，否则就会悲剧了。我选择了ubuntu14.04的主机，redhat7.2的主机yum源不太全，没有选择使用。

最终会得到ssh登录用的pem文件，用于ssh远程登录主机。并在界面上启动刚刚创建的实例。

# 按照shadowsocks

接下来就是在EC2实例上安装sock5代理工具了。

登录刚刚启动的EC2，需要pem文件。可以通过`ssh -i "key.pem" ubuntu@ec2-52-26-2-14.us-west-2.compute.amazonaws.com`命令来登录到远程主机，其他工具请自行google。

使用命令`pip install shadowsocks`来安装shadowssocks，pip命令的安装自行解决。

在ubuntu的home目录下执行`mkdir shadowsocks`创建保存配置文件的文件夹，并创建配置文件config.json，内容如下：

```
{
    "server":"0.0.0.0",
    "server_port":10001,
    "local_port":1080,
    "password":"xxx",
    "timeout":600,
    "method":"bf-cfb"
}
```

需要说明的是最好配置一下server_port选项，更改shadowsocks的默认端口号。method选项用于控制加密方式，我这里更改为了bf-cfb。

执行`nohup ssserver -c config.json &`命令即可启动shadowsocks服务。

由于对外增加了10001端口号，aws的默认安全策略为仅对外提供22端口，需要在EC2主机的安全策略中增加外放访问tcp端口10001的权限。

# 脚本

为了安装方便，我简单写了个脚本如下

```
yum -y install epel-release
#yum update -y
yum install python2-pip -y
pip install shadowsocks
mkdir ~/shadowsocks
echo '{
    "server":"0.0.0.0",
    "server_port":10001,
    "local_port":1080,
    "password":"xxx",
    "timeout":600,
    "method":"aes-256-cfb"
}' > ~/shadowsocks/config.json
systemctl disable firewalld.service
systemctl stop firewalld.service
nohup ssserver -c ~/shadowsocks/config.json &
```

在某些云主机的CentOS7系统发现无法使用`yum install python2-pip`进行安装，原因是有些源被禁用了，可以使用`yum repolist disabled`来查看被禁用的源，其中会包含epel源。可以使用`yum install python2-pip -y --enablerepo=epel`的方式来安装。

# 安装shadowsocks客户端

这里是支持的[客户端列表](https://shadowsocks.com/client.html)，​我这里仅使用的mac客户端ShadowsocksX，支持Auto Proxy Mode和Global Mode两种方式，其中Auto方式会自动下载使用sock5代理的列表，非常方便。

# kcptun

为了加快访问速度，推荐使用kcp + shadowsocks

kcp的服务端配置如下，即启用20001端口，该端口会将流量导入到127.0.0.1:10001端口，即本机的shadowsocks端口

```
cd ~ && mkdir kcptun && cd kcptun
wget https://github.com/xtaci/kcptun/releases/download/v20190109/kcptun-linux-amd64-20190109.tar.gz
tar zvxf kcptun-linux-amd64-20190109.tar.gz
nohup ./server_linux_amd64 -l :20001 -t 127.0.0.1:10001 -key xxx -mode fast2 --log ~/kcptun/20001.log &
```

配置了kcptun的shadowsocks客户端仅需要配置代理为远程的kcpdun端口即可，不再需要指定shadowsocks的端口，相当于shadowsocks是透明的。

# 监控

为了避免aws产生额外的费用，一定要设置一下费用报警，否则被扣费了就麻烦了。

另外，可定期查看下aws的费用。试用期为一年，一年后一定要记得停掉aws服务。

最后，祝你玩的愉快！
