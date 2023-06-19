---
title: lvm
date: 2021-11-28 02:13:14
tags: 磁盘
---

# 概念

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/lvm.png)

1. 物理卷PV(Physical Volume)：可以是整块物理磁盘或者物理磁盘上的分区
2. 卷组VG(Volume Group)：由一个或多个物理卷PV组成，可在卷组上创建一个或者多个LV，可以动态增加pv到卷组
3. 逻辑卷LV(Logical Volume)：类似于磁盘分区，建立在VG之上，在LV上可以创建文件系统 逻辑卷建立后可以动态的增加或缩小空间
4. PE(Physical Extent): PV可被划分为PE的基本单元，具有唯一编号的PE是可以被LVM寻址的最小单元。PE的大小是可以配置的，默认为4MB。
5. LE(Logical Extent): LV可被划分为LE的基本单元，LE跟PE是一对一的关系。

# 基本操作

lvm相关的目录如果没有按照，在centos下使用`yum install lvm2`进行安装。

初始磁盘状态如下，/dev/sda上有40g磁盘空间未分配：

```
# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        40G  2.8G   38G   7% /
devtmpfs        236M     0  236M   0% /dev
tmpfs           244M     0  244M   0% /dev/shm
tmpfs           244M  4.5M  240M   2% /run
tmpfs           244M     0  244M   0% /sys/fs/cgroup
tmpfs            49M     0   49M   0% /run/user/1000

# fdisk -l

Disk /dev/sda: 85.9 GB, 85899345920 bytes, 167772160 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000a05f8

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048    83886079    41942016   83  Linux
```

对磁盘的操作

```
# fdisk /dev/sda
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

# 创建主分区2，空间为1g，磁盘格式为lvm
Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (2-4, default 2): 2
First sector (83886080-167772159, default 83886080):
Using default value 83886080
Last sector, +sectors or +size{K,M,G} (83886080-167772159, default 167772159): +1G
Partition 2 of type Linux and of size 1 GiB is set

Command (m for help): t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

# 创建主分区3，空间为5g，磁盘格式为lvm
Command (m for help): n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
Partition number (3,4, default 3): 3
First sector (85983232-167772159, default 85983232):
Using default value 85983232
Last sector, +sectors or +size{K,M,G} (85983232-167772159, default 167772159): +5G
Partition 3 of type Linux and of size 5 GiB is set

Command (m for help): t
Partition number (1-3, default 3): 3
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

# 保存上述操作
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```

当前磁盘空间状态如下：

```
# fdisk -l

Disk /dev/sda: 85.9 GB, 85899345920 bytes, 167772160 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000a05f8

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048    83886079    41942016   83  Linux
/dev/sda2        83886080    85983231     1048576   8e  Linux LVM
/dev/sda3        85983232    96468991     5242880   8e  Linux LVM
```

将上述两个lvm磁盘分区创建pv，使用`pvremove /dev/sda2`可以删除pv

```
# pvcreate /dev/sda2 /dev/sda3
  Physical volume "/dev/sda2" successfully created.
  Physical volume "/dev/sda3" successfully created.
[root@localhost vagrant]# pvdisplay
  "/dev/sda2" is a new physical volume of "1.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sda2
  VG Name
  PV Size               1.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               MgarF2-Ka6D-blDi-8ecd-1SEU-y2GD-JtiK2c

  "/dev/sda3" is a new physical volume of "5.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sda3
  VG Name
  PV Size               5.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               ToKp2T-30mS-0P8c-ZrcB-2lO4-ayo9-62fuDx
```

接下来创建vg，vg的名字可以随便定义，并将创建的两个pv都添加到vg中，可以看到vg的空间为两个pv之和

```
# vgcreate vg1 /dev/sda1
# vgdisplay -v
  --- Volume group ---
  VG Name               vg1
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               5.99 GiB
  PE Size               4.00 MiB
  Total PE              1534
  Alloc PE / Size       0 / 0
  Free  PE / Size       1534 / 5.99 GiB
  VG UUID               XI8Biv-JtUv-tsur-wuvm-IJQz-HLZu-6a2u5G

  --- Physical volumes ---
  PV Name               /dev/sda2
  PV UUID               MgarF2-Ka6D-blDi-8ecd-1SEU-y2GD-JtiK2c
  PV Status             allocatable
  Total PE / Free PE    255 / 255

  PV Name               /dev/sda3
  PV UUID               ToKp2T-30mS-0P8c-ZrcB-2lO4-ayo9-62fuDx
  PV Status             allocatable
  Total PE / Free PE    1279 / 1279
```

接下来创建lv，并从vg1中分配空间2g

```
# lvcreate -L 2G -n lv1 vg1
  Logical volume "lv1" created.

# lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg1/lv1
  LV Name                lv1
  VG Name                vg1
  LV UUID                EesY4i-lSqY-ef1R-599C-XTrZ-IcVL-P7W46Q
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2019-06-16 07:29:38 +0000
  LV Status              available
  # open                 0
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
```

接下来给lv1格式化磁盘格式为ext4，并将磁盘挂载到/tmp/lvm目录下

```
# mkfs.ext4 /dev/vg1/lv1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
131072 inodes, 524288 blocks
26214 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=536870912
16 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

# mkdir /tmp/lvm
[root@localhost vagrant]# mount /dev/vg1/lv1 /tmp/lvm
[root@localhost vagrant]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/sda1             40G  2.9G   38G   8% /
devtmpfs             236M     0  236M   0% /dev
tmpfs                244M     0  244M   0% /dev/shm
tmpfs                244M  4.5M  240M   2% /run
tmpfs                244M     0  244M   0% /sys/fs/cgroup
tmpfs                 49M     0   49M   0% /run/user/1000
/dev/mapper/vg1-lv1  2.0G  6.0M  1.8G   1% /tmp/lvm
```

接下来对lv的空间从2G扩展到3G，此时通过df查看分区空间大小仍然为2g，需要执行`resize2fs`命令

```
# lvextend -L +1G /dev/vg1/lv1
  Size of logical volume vg1/lv1 changed from 2.00 GiB (512 extents) to 3.00 GiB (768 extents).
  Logical volume vg1/lv1 successfully resized.

# resize2fs /dev/vg1/lv1
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/vg1/lv1 is mounted on /tmp/lvm; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/vg1/lv1 is now 786432 blocks long.

# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/sda1             40G  2.9G   38G   8% /
devtmpfs             236M     0  236M   0% /dev
tmpfs                244M     0  244M   0% /dev/shm
tmpfs                244M  4.5M  240M   2% /run
tmpfs                244M     0  244M   0% /sys/fs/cgroup
tmpfs                 49M     0   49M   0% /run/user/1000
/dev/mapper/vg1-lv1  2.9G  6.0M  2.8G   1% /tmp/lvm
```

## 问题

试验时操作失误，出现了先格式化磁盘，后发现pv找不到对应设备。vg删除不成功。

正常删除vg的方式，此时lv会自动消失

```
vgreduce --removemissing vg1
vgremove vg1
```

lvremove操作执行的时候经常会出现提示“Logical volume xx contains a filesystem in use.”的情况，该问题一般是由于有其他进程在使用该文件系统导致的。网络上经常看到的是通过fuser或者lsof命令来查找使用方，但偶尔该命令会失效，尤其在本机上有容器的场景下。另外一个可行的办法是通过 `grep -nr "/data" /proc/*/mount` 命令，可以找到挂载该目录的所有进程，简单有效。

# 三种Logic Volume

LVM的机制可以类比于RAID，RAID一个核心的机制是性能和数据冗余，并提供了多种数据的冗余模块可供配置。lvm在性能和数据冗余方面支持如下三种Logic Volume: 线性逻辑卷、条带化逻辑卷和镜像逻辑卷。上述几种模式是在vg已经创建完成后创建lv的时候指定的模式，会影响到lv中的pe分配。

下面使用如下的机器配置来进行各个模式的介绍和功能测试。
```
$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda    253:0    0   40G  0 disk
└─vda1 253:1    0   40G  0 part /
vdb    253:16   0  100G  0 disk
vdc    253:32   0  200G  0 disk
vdd    253:48   0  300G  0 disk
vde    253:64   0  400G  0 disk
```

## 线性逻辑卷 Linear Logic Volume

当一个VG中有两个或者多个磁盘的时候，LV分配磁盘容量的时候是按照VG中的PV安装顺序分配的，即一个PV用完后才会分配下一块PV。也可以在创建LV的通过指定PV中的PE段来将数据分散到多个PV上。该模式也是LVM的默认模式。

使用/dev/vdb和/dev/vdc两块磁盘来进行测试，先创建对应的PV。

