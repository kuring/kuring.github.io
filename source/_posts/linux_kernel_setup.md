---
title: Linux下内核编译安装
Status: public
url: linux_kernel_setup
date: 2014-04-01
---

本文仅为了练习Linux内核源码的编译安装，安装环境为VMWare下的CentOS，现有CentOS版本为`2.6.32-358.el6.x86_64`。
/boot/grub/grub.conf文件内容如下：

```
# 注释部分去掉
default=0
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title CentOS (2.6.32-358.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-358.el6.x86_64 ro root=/dev/mapper/vg_livedvd-lv_root rd_NO_LUKS rd_LVM_LV=vg_livedvd/lv_root rd_NO_MD crashkernel=auto rd_LVM_LV=vg_livedvd/lv_swap  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM LANG=zh_CN.UTF-8 rhgb quiet
        initrd /initramfs-2.6.32-358.el6.x86_64.img
```

# 获取内核源码

首先从Linux的[官方网站](www.kernel.org)下载最新版内核Linux3.13。

执行`tar Jxvf linux-3.13.tar.xz -C/usr/src/kernels`命令将内核源码解压到内核源代码存放目录/usr/src/kernels/，该源码目录并不固定，但推荐将内核源码存放到该目录下。

为了将上次编译时的目标文件及相关设置文件删除，执行`make mrproper`。

# 挑选功能

可以采用了多种方式，这里采用`make menuconfig`的方式来挑选内核功能，该方式不需要X Window（`make xconfig`方式）的支持，而且要比纯命令行方式（`make config`）要直观。执行`make menuconfig`遇到如下错误：

```
[root@localhost linux-3.13]# make menuconfig
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
 *** Unable to find the ncurses libraries or the
 *** required header files.
 *** 'make menuconfig' requires the ncurses libraries.
 ***
 *** Install ncurses (ncurses-devel) and try again.
 ***
make[1]: *** [scripts/kconfig/dochecklxdialog] 错误 1
make: *** [menuconfig] 错误 2
```

这是因为需要ncurses库的支持，下面采用从源码安装的方式安装ncurses。

从ncurses的[官方网站](https://www.gnu.org/software/ncurses/)下载最新版的ncurses-5.9.tar.gz。然后分别执行`./configure`、make`、`make install`命令安装。

# 更改内核版本号标识

为了能够在编译完成后的内核版本中通过`uname -r`看到定义的内核版本号，修改Makefile文件。其中`EXTRAVERSION`字段值为空，将其赋值为kuring。

# 编译内核

执行`make`命令，该过程需要话费很长时间，我在512MB的VM下跑，花费了大约1个半小时时间。

# 编译内核模块

执行`make modules`命令。

# 安装内核模块

执行`make modules_install`命令，会将内核模块安装到/lib/modules目录下。

# 安装内核

执行`make install`命令，产生如下输出：

```
sh /usr/src/kernels/linux-3.13/arch/x86/boot/install.sh 3.13.0kuring arch/x86/boot/bzImage \
                System.map "/boot"
ERROR: modinfo: could not find module vmhgfs
ERROR: modinfo: could not find module vsock
ERROR: modinfo: could not find module vmware_balloon
ERROR: modinfo: could not find module vmci
```
这个错误跟vmware的vmware tools有关，暂时不去管。

这样再去看/boot/grub/grub.conf文件，会看到文件已经变化，已经将新内核添加到开机启动项中。

```
# 注释部分去掉
default=1
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title CentOS (3.13.0kuring)
        root (hd0,0)
        kernel /vmlinuz-3.13.0kuring ro root=/dev/mapper/vg_livedvd-lv_root rd_NO_LUKS rd_LVM_LV=vg_livedvd/lv_root rd_NO_MD crashkernel=auto rd_LVM_LV=vg_livedvd/lv_swap  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM LANG=zh_CN.UTF-8 rhgb quiet
        initrd /initramfs-3.13.0kuring.img
title CentOS (2.6.32-358.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-358.el6.x86_64 ro root=/dev/mapper/vg_livedvd-lv_root rd_NO_LUKS rd_LVM_LV=vg_livedvd/lv_root rd_NO_MD crashkernel=auto rd_LVM_LV=vg_livedvd/lv_swap  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM LANG=zh_CN.UTF-8 rhgb quiet
        initrd /initramfs-2.6.32-358.el6.x86_64.img
```

同时在/boot目录下已经多出了vmlinuz-3.13.0kuring、System.map-3.13.0kuring、initramfs-3.13.0kuring.img文件。

重启系统后，在启动菜单中多出了新内核选项。进入新内核后，执行`uname -r`显示`3.13.0kuring`，说明新内核已经安装完成。