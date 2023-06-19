---
title: docker基础知识之mount namespace
date: 2018-09-15 01:52:17
tags: docker
---

在使用CLONE_NEWNS来创建新的mount namespace时，子进程会共享父进程的文件系统，如果子进程执行了新的mount操作，仅会影响到子进程自身，不会对父进程造成影响。

```
#define _GNU_SOURCE
#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <sched.h>
#include <signal.h>
#include <unistd.h>

/* 定义一个给 clone 用的栈，栈大小1M */
#define STACK_SIZE (1024 * 1024)
static char container_stack[STACK_SIZE];

char* const container_args[] = {
    "/bin/bash",
    NULL
};

int container_main(void* arg)
{
    printf("Container [%5d] - inside the container!\n", getpid());
    sethostname("container",10);
    /* 重新mount proc文件系统到 /proc下 */
    system("mount -t proc proc /proc");
    execv(container_args[0], container_args);
    printf("Something's wrong!\n");
    return 1;
}

int main()
{
    printf("Parent - start a container!\n");
    int container_pid = clone(container_main, container_stack+STACK_SIZE,
            CLONE_NEWUTS | CLONE_NEWPID | CLONE_NEWNS | SIGCHLD, NULL); /*启用CLONE_NEWUTS Namespace隔离 */
    waitpid(container_pid, NULL, 0);
    printf("Parent - container stopped!\n");
    return 0;
}
```

执行效果：

```
[root@centos7 docker_learn]# ./mount
Parent - start a container!
Container [    1] - inside the container!

# 由于ps是读取的/proc目录下的文件来显示当前系统系统的进程，新进程挂载/proc目录到新的proc文件系统，自然看不到父进程的/proc目录了
[root@container docker_learn]# ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 22:20 pts/3    00:00:00 /bin/bash
root         30      1  0 22:20 pts/3    00:00:00 ps -ef
```

但是在容器技术中，在容器中不应该看到宿主机上挂载的目录。在Linux中使用chroot技术来实现，在容器中将`/`挂载到指定目录，这样在容器中就看不到宿主机上的其他挂载项。在容器技术中，chroot所使用的目录即容器镜像的目录。容器镜像中并不包含操作系统的内核。

mount namespace是Linux中第一个namespace，是基于chroot技术改良而来。

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

## ref

极客时间-深入剖析Kubernetes-08
