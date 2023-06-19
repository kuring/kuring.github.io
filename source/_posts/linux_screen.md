---
title: Linux下的screen命令
date: 2014-02-18 20:51:12
Status: public
url: linux_screen
---

在使用SSH或者telent远程登录到Linux 服务器，经常运行一些需要很长时间才能完成的任务，比如系统备份、ftp 传输等等。通常情况下我们都是为每一个这样的任务开一个远程终端窗口，因为它们执行的时间太长了。必须等待它们执行完毕，在此期间不能关掉窗口或者断开连接，否则这个任务就会被杀掉，一切半途而废了。

# 安装
CentOS下默认没有安装该命令，从screen的官方网站下载，下载地址：http://ftp.gnu.org/gnu/screen/。

1. 解压后执行`./configure`。
2. 执行`make`命令。在执行make命令时会遇到错误`pty.c:38:26: 错误：sys/stropts.h：没有那个文件或目录`，在/usr/install/sys/目录下创建一个stropts.h的空文件即可。
3. 执行`make install`，该命令并不会将screen命令复制到系统的PATH变量包含的路径下，即不能执行screen命令。
4. 执行`install -m 644 etc/etcscreenrc /etc/screenrc`。
5. 执行`cp screen /bin/`。
6. 执行`cp doc/man/man1/screen.1 /usr/share/man/man1/`，即可以可使用`man screen`查看帮助。

# 实现原理
当关闭窗口或断开连接时，内核会将SIGHUP信号发送到会话期首进程，进程对SIGHUP的处理动作为终止。如果会话期首进程终止，则该信号发送到该会话期前台进程组。一个进程退出导致一个孤儿进程组中产生时，如果任意一个孤儿进程组进程处于STOP状态，发送SIGHUP和SIGCONT信号到该进程组中所有进程。因此当网络断开或终端窗口关闭后，控制进程收到SIGHUP信号退出，会导致该会话期内其他进程退出。

为解决上述问题，Linux程序在设计时可设计成守护进程的方式启动。

另外也可以通过`nohup 命令 &`的方式启动来解决问题。

# 新建screen

* 直接输入`screen`命令即可使用。
* 输入`screen -S kuring`，这里给screen取了一个名字，方便辨认。
* `screen 命令`直接在screen中执行命令，命令结束后screen退出。

# 分离与恢复

在screen窗口中执行`ctrl + a d`命令，screen会给出_[detached]_提示，并恢复到执行screen之前的bash。
查找之前的screen执行`screen -ls`，会列出当前打开的screen。
```
There are screens on:
        8576.pts-3.localhost    (Attached)
        8449.kuring     (Detached)
2 Sockets in /tmp/uscreens/S-kuring.
```
这里系统中打开了两个screen，一个为Attached，另一个为Detached。

重新连接执行`screen -r kuring`或`screen -r 8449`或`screen -r`，当系统中仅有一个处于Detached状态的screen时就可以直接执行`screen -r`命令。

# 关闭窗口

在screen的shell中执行`exit`命令即可关闭screen。

也可以执行`ctrl + a k`，会杀死当前screen中的所有运行进程。

# 错误

在screen中执行vi命令时会提示_E437: terminal capability "cm" required_错误，执行`echo $TERM`查看发现打印值为_screen_，而未执行screen时的bash打印值为_xterm_，在screen中执行`export TERM=xterm`即可解决该问题。

# 参考文档

更多高级使用方法请参考以下文档：

[linux 技巧：使用 screen 管理你的远程会话](http://www.ibm.com/developerworks/cn/linux/l-cn-screen/)
[linux screen 命令详解](http://www.cnblogs.com/mchina/archive/2013/01/30/2880680.html)