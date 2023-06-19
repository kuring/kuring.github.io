---
title: 理解OverlayFS
date: 2018-07-28 21:30:25
tags: overlay overlay2
---

## docker image结构

docker可以通过命令`docker image inspect ${image}`来查看image的详细信息，其中包含了所使用的底层文件系统及各层的信息。

docker container的存储结构分为了只读层、init层和可读写层。

只读层跟docker image的层次结构恰好对应，主要包含操作系统的文件、配置、目录等信息，不包含操作系统镜像。

init层在只读层和读写层中间，用来专门存放/etc/hosts /etc/resolv.conf等信息，这些文件往往需要在启动的时候写入一些指定值，但不期望`docker commit`命令对其进行提交。

可读写层为容器在运行中可以进行写入的层。

## overlay

![image](https://arkingc.github.io/img/in-post/post-docker-filesystem/overlay_constructs.jpg)

采用了两层结构，lowerdir为镜像层，只读。upperdir为容器层。

每层都会在/var/run/docker/overlay创建一个文件夹，文件夹中为实际层的内容，文件采用硬链接的方式链接到真实层中的文件，每一层都包含该层该拥有的所有文件，而该文件的真实存储可能是采用硬链接的方式链接到上层中的真实文件，因此比较耗费inode。

创建一个容器时，会新增两个目录，一个为读写层，一个为初始层。初始层中保存了容器初始化时的环境信息，如hostname、hosts文件等。读写层用于记录容器的所有改动。

## overlay2

为了规避overlay消耗inode节点过多的问题，overlay2采用在每层中增加lower文件的方式来记录所有底层的信息，类似于链表的形式。

`docker pull ubuntu`

```
[root@localhost runc]# docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
c64513b74145: Pull complete
01b8b12bad90: Pull complete
c5d85cf7a05f: Pull complete
b6b268720157: Pull complete
e12192999ff1: Pull complete
Digest: sha256:3f119dc0737f57f704ebecac8a6d8477b0f6ca1ca0332c7ee1395ed2c6a82be7
Status: Downloaded newer image for ubuntu:latest
```

会在/var/run/docker/overlay2目录下创建如下文件：

```
[root@localhost overlay2]# tree -L 2
.
|-- 664ae13f1c21402385076025d68476eb8d1cc4be6c6a218b24bd55217ac62672    // 第0层
|   |-- diff
|   `-- link    // MZUEUOFHBNVTRCJYJEG7QY4VWT
|-- 783ad02709b67ac47b55198e9659c4592f0972334987ab97f42fd10f1784cbba    // 第2层
|   |-- diff
|   |-- link    // HXATFASQ4E2JBG434DUEN54EZZ
|   |-- lower   // l/WUZC5WSTQTPJUJ4KFAYCUT5IPD:l/MZUEUOFHBNVTRCJYJEG7QY4VWT
|   |-- merged
|   `-- work
|-- 89f7a20dda3d868840e20d9e8f1bfe20c5cca51c27b07825f100da0f474672f6    // 第3层
|   |-- diff
|   |-- link    // 5PHT7S3MCZTTQOXVPA4CKJRRFD
|   |-- lower   // l/HXATFASQ4E2JBG434DUEN54EZZ:l/WUZC5WSTQTPJUJ4KFAYCUT5IPD:l/MZUEUOFHBNVTRCJYJEG7QY4VWT
|   |-- merged
|   `-- work
|-- backingFsBlockDev
|-- bad073a2d1f79a03af6caa0b3f51a22e6762cebbc0c30e45458fe6c1ff266f68    // 第4层
|   |-- diff
|   |-- link    // QMKHIPSDT4JTPE4FLT7QGJ33ND
|   |-- lower   // l/5PHT7S3MCZTTQOXVPA4CKJRRFD:l/HXATFASQ4E2JBG434DUEN54EZZ:l/WUZC5WSTQTPJUJ4KFAYCUT5IPD:l/MZUEUOFHBNVTRCJYJEG7QY4VWT
|   |-- merged
|   `-- work
|-- cb40b5b47c699050305676b35b1cea1ce08b38604dd68243c4be48934125b1a3    // 第1层
|   |-- diff
|   |-- link    // WUZC5WSTQTPJUJ4KFAYCUT5IPD
|   |-- lower   // l/MZUEUOFHBNVTRCJYJEG7QY4VWT
|   |-- merged
|   `-- work
`-- l
    |-- 5PHT7S3MCZTTQOXVPA4CKJRRFD -> ../89f7a20dda3d868840e20d9e8f1bfe20c5cca51c27b07825f100da0f474672f6/diff
    |-- HXATFASQ4E2JBG434DUEN54EZZ -> ../783ad02709b67ac47b55198e9659c4592f0972334987ab97f42fd10f1784cbba/diff
    |-- MZUEUOFHBNVTRCJYJEG7QY4VWT -> ../664ae13f1c21402385076025d68476eb8d1cc4be6c6a218b24bd55217ac62672/diff
    |-- QMKHIPSDT4JTPE4FLT7QGJ33ND -> ../bad073a2d1f79a03af6caa0b3f51a22e6762cebbc0c30e45458fe6c1ff266f68/diff
    `-- WUZC5WSTQTPJUJ4KFAYCUT5IPD -> ../cb40b5b47c699050305676b35b1cea1ce08b38604dd68243c4be48934125b1a3/diff
```

l目录下为超链接，缩短后的目录，为了避免mount时超出页大小限制。

每一层中的diff文件夹包含实际内容。

每一层中都有一个link文件，内容为l目录中的超链接，超链接实际指向当前层目录中的diff文件夹。

除去最底层的目录外，其余每一层中包含一个lower文件，包含了该层的所有更底层名称和顺序，可以根据该文件构建出整个镜像的层次结构。

work目录用于OverlayFS内部使用。

最底层只有link文件，无lower文件，因此664ae13f1c21402385076025d68476eb8d1cc4be6c6a218b24bd55217ac62672为最底层。

以上五层为lower，只读。

当使用`docker run -it ubuntu:latest /bin/bash`启动一个容器后，在overlay2目录下会多出两个文件夹。

```
[root@localhost overlay2]# tree -L 1 0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08-init 0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08 l
0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08-init
|-- diff
|-- link    // ZJVMGTB2IOJ6QF57TYM5O7EWXW
|-- lower   // l/QMKHIPSDT4JTPE4FLT7QGJ33ND:l/5PHT7S3MCZTTQOXVPA4CKJRRFD:l/HXATFASQ4E2JBG434DUEN54EZZ:l/WUZC5WSTQTPJUJ4KFAYCUT5IPD:l/MZUEUOFHBNVTRCJYJEG7QY4VWT
|-- merged
`-- work
0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08
|-- diff
|-- link
|-- lower   // l/ZJVMGTB2IOJ6QF57TYM5O7EWXW:l/QMKHIPSDT4JTPE4FLT7QGJ33ND:l/5PHT7S3MCZTTQOXVPA4CKJRRFD:l/HXATFASQ4E2JBG434DUEN54EZZ:l/WUZC5WSTQTPJUJ4KFAYCUT5IPD:l/MZUEUOFHBNVTRCJYJEG7QY4VWT
|-- merged
`-- work
l
|-- 5PHT7S3MCZTTQOXVPA4CKJRRFD -> ../89f7a20dda3d868840e20d9e8f1bfe20c5cca51c27b07825f100da0f474672f6/diff
|-- AOLYGFOHIAHWU5CBAJFULNAXI7 -> ../0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08/diff
|-- HXATFASQ4E2JBG434DUEN54EZZ -> ../783ad02709b67ac47b55198e9659c4592f0972334987ab97f42fd10f1784cbba/diff
|-- MZUEUOFHBNVTRCJYJEG7QY4VWT -> ../664ae13f1c21402385076025d68476eb8d1cc4be6c6a218b24bd55217ac62672/diff
|-- QMKHIPSDT4JTPE4FLT7QGJ33ND -> ../bad073a2d1f79a03af6caa0b3f51a22e6762cebbc0c30e45458fe6c1ff266f68/diff
|-- WUZC5WSTQTPJUJ4KFAYCUT5IPD -> ../cb40b5b47c699050305676b35b1cea1ce08b38604dd68243c4be48934125b1a3/diff
`-- ZJVMGTB2IOJ6QF57TYM5O7EWXW -> ../0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08-init/diff
```

`0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08-init`用于存放容器初始化时的信息，通过下面查看更直观。

```
[root@localhost overlay2]# tree 0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08-init
0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08-init
|-- diff
|   |-- dev
|   |   `-- console
|   `-- etc
|       |-- hostname
|       |-- hosts
|       |-- mtab -> /proc/mounts
|       `-- resolv.conf
|-- link
|-- lower
|-- merged
`-- work
    `-- work
```

`0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08`的直接底层为init层，更详细的目录结构如下。

```
[root@localhost overlay2]# tree -L 2  0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08
0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08
|-- diff
|-- link
|-- lower
|-- merged
|   |-- bin
|   |-- boot
|   |-- dev
|   |-- etc
|   |-- home
|   |-- lib
|   |-- lib64
|   |-- media
|   |-- mnt
|   |-- opt
|   |-- proc
|   |-- root
|   |-- run
|   |-- sbin
|   |-- srv
|   |-- sys
|   |-- tmp
|   |-- usr
|   `-- var
`-- work
    `-- work
```

merged文件夹中内容较多，为overlay2的直接挂载点，对容器的修改会反应到该目录中。例如在容器中增加/root/hello.txt文件，在merged目录下会增加root/hello.txt文件。

```
[root@localhost overlay2]# mount  | grep overlay2
/dev/mapper/centos-root on /var/lib/docker/overlay2 type xfs (rw,relatime,attr2,inode64,noquota)
overlay on /var/lib/docker/overlay2/0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08/merged type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/ZJVMGTB2IOJ6QF57TYM5O7EWXW:/var/lib/docker/overlay2/l/QMKHIPSDT4JTPE4FLT7QGJ33ND:/var/lib/docker/overlay2/l/5PHT7S3MCZTTQOXVPA4CKJRRFD:/var/lib/docker/overlay2/l/HXATFASQ4E2JBG434DUEN54EZZ:/var/lib/docker/overlay2/l/WUZC5WSTQTPJUJ4KFAYCUT5IPD:/var/lib/docker/overlay2/l/MZUEUOFHBNVTRCJYJEG7QY4VWT,upperdir=/var/lib/docker/overlay2/0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08/diff,workdir=/var/lib/docker/overlay2/0326c1da0af912a6ea5efda77b65b04e796993e0f111ed8f262c55b2716f1c08/work)
```

## ref

- [Use the OverlayFS storage driver](https://docs.docker.com/storage/storagedriver/overlayfs-driver/)
- [Docker存储驱动—Overlay/Overlay2「译」](https://arkingc.github.io/2017/05/05/docker-filesystem-overlay/)