```
$ pvcreate /dev/vdb /dev/vdc
  Physical volume "/dev/vdb" successfully created.
  Physical volume "/dev/vdc" successfully created.

$ pvdisplay
  "/dev/vdb" is a new physical volume of "100.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/vdb
  VG Name               
  PV Size               100.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               8NnlYc-4f3f-fkeW-a3l3-LXoC-9UEH-fvpb5V

  "/dev/vdc" is a new physical volume of "200.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/vdc
  VG Name               
  PV Size               200.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               I9ffpN-c1vc-PQOB-yKyd-MdzO-Ngff-6e116t
```

创建VG vg1，该容量为上面两个磁盘空间之和
```
$ vgcreate vg1 /dev/vdb /dev/vdc
  Volume group "vg1" successfully created

$ vgdisplay
  --- Volume group ---
  VG Name               vg1
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               299.99 GiB
  PE Size               4.00 MiB
  Total PE              76798
  Alloc PE / Size       0 / 0   
  Free  PE / Size       76798 / 299.99 GiB
  VG UUID               Gi0bJx-jqY8-YpSo-kB0l-9wdk-ZfCT-GpgFZY
```

从vg1中创建LV lv1，其大小为vg的全部大小

```
$ lvcreate --type linear -l 100%VG -n lv1 vg1
  Logical volume "lv1" created.

$ lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg1/lv1
  LV Name                lv1
  VG Name                vg1
  LV UUID                JQZ193-dz6A-I0Ue-rTKC-6XrQ-gb1F-Qy9kDl
  LV Write Access        read/write
  LV Creation host, time iZt4nd6wiprf8foracovwqZ, 2022-01-08 23:28:45 +0800
  LV Status              available
  # open                 0
  LV Size                299.99 GiB
  Current LE             76798
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0

$ lvs -o lv_name,lv_attr,lv_size,seg_pe_ranges
  LV   Attr       LSize   PE Ranges       
  lv1  -wi-a----- 299.99g /dev/vdc:0-51198
  lv1  -wi-a----- 299.99g /dev/vdb:0-25598
```

## 条带化逻辑卷 Striped Logic Volume

类似于raid0模式，在该模式下，多块磁盘均会分配PE给LV。可以通过-i参数来指定可以使用VG中多少个PV。

优点：将数据的读写压力分散到了多个磁盘，可以提升读写性能。
缺点：一个磁盘损坏后会导致数据丢失。

但在使用striped模式时，需要注意：
1. 如果PV来自于同一个磁盘的不同分区，会导致更多的随机读写，不仅不能提升磁盘性能，反而会导致性能下降。
2. 如果VG中某一个PV过小，则无法将所有的PV平均使用起来，存在木桶效应。

使用`--type striped`来指定为striped模式，`--stripes`来指定需要使用的pv数量，`--stripesize`指定写足够数量的数据后再更换为另外一个pv。在下面的创建命令中，可以看到创建出来了12800个LE，PE是平均分配到了两个pv上。

```
$ lvcreate --type striped --stripes 2 --stripesize 32k -L 50G -n lv1 vg1
  Logical volume "lv1" created.

$ lvs -o lv_name,lv_attr,lv_size,seg_pe_ranges                          
  LV   Attr       LSize  PE Ranges                      
  lv1  -wi-a----- 50.00g /dev/vdb:0-6399 /dev/vdc:0-6399

$ lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg1/lv1
  LV Name                lv1
  VG Name                vg1
  LV UUID                FnOmCM-IkuE-3Rvv-fQqk-Cv1Q-lb8S-l5X7O3
  LV Write Access        read/write
  LV Creation host, time iZt4nd6wiprf8foracovwqZ, 2022-01-09 00:07:08 +0800
  LV Status              available
  # open                 0
  LV Size                50.00 GiB
  Current LE             12800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
```

如果指定的lv大小为250G，由于分配到两块磁盘上，由于最小的磁盘只有100G，100G * 2无法满足250G的磁盘大小需求，此时会报错。

```
lvcreate --type striped --stripes 2 --stripesize 32k -L 250G -n lv1 vg1
  Insufficient suitable allocatable extents for logical volume lv1: 12802 more required
```

## 镜像逻辑卷 Mirror Logic Volume

类似于raid1。可以解决磁盘的单点问题，一块磁盘挂掉后不至于丢失数据。

通过使用`--type mirror`来指定为镜像模式，-m参数来指定冗余的数量。

# ref

- [Linux LVM简明教程](https://linux.cn/article-3218-1.html)
- [初识LVM及ECS上LVM分区扩容](https://yq.aliyun.com/articles/572204)
- [Linux LVM--三种Logic Volume](https://www.cnblogs.com/xibuhaohao/p/11731699.html)
