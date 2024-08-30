---
title: docker基础知识之mount namespace
date: 2018-09-15 01:52:17
tags:
  - docker
permalink: /namespace_mount/
---
Mount Namespace 是 Linux 下的一种挂载隔离机制，可以实现不同的进程拥有独立的挂载视图，各挂载点之间相互不受影响。
# mount namespace 基本用法

Linux 启动时会创建一个默认的 mount namespace，在使用 clone() 或者 unshare() 系统调用通过 CLONE_NEWNS 会创建出新的 mount namespace。因为是第一个加入到 Linux 内核的中的 namespace，CLONE_NEWNS 的取名并不是太合理。

每个进程的挂载信息保存在了 `/proc/<pid>/{mountinfo,mounts,mountstats}` 中。

使用如下命令可以看到一个进程对应的 mount namespace 名称。
```
#ll /proc/$$/ns/mnt
lrwxrwxrwx 1 root root 0 Aug 30 00:02 /proc/94832/ns/mnt -> mnt:[4026531840]
```

可以使用 unshare 命令来创建一个新的命名空间，执行如下命令：
```
# 创建新的命名空间，此时会继承当前 mount namespace 的挂载点信息
unshare --mount --fork --pid --mount-proc /bin/bash
# 创建新的挂载点并挂载，执行完成后可以看到新的挂载点存在
mkdir /mnt/new_mount
mount -t tmpfs tmpfs /mnt/new_mount
# 查看新的挂载点
df -h | grep new_mount
```

新打开一个终端，切换回原始的命名空间，可以看到挂载点并不存在。

# mount namespace 的 shared subtree

mount namespace 实现了挂载点的隔离，但在很多创建下又希望能够在不同的 namespace 下共享挂载点。[shared subtree](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt) 机制允许在 mount namespace 之间自动的或者受控的传播  mount 和 umount 事件。

> 注意：该处理解起来比较绕。在创建 mount namespace 和挂载目录的时候分别可以指定参数。

在做单个目录挂载的时候通过指定传播类型来指定的不同隔离行为，支持如下的值：
- private：挂载点不会传播到其他命名空间，默认类型。
- shared：挂载点会传播到共享这个挂载点的其他命名空间。
- slave：挂载点不会传播到另一个命名空间，但其他命名空间的传播会影响到这个挂载点。
- unbindable：挂载点不能被移动。
- rprivate：递归地将挂载点及其子挂载点设置为 private。
- rshared：递归地将挂载点及其子挂载点设置为 shared。
- rslave：递归地将挂载点及其子挂载点设置为 slave。

mount 在挂载目录的时候可以指定如下参数：
1. --make-rshared：将挂载方式设置为 rshared。
2. --make-rslave：将挂载方式设置为 rslave。
3. --make-rprivate：将挂载方式设置为 rprivate。
4. --make-runbindable：将挂载方式设置为不可移动。

可以通过 `findmnt -o TARGET,PROPAGATION` 命令查看到当前 mount namespace 下的挂载点的 shared subtree 属性。但该命令无法区分出 shared 和 rshared 属性。

unshare 命令在创建新的 mount namespace 时通过选项 `--propagation`来指定新创建的 mount namespace 中所有挂载点的共享方式。支持如下值：
1. private：新创建的 mount namespace 中的所有挂载点的 shared subtrees 属性全部为 private。默认值。
2. shared：新创建的 mount namespace 中的挂载点的 shared subtrees 属性全部为 share。
3. slave：新创建的 mount namespace 中的挂载点的 shared subtrees 属性全部为 slave。
4. unchanged：新创建的 mount namespace 中的挂载点的 shared subtrees 属性保持不变。

在上面的例子中可以通过指定为 rshared 的模式来查看效果。

```
# 创建新的命名空间
unshare --mount --fork --pid --mount-proc /bin/bash

# 创建新的挂载点并挂载，执行完成后可以看到新的挂载点存在
mount --make-rshared /

# 查看新的挂载点
df -h | grep new_mount
```

# mount namespace 在容器技术中的应用

在容器技术中，在容器中不应该看到宿主机上挂载的目录。在Linux中使用chroot技术来实现，在容器中将`/`挂载到指定目录，这样在容器中就看不到宿主机上的其他挂载项。在容器技术中，chroot所使用的目录即容器镜像的目录。容器镜像中并不包含操作系统的内核。

## Docker Volume

为了解决容器中能够访问宿主机上文件的问题，docker引入了Volume机制，将宿主机上指定的文件或者目录挂载到容器中。而整个的docker Volume机制跟mount namespace的关系不太大。

Volume用到的技术为Linux的绑定挂载机制，该机制将一个指定的目录或者文件挂载到一个指定的目录上。

容器启动顺序如下：

1. 创建新的mount namespace
2. dockerinit根据容器镜像准备好rootfs
3. dockerinit使用绑定挂载机制将一个指定的目录挂载到rootfs的某个目录上
4. dockerinit调用chroot

容器启动时需要创建新的mount namespace，根据容器镜像准备好rootfs，调用chroot。docker volume的挂载时机是在rootfs准备好之后，调用chroot之前完成。

上文提到进入新的mount namespace后，mount namespace会继承父mount namespace的挂载, docker volume一定是在新的mount namespce中执行，否则会影响到宿主机上的mount。在调用chroot之后已经看不到宿主机上的文件系统，无法进行挂载。

执行这一操作的进程为docker的容器进程dockerinit，该进程会负责完成根目录的准备、挂载设备和目录、配置hostname等一系列需要在容器内进行的初始化操作。在初始化完成后，会调用execv()系统调用，用容器中的ENTRYPOINT进程取代，成为容器中的1号进程。

volume挂载的目录是挂载在读写层，由于使用了mount namespace，在宿主机上看不到挂载目录的信息，因此docker commit操作不会将挂载的目录提交。

下面使用例子来演示docker volume的用法

在宿主机上使用`docker run -d -v /test ubuntu sleep 10000`创建新的容器，并创建docker容器中的挂载点/test，该命令会自动在容器中创建目录，并将宿主机上指定目录下的随机目录挂载到容器中的/test目录下。

可在宿主机上通过如下命令查看到volume的情况

```
# 列出当前docker在使用的所有volume
[root@localhost vagrant]# docker volume ls
DRIVER              VOLUME NAME
local               4ad97e6356707b66cd1cacc4a2e223d9c79d11eca26fe12b1becc9dd664fc5c6

# 查看volume在宿主机上的挂载点
[root@localhost vagrant]# docker volume inspect 4ad97e6356707b66cd1cacc4a2e223d9c79d11eca26fe12b1becc9dd664fc5c6
[
    {
        "CreatedAt": "2018-09-15T23:36:09+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/4ad97e6356707b66cd1cacc4a2e223d9c79d11eca26fe12b1becc9dd664fc5c6/_data",
        "Name": "4ad97e6356707b66cd1cacc4a2e223d9c79d11eca26fe12b1becc9dd664fc5c6",
        "Options": {},
        "Scope": "local"
    }
]
```

## docker文件系统

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/docker_fs.png)

rootfs在最下层为docker镜像的只读层。

rootfs之上为dockerinit进程自己添加的init层，用来存放dockerinit添加或者修改的/etc/hostname等文件。

rootfs的最上层为可读写层，以Copy-On-Write的方式存放任何对只读层的修改，容器声明的volume挂载点也出现这一层。
# mount namespace 在 k8s 中应用
在 k8s 中可以通过 volumeMounts 中的参数 mountPropagation 来指定挂载的参数。

## 引用
- 极客时间-深入剖析Kubernetes-08
