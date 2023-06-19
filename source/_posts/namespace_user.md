---
title: docker基础知识之user namespace
tags: docker
date: 2018-09-23 19:13:16
---


user namespace是所有namespace中实现最复杂的一个，也是最晚引入的一个，是在linux2.6版本中才引入。因为涉及到权限机制，跟capability有着比较密切的关系。

## 创建user namespace

在clone或者unshare系统调用使用CLONE_NEWUSER参数后，在子进程中看到的uid和gid跟父进程中的不一样，子进程中找不到uid时，会显示最大的uid 65534（在/proc/sys/kernel/overflowuid中设置）。

```
vagrant@ubuntu-xenial:/tmp$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant)
vagrant@ubuntu-xenial:/tmp$ readlink /proc/$$/ns/user
user:[4026531837]

# 使用unshare命令创建新的user namespace
vagrant@ubuntu-xenial:/tmp$ unshare --user /bin/bash
nobody@ubuntu-xenial:/tmp$ readlink /proc/$$/ns/user
user:[4026532145]
# 新的user namespace没有映射关系，默认使用/proc/sys/kernel/overflowuid中定义的user id和/proc/sys/kernel/overflowgid中的group id
nobody@ubuntu-xenial:/tmp$ id
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)

#--------------------------第二个shell窗口----------------------
# 在父user namespace上创建文件夹，可以看到用户为vagrant
vagrant@ubuntu-xenial:/tmp$ mkdir test
vagrant@ubuntu-xenial:/tmp$ ll /tmp | grep test
drwxrwxr-x  2 vagrant vagrant  4096 Sep 23 17:11 test/

#--------------------------第一个shell窗口----------------------
# 在新创建的user namespace下用户显示为nobody
nobody@ubuntu-xenial:/tmp$ ll /tmp | grep test
drwxrwxr-x  2 nobody nogroup  4096 Sep 23 17:11 test/
```

## 映射user id和group id到新的user namespace

创建完新的user namespace后，通常会先映射user id和group id，方法为添加映射关系到/proc/${pid}/uid_map和/proc/${pid}/gid_map中。

user namespace被创建以后，第一个进程被赋予了该namespace的所有权限，但该进程并不拥有父namespace的任何权限。利用该机制可以做到一个用户在父user namespace中是普通用户，在子user namespace中是超级用户的功能。

为了将容器中的uid和父user namespace上的uid和gid进行 关联起来，可通过/proc/<pid>/uid_map和/proc/<pid>/gid_map来进行映射的。这两个文件的格式为：

```
ID-inside-ns ID-outside-ns length
```

第一个字段ID-inside-ns表示在容器显示的UID或GID，
第二个字段ID-outside-ns表示容器外映射的真实的UID或GID。
第三个字段表示映射的范围，一般填1，表示一一对应。

`0 1000 256`这个配置的含义为父user namespace的1000-1256映射到新user namespace的0-256.

创建子进程在没有指定CLONE_NEWUSER时文件内容如下，子进程跟父进程的用户完全一致：

```
# 表示把namespace内部从0开始的uid映射到外部从0开始的uid，其最大范围是无符号32位整形
[root@centos7 1325]# cat uid_map
         0          0 4294967295
```

要想实现以普通用户运行程序，在子进程中以root用户执行，仅需要将uid_map文件修改为普通用户映射到子进程中的0即可，因为uid为0表示root用户。

那么谁拥有写入该文件的权限呢？

/proc/${pid}/[u|g]id的拥有者为创建新user namespace的用户，拥有map文件写入权限的仅有两个用户：和该用户在同一个user namespace中的root用户，创建新的user namespace的用户。创建新的user namespace有没有写入map文件的权限，还要取决于capability中的CAP_SETUID和CAP_SETGID两个权限。

为了方便写入/proc/${pid}/uid_map和/proc/${pid}/gid_map文件，可以使用newuidmap和newgidmap命令来完成。

继续上述例子

```
#--------------------------第一个shell窗口----------------------
# 在新user namespace中获取当前的进程号
nobody@ubuntu-xenial:/tmp$ echo $$
13226

#--------------------------第二个shell窗口----------------------
vagrant@ubuntu-xenial:/tmp$ ll /proc/13226/uid_map  /proc/13226/gid_map
-rw-r--r-- 1 vagrant vagrant 0 Sep 23 17:31 /proc/13226/gid_map
-rw-r--r-- 1 vagrant vagrant 0 Sep 23 17:31 /proc/13226/uid_map
# 提示当前进程没有权限写入
vagrant@ubuntu-xenial:/tmp$ echo '0 1000 100' > /proc/13226/uid_map
-bash: echo: write error: Operation not permitted

# 查看当前bash没有任何capability
vagrant@ubuntu-xenial:/tmp$ cat /proc/$$/status | egrep 'Cap(Inh|Prm|Eff)'
CapInh:	0000000000000000
CapPrm:	0000000000000000
CapEff:	0000000000000000

# 使用root权限给/bin/bash可执行文件增加cap_setgid和cap_setuid
vagrant@ubuntu-xenial:/tmp$ sudo setcap cap_setgid,cap_setuid+ep /bin/bash
# 启动新的bash后capability会生效
vagrant@ubuntu-xenial:/tmp$ bash
vagrant@ubuntu-xenial:/tmp$ cat /proc/$$/status | egrep 'Cap(Inh|Prm|Eff)'
CapInh:	0000000000000000
CapPrm:	00000000000000c0
CapEff:	00000000000000c0

# 重新写入
vagrant@ubuntu-xenial:/tmp$ echo '0 1000 100' > /proc/13226/uid_map
vagrant@ubuntu-xenial:/tmp$ echo '0 1000 100' > /proc/13226/gid_map

# 第二次再写入会失败，仅允许写入一次
vagrant@ubuntu-xenial:/tmp$ echo '0 1000 100' > /proc/13226/uid_map
bash: echo: write error: Operation not permitted

# 将刚才设置的capability取消
vagrant@ubuntu-xenial:/tmp$ sudo setcap cap_setgid,cap_setuid-ep /bin/bash
vagrant@ubuntu-xenial:/tmp$ getcap /bin/bash
/bin/bash =
vagrant@ubuntu-xenial:/tmp$ exit
exit

#--------------------------第一个shell窗口----------------------
# 第一个窗口中userid已经变更为0了
nobody@ubuntu-xenial:/tmp$ id
uid=0(root) gid=0(root) groups=0(root)
# 重新执行一个新的bash，会发现提示符已经变更为root了
nobody@ubuntu-xenial:/tmp$ bash
root@ubuntu-xenial:/tmp#

# 可以看到新的bash已经拥有的所有的capability，但也仅限于当前的user namespace中
root@ubuntu-xenial:/tmp# cat /proc/$$/status | egrep 'Cap(Inh|Prm|Eff)'
CapInh:	0000000000000000
CapPrm:	0000003fffffffff
CapEff:	0000003fffffffff
```

## 问题

user namespace在linux3.8内核版本上才实现，存在一定的安全问题。在redhat和centos系统下，user namespace作为了一个实验feature，默认情况下未开启。

执行如下命令`sudo grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"`并重启系统后可以就可以打开user namespace feature了。执行`grubby --remove-args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"`可关闭user namespace feature。

经实验，上述操作未生效，后续待查该问题。

## ref

- [What’s Next for Containers? User Namespaces
](https://rhelblog.redhat.com/2015/07/07/whats-next-for-containers-user-namespaces/#comment-6796)
- [Linux Namespace系列（07）：user namespace (CLONE_NEWUSER) (第一部分)](https://segmentfault.com/a/1190000006913195)
