---
Status: public
url: redhat_setup_base
tags: Linux
date: 2014-05-01
title: Redhat安装完成之后的设置
---

[TOC]

＃ 安装kdevelop

确保可以上网，这里采用yum的安装方式进行安装。
首先执行命令：yum install kdevelop，会出现如下提示：
```
Loaded plugins: rhnplugin, security
This system is not registered with RHN.
RHN support will be disabled.
file:///mnt/file/rh5/Cluster/repodata/repomd.xml: [Errno 5] OSError: [Errno 2] 没有那个文件或目录: '/mnt/file/rh5/Cluster/repodata/repomd.xml'
Trying other mirror.
Error: Cannot retrieve repository metadata (repomd.xml) for repository: Cluster. Please verify its path and try again
```

出现上述错误是由于redhat没有注册，所有不能使用它自身的源进行更新，可以更换为CentOS系统的源进行更新，操作步骤为：
1、进入/etc/yum.repos.d/目录。在命令行输入：wget http://docs.linuxtone.org/soft/lemp/CentOS-Base.repo。
2、ls 一下，会看到一个文件名为CentOS-Base.repo的文件。
3、将原来的文件rhel-debuginfo.repo改名为rhel-debuginfo.repo.bak。
4、将CentOS-Base.repo改名为rhel-debuginfo.repo

再次运行命令：yum install kdevelop，就可以安装kdevelop了。

安装过程中遇到了需要的pcre包无法从centos的源中下载的问题，解决方法为根据yum命令无法下载的包，在google中搜索，下载包然后再redhat上用rpm的升级命令来安装。具体下载网址为：[电子科技大学星辰工作室开源镜像服务](http://mirrors.stuhome.net/centos/5.9/cr/x86_64/RPMS/)。

rpm相关命令：
安装一个包：rpm   -ivh       
升级一个包：rpm   -Uvh   
移走一个包：rpm   -e 

# 安装konsole

安装上kdevelop后在执行程序的时候会提示`/bin/sh:konsole:command not found`。执行`yum install kdebase`命令来安装konsole。

# 配置ssh服务

修改ssh服务的配置文件/etc/ssh/sshd_config文件，将文件中的#PasswordAuthentication yes注释打开。
修改ssh服务的配置文件/etc/ssh/sshd_config文件，将文件中的PermitRootLogin no更改为yes。这样即可以用ssh工具连接到该机器。

# xmanager连接配置

该部分参考文档的网址为：http://blog.csdn.net/gltyi99/article/details/6141972
1. 修改/usr/share/gdm/defaults.conf文件的权限，默认权限为444，chmod 700 /usr/share/gdm/defaults.conf。
2. 在/usr/share/gdm/defaults.conf文件的末尾添加如下内容：
```
Enable=true
DisplaysPerHost=10
Port=177
AllowRoot=true
AllowRemoteroot=true
AllowRemoteAutoLogin=false
```
3. 修改/etc/gdm/custom.conf文件
```
[xdmcp]
Enable=1
```
4. 修改/etc/inittab文件，不修改原来的设置，在文件的最后增加一行：
```
x:5:respawn:/usr/sbin/gdm
```
5. 修改/usr/share/gdm/defaults.conf文件，将其中的
```
[security]
# Allow root to login.  It makes sense to turn this off for kiosk use, when
# you want to minimize the possibility of break in.
AllowRoot=true
# Allow login as root via XDMCP.  This value will be overridden and set to
# false if the /etc/default/login file exists and contains
# "CONSOLE=/dev/login", and set to true if the /etc/default/login file exists
# and contains any other value or no value for CONSOLE.
AllowRemoteRoot=false
# This will allow remote timed login.
AllowRemoteAutoLogin=false
# 0 is the most restrictive, 1 allows group write permissions, 2 allows all
# write permissions.
RelaxPermissions=0
# Check if directories are owned by logon user.  Set to false, if you have, for
# example, home directories owned by some other user.
CheckDirOwner=true
# Number of seconds to wait after a failed login
#RetryDelay=1
# Maximum size of a file we wish to read.  This makes it hard for a user to DoS
# us by using a large file.
#UserMaxFile=65536
```
AllowRemoteRoot=false更改为AllowRemoteRoot=true。

6. 修改/etc/securetty文件，在文件底部添加如下内容：
```
pts/0
pts/1
pts/2
pts/3
pts/4
```
7. 修改/etc/pam.d/login文件，将其中的一行注释
```
#auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_securetty.so
```
8. 修改/etc/pam.d/remote，将其中的一行注释
```
#auth       required     pam_securetty.so
```
9. 修改/etc/xinetd.d/krb5-telnet文件，将文件内容由
```
service telnet
{
        flags           = REUSE
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/kerberos/sbin/telnetd
        log_on_failure  += USERID
        disable         = yes
}
```
更改为：
```
service telnet
{
        flags           = REUSE
        socket_type     = stream
        wait            = no
        user            = root
        server          = /usr/kerberos/sbin/telnetd
        log_on_failure  += USERID
        disable         = no
}
```
同样将/etc/xinetd.d/ekrb5-telnet文件中的disable=yes更改为disable=no。

# 安装中文字体
为了阅读代码方便，安装字体。
1. 将字体文件YaHei.Consolas.1.12.ttf放到Redhat的目录/usr/share/fonts/chinese/TrueType目录下
2. 执行mkfontscale命令，重新生成fonts.scale文件
3. 执行mkfontdir命令，重新生成了fonts.dir文件。
4. 执行chkfontpath --add /usr/share/fonts/chinese/TrueType

# 更改操作系统编码从utf8到gb18030
可以通过locale命令来查看操作系统编码。输出如下：
```
LANG=zh_CN.UTF-8
LC_CTYPE="zh_CN.UTF-8"
LC_NUMERIC="zh_CN.UTF-8"
LC_TIME="zh_CN.UTF-8"
LC_COLLATE="zh_CN.UTF-8"
LC_MONETARY="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
LC_PAPER="zh_CN.UTF-8"
LC_NAME="zh_CN.UTF-8"
LC_ADDRESS="zh_CN.UTF-8"
LC_TELEPHONE="zh_CN.UTF-8"
LC_MEASUREMENT="zh_CN.UTF-8"
LC_IDENTIFICATION="zh_CN.UTF-8"
LC_ALL=
``` 

打开/etc/sysconfig/i18n文件，文件默认内容为：
```
LANG="zh_CN.UTF-8"
```
将文件内容修改为：
```
LANG="zh_CN.GBK"
```

# 相关下载

[最适合程序员的字体：微软雅黑+Consolas](http://pan.baidu.com/s/10ABX8)
