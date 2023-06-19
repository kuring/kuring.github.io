---
title: CentOS中将光盘作为安装源
Status: public
url: centos_yum_iso
date: 2014-02-18
---

本文是VMWare下的CentOS操作系统将yum源更改为光盘的实例，光盘的iso文件存放在宿主机器上，通过VMWare的共享文件夹功能与CentOS系统共享文件。CentOS中共享文件夹的存放路径为/mnt/hgfs中。CentOS的光盘为两张，分别为CentOS-6.5-x86_64-bin-DVD1.iso、CentOS-6.5-x86_64-bin-DVD2.iso。注意LiveCD版的CentOS系统盘是不可以作为yum源的。

# 挂载光盘

1. `mkdir -p /media/cdrom;mkdir -p /media/CentOS`，创建挂载两个挂载目录，分别挂载DVD1和DVD2。
2. 执行`mount /mnt/hgfs/iso/CentOS-6.5-x86_64-bin-DVD1.iso /media/cdrom/ -o loop;mount /mnt/hgfs/iso/CentOS-6.5-x86_64-bin-DVD2.iso /media/CentOS/ -o loop;`，将iso挂载到创建的目录下。
3. 执行`df -h`命令即可看到挂载的文件系统，输出如下：
```
/mnt/hgfs/iso/CentOS-6.5-x86_64-bin-DVD1.iso  4.2G  4.2G     0 100% /media/cdrom
/mnt/hgfs/iso/CentOS-6.5-x86_64-bin-DVD2.iso  1.2G  1.2G     0 100% /media/CentOS
```

# 设置本地yum源

在/etc/yum.repos.d目录下CentOS-Base.repo记录着yum通过网络更新的源，CentOS-Media.repo记录着通过本地文件更新的源。其中CentOS-Media.repo文件的内容如下：
```
# CentOS-Media.repo
#
#  This repo can be used with mounted DVD media, verify the mount point for
#  CentOS-6.  You can use this repo and yum to install items directly off the
#  DVD ISO that we release.
#
# To use this repo, put in your DVD and use it with the other repos too:
#  yum --enablerepo=c6-media [command]
#
# or for ONLY the media repo, do this:
#
#  yum --disablerepo=\* --enablerepo=c6-media [command]

[c6-media]
name=CentOS-$releasever - Media
baseurl=file:///media/CentOS/
        file:///media/cdrom/
        file:///media/cdrecorder/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
```

其中已经包含了/media/cdrom路径和/media/CentOS路径，至此配置已经完毕。

安装软件需要通过命令`yum --disablerepo=\* --enablerepo=c6-media [command]`，执行`yum [command]`命令时还是联网执行。

# 设置开启启动自动挂载iso文件

在/etc/fstab文件中的末尾增加如下内容：
```
/mnt/hgfs/iso/CentOS-6.5-x86_64-bin-DVD1.iso /media/cdrom/ iso9660 loop 0 0
/mnt/hgfs/iso/CentOS-6.5-x86_64-bin-DVD2.iso /media/CentOS/ iso9660 loop 0 0
```