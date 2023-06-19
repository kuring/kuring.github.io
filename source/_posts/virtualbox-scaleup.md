---
title: VirtualBox磁盘扩容
date: 2019-04-04 20:15:44
tags:
---

今天发现vagrant的其中一个虚拟机磁盘空间不够了，需要对其进行磁盘扩容，但不期望是通过增加新硬盘的方式，而是直接增加原磁盘容量的方式来无缝扩容。

## 修改磁盘文件

进入到vm磁盘文件所在的目录`~/VirtualBox VMs/dev_default_1531796361866_92956`下

```
% vboxmanage showhdinfo  centos-vm-disk1.vmdk
UUID:           acbb4ffc-0580-40d6-8627-3ed24cd0beff
Parent UUID:    base
State:          created
Type:           normal (base)
Location:       /Users/lvkai/VirtualBox VMs/dev_default_1531796361866_92956/centos-vm-disk1.vmdk
Storage format: VMDK
Format variant: dynamic default
Capacity:       10000 MBytes
Size on disk:   9634 MBytes
Encryption:     disabled
In use by VMs:  dev_default_1531796361866_92956 (UUID: a153957c-e43f-4dd2-8512-f51d42dee3d3)

# 将之前存储的vmdk格式的文件复制一份vdi格式的文件，由于需要复制文件，该命令需要执行一段时间
% vboxmanage clonehd centos-vm-disk1.vmdk new-centos-vm-disk1.vdi --format vdi

# 将vdi格式的文件修改磁盘空间上限大小为80g，但实际占用磁盘空间仍然为之前的大小
% vboxmanage modifyhd  new-centos-vm-disk1.vdi --resize 81920

# 将vdi格式的文件重新转换为vmdk格式，会产生一个新的uuid
% vboxmanage clonehd new-centos-vm-disk1.vdi resized.vmdk --format vmdk
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Clone medium created in format 'vmdk'. UUID: 7e454b50-0681-494b-b9ca-81700d217c0a
```

新的硬盘创建完成后，在virtualbox的界面上将对应虚拟机的硬盘更换为resized.vmdk，并将之前旧的centos-vm-disk1.vmdk给删除掉。

## 使用fdisk创建新的磁盘分区

以上命令执行完成后，开启虚拟机，进入系统，可以看到磁盘空间大小变更为85.9GB，但挂载的磁盘空间大小仍然为8.3G，新增加的磁盘空间仍然处于未分配状态。

```
# fdisk -l

Disk /dev/sda: 85.9 GB, 85899345920 bytes, 167772160 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0000ca5e

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    20479999     9726976   8e  Linux LVM

Disk /dev/mapper/centos-root: 8866 MB, 8866758656 bytes, 17317888 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-swap: 1048 MB, 1048576000 bytes, 2048000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root  8.3G  7.9G  386M  96% /
devtmpfs                 296M     0  296M   0% /dev
tmpfs                    307M     0  307M   0% /dev/shm
tmpfs                    307M  4.5M  303M   2% /run
tmpfs                    307M     0  307M   0% /sys/fs/cgroup
/dev/sda1                497M  195M  303M  40% /boot
vagrant                  466G  390G   77G  84% /vagrant
vagrant_data             466G  390G   77G  84% /vagrant_data
tmpfs                     62M     0   62M   0% /run/user/1000
```

接下来需要将未分配的磁盘空间

```
# fdisk /dev/sda
# 依次输入可创建新的分区
n
p
回车
回车

# 继续输入p，可以看到磁盘的情况，多出了/dev/sda3
# /dev/sda3的System为Linux，而/dev/sda2的System为Linux LVM
Command (m for help): p

Disk /dev/sda: 85.9 GB, 85899345920 bytes, 167772160 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0000ca5e

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    20479999     9726976   8e  Linux LVM
/dev/sda3        20480000   167772159    73646080   83  Linux

# 依次输入将/dev/sda3更改为LVM格式
t
3
8e
p

Disk /dev/sda: 85.9 GB, 85899345920 bytes, 167772160 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0000ca5e

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    20479999     9726976   8e  Linux LVM
/dev/sda3        20480000   167772159    73646080   8e  Linux LVM

# 输入w后进行保存操作
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```

## 将新创建的磁盘分区添加到LVM分区中

将机器重启后，继续执行如下命令

```
# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               9.27 GiB
  PE Size               4.00 MiB
  Total PE              2374
  Alloc PE / Size       2364 / 9.23 GiB
  Free  PE / Size       10 / 40.00 MiB
  VG UUID               cpEmYK-XFew-6ZWT-GEeY-yEou-0vLq-OJiD08

# lvscan
  ACTIVE            '/dev/centos/swap' [1000.00 MiB] inherit
  ACTIVE            '/dev/centos/root' [<8.26 GiB] inherit

# pvcreate /dev/sda3
  Physical volume "/dev/sda3" successfully created.

# vgextend centos /dev/sda3
  Volume group "centos" successfully extended

# lvextend /dev/centos/root /dev/sda3
  Size of logical volume centos/root changed from <8.26 GiB (2114 extents) to <78.49 GiB (20093 extents).
  Logical volume centos/root successfully resized.

# xfs_growfs /dev/centos/root
meta-data=/dev/mapper/centos-root isize=256    agcount=4, agsize=541184 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=2164736, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2164736 to 20575232

# 最后执行命令可以看到磁盘空间已经增加
# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   79G  7.9G   71G  11% /
devtmpfs                 296M     0  296M   0% /dev
tmpfs                    307M     0  307M   0% /dev/shm
tmpfs                    307M  4.5M  303M   2% /run
tmpfs                    307M     0  307M   0% /sys/fs/cgroup
/dev/sda1                497M  195M  303M  40% /boot
vagrant                  466G  390G   77G  84% /vagrant
vagrant_data             466G  390G   77G  84% /vagrant_data
tmpfs                     62M     0   62M   0% /run/user/1000
```

