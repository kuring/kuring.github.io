---
url: x_window
Tags: Linux
title: X Window学习
date: 2014-02-17
---

X Window的实现机制还是比较难以理解的，尤其是跟软件开发中的客户端-服务器模式不太一样。涉及到概念也比较多，甚至很对教程对概念的理解不一。最近深入学习了下X Window的原理，在此做一下整理。先上一个摘自维基百科的图：
![X Window架构](/ref/x_window/X_client_server_example.svg.png)

常用快捷键
ctrl+alt+fn：切换到相应的虚拟控制台，n为1-12。默认情况下，linux操作系统会在1-6上运行6个虚拟控制台。
ctrl+alt+退格键：关闭X window系统。
在vmware环境下，ctrl+alt快捷键跟vmware冲突，需要先按住ctrl+alt，然后按一下空格键并松开，再按下相应的fn键才能使用。

X Server
负责硬件管理、屏幕绘制、字体，并接收输入设备（如键盘、鼠标等）的动作，并且告知X Client。
Linux下的X Server软件为Xorg，通过X（Xorg的链接文件）命令即可执行。
输入X命令后，会在第7个控制台启动X Server，将会出现一个什么都没有的漆黑界面，这是由于没有任何X client程序输入的原因。
可以在Linux下启动多个X Server软件，从0开始编号。如果再执行`X:2`命令会启动第二个X Server，此时X Server会在第8个控制台运行。如果第一次执行的是`X:2`命令则X Server会在第7个控制台运行。
在Windows操作系统下的Xming、Xmanager等可以远程连接Linux界面的软件其实就是X Server。

X Client
即X应用程序，运行在X Window下的窗口程序都属于X client。比如firefox就是一个X Client。接收来自X Server的处理动作，将动作处理成为绘图数据，并将绘图数据传回给X Server。X Client与X Server之间通过X Window System Core Protocol协议进行通讯。
xclock是一个简易的X Client的时钟程序，在:1上启动X Server后，执行`xclock –display :1&`命令将xclock输出到X Server后的画面如下：
![xclock](/ref/x_window/xclock_no_wm.png)
该程序可以在X Server上执行，但是画面非常简陋，甚至没有窗口的菜单栏和最大化等按钮。

Window Manager
一种特殊的X Client，提供了窗口的样式。常用的Window Manager包括GNOME默认的metacity、twm等。
将metacity输出到:1上的X Server的命令为`metacity –display=:1 &`，效果如下：
![Image Title](/ref/x_window/xclock_wm.png)
可以看到窗口多出了最小化、最大化、关闭按钮，并且窗口可以移动和缩放等操作。

Display Manager
提供用户登录画面、帮助X Server建立Session。
gnome采用的Display Manager为gdm，KDE采用kdm，还有tdm、xdm等。

Desktop Manager
X Server、X Client、Window Manager的一个集合。常用的Desktop Manager包括：KDE、GNOME等。

startx启动流程
在命令行下执行startx命令后，系统直接进入了桌面环境，并未出现登录界面。进程树如下：
![Image Title](/ref/x_window/startx_pstree.png)
1. startx会调用xinit命令，xinit命令的主要是启动一个X Server软件。
2. 接着xinit命令会调用gnome-session启动gnome的环境所需要的软件。

init 5启动流程
在命令行下执行init 5，首先出现的画面为登录信息。进程树如下：
![Image Title](/ref/x_window/init5_pstree.png)
1. 执行/etc/rc.d/rc5.d中的daemon。
2. 执行/etc/X11/prefdm文件，会选择启动gdm、kdm、xdm、tdm。
3. 这里以gdm为例，gdm是一个shell脚本，会启动gdm-binary命令。

实战
Windows主机连接Linux的教程参见我的另外一篇文章《Redhat安装完成之后的设置》中的相关部分。
两台Linux机器之间通过XWindow实现连接的用法比较少见，通常情况下可以通过vnc代替。
Linux主机连接Windows的工具为rdesktop。

参考文章
《鸟哥的Linux私房菜》
《Linux操作系统之奥秘》
视频：[RH033-ULE112-16-linux下X图形显示体系](http://edu.51cto.com/lesson/id-12709.html)
视频：[Xwindow详解](http://www.rhce.cc/?p=1117)