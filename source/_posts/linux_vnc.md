---
title: Linux下vnc的配置
Status: public
url: linux_vnc
date: 2013-11-10
---

[TOC]

VNC（Virtual Network Computing）是一套由AT&T实验室所开发的可操控远程的计算机的软件，其采用了GPL授权条款，任何人都可免费取得该软件。VNC软件主要由两个部分组成：VNC server及VNC viewer。用户需先将VNC server安装在被控端的计算机上后，才能在主控端执行VNC viewer控制被控端。VNC与操作系统无关，因此可跨平台使用，例如可用Windows连接到某Linux的电脑，反之亦同。

该软件在RedHat或CentOS中默认是安装的，但是没有启用，一可以通过`which vncserver`命令来查看该命令是否安装。本文讲解在Linux下的搭建server，在Windows下搭建client的步骤。

# 设置vncserver的密码
vncserver需要设置一个密码，该密码并不等同于系统帐号的密码，而是vnc客户端登录的时候输入的密码。执行`vncpasswd`命令来创建密码。

# 修改vncserver配置文件
修改文件/etc/sysconfig/vncservers，在该文件末尾添加如下内容：
```
VNCSERVERS="1:root"
VNCSERVERARGS[1]="-geometry 1024x768 -alwaysshared -depth 24"
```

# 启动vncserver
执行`service vncserver start`命令来开启vncserver服务。

# 客户端连接到服务端
这里采用windows下的vnc viewer工具来连接到vncserver。在输入IP的地方输入：`IP地址:1`来连接到vncserver端，其中`:1`要跟/etc/sysconfig/vncservers文件中的对应标号一致。这样就可以连接上vncserver，但是连接后界面非常简单，跟命令行界面类似。还需要对vncserver进一步配置。

# 配置vncserver
vncserver的配置文件在~/.vnc/xstartup文件中，该文件默认创建的内容如下：
```
#!/bin/sh

# Uncomment the following two lines for normal desktop:
# unset SESSION_MANAGER
# exec /etc/X11/xinit/xinitrc

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
xterm -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
twm &
```

将其中的注释打开，即文件内容如下：
```
#!/bin/sh

unset SESSION_MANAGER
exec /etc/X11/xinit/xinitrc

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
xterm -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
twm &
```

然后执行`service vncserver restart`重新启动vncserver服务。客户端再重新连接vncserver既可以看到正常的界面了。

# 参考文章
[CentOS Linux下VNC Server远程桌面配置详解](http://www.ha97.com/4634.html)
