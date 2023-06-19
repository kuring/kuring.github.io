---
title: Linux From Scratch学习笔记
Status: public
url: lfs
date: 2014-01-20
---

最近买了本新书《深度探索Linux操作系统》，在按照书中步骤学习的过程中，无奈在安装步骤中出错，于是只能到网上找书评。在浏览书评的过程中偶然看到LFS三个名词，google之，发现是手动安装Linux的官方学习手册，学习之。

# 解压命令
一直对解压命令的参数记不清楚，记录一下：
tar -Jvxf *.tar.xz
tar -zvxf *.tar.gz
tar -xvf *.tar.bz2

# 将一般用户可以使用sudo命令执行命令的方法
执行visudo命令来修改文件内容，将本用户添加到文本文件中。修改的文件为/etc/sudoers，该文件默认为只读的，但是可以通过visudo命令来修改。

# login shell和non-login shell
login shell：shell会重新读取/etc/profile和~/.bash_profile来应用新的环境变量。通过`su - 用户名`的方式登录为login shell方式。
non-login shell：此时shell不会读取/etc/profile和~/.bash_profile，而是读取~/.bashrc来应用新的环境变量。通过`su 用户名`登录的方式为non-login shell方式。

文中lfs用户下的~/.bash_profile的内容如下：
```
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
```

而lfs用户下的~/.bashrc文件的内容如下：
```
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/tools/bin:/bin:/usr/bin
export LFS LC_ALL LFS_TGT PATH
```

而最令人奇怪的是即使通过`su - lfs`命令登录也会执行到.bashrc文件的内容，不信可以通过在.bash_profile和.bashrc文件的开始地方打印内容来验证。之所以出现如此奇怪的问题，原因在于~/.bash_profile中的命令。其中exec命令和linux系统中的exec系列函数的含义是一致的，即在当前bash中直接执行exec后面的命令，而不用fork一个新的进程来执行。env命令会通过当前用户的HOME和TERM环境变量及自定义的PS1环境变量来执行新的/bin/bash，而新执行的bash为non-login shell方式，因此会执行lfs用户下的.bahsrc文件。

总结一下，就是~/.bash_profile文件中的env命令通过non-login shell方式执行了新的bash，exec命令的作用是不在当前bash中执行新的bash，而不是通过产生一个新进程的方式来执行bash。

# POSIX && FHS && LSB
[POSIX.1-2008](http://pubs.opengroup.org/onlinepubs/9699919799/)，通过该网站来查询系统函数等非常方便。
Filesystem Hierarchy Standard (FHS)
[Filesystem Hierarchy Standard (FHS)](http://www.pathname.com/fhs/pub/fhs-2.3.html)，可以通过此标准来学习Linux的目录含义。
[Linux Standard Base (LSB) Specifications](http://refspecs.linuxfoundation.org/lsb.shtml)

# set +h
The set +h command turns off bash's hash function. Hashing is ordinarily a useful feature—bash uses a hash table to remember the full path of executable files to avoid searching the PATH time and again to find the same executable.

# 虚拟终端PTYs
PTY 设备与终端设备（terminal device）相类似——它接受来自键盘的输入，并将文字传递给运行在其上的程序以备输出。PTY 被依次编号，且每个 PTY 的编号就是它在 /dev/pts 目录中对应设备文件的文件名。

# devpts file system
远程登陆(telnet,ssh等)后创建的控制台设备文件所在的目录。 

# specs文件
gcc 是一个驱动式的程序. 它调用其它程序来依次进行编译, 汇编和链接. GCC 分析命令行参数, 然后决定该调用哪一个子程序, 哪些参数应该传递给子程序. 所有这些行为都是由 SPEC 字符串(spec strings)来控制的. 通常情况下, 每一个 GCC 可以调用的子程序都对应着一个 SPEC 字符串, 不过有少数的子程序需要多个 SPEC 字符串来控制他们的行为.

# 查看当前shell
1. 查看默认shell可以用命令：`echo $SHELL`。
2. 查看当前shell：`ps | grep $$ | awk '{print $4}' `。
3. 通过输入一个不存在的命令来查看，如输入tom，可显示`bash: tom: command not found`，说明当前的shell为bash。

# expect
一种提供自动交互的脚本语言。

# tee命令
重定向输出到多个文本文件命令。

# pkg-config
configure脚本在检查相应依赖环境时(例：所依赖软件的版本、相应库版本等)，通常会通过pkg-config的工具来检测相应依赖环境。

详细内容见：[简述configure、pkg-config、pkg_config_path三者的关系](http://www.mike.org.cn/articles/description-configure-pkg-config-pkg_config_path-of-the-relations-between/)
