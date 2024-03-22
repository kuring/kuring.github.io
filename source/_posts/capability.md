---
title: linux capability
date: 2018-09-23 21:42:36
tags:
---

传统的unix权限模型将进程分为root用户进程（有效用户id为0）和普通用户进程。普通用户需要root权限的某些功能，通常通过setuid系统调用实现。但普通用户并不需要root的所有权限，可能仅仅需要修改系统时间的权限而已。这种粗放的权限管理方式势必会带来一定的安全隐患。

linux内核中引入了capability，用于消除需要执行某些操作的程序对root权限的依赖。

capability用于分割root用户的权限，将root的权限分割为不同的能力，每一种能力代表一定的特权操作。例如，CAP_SYS_MODULE用于表示用户加载内核模块的特权操作。根据进程具有的能力来进行特权操作的访问控制。

只有进程和可执行文件才有能力，每个进程拥有以下几组能力集(set)。

- cap_effective: 进程当前可用的能力集
- cap_inheritable: 进程可以传递给子进程的能力集
- cap_permitted: 进程可拥有的最大能力集
- cap_ambient: Linux 4.3后引入的能力集，
- cap_bounding: 用于进一步限制能力的获取

可以通过/proc/${pid}/status文件中的CapInh CapPrm CapEff CapBnd CapAmb来表示，每个字段为8个字节即64bit，每个比特表示一种能力，这几个字段存放在进程的内核数据结构task_struct中，由此可见capability的最小单位为线程，而不是进程。

## example 1 设置进程能力

在执行下面程序之前需要安装`yum install libcap-devel`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <unistd.h>

#undef _POSIX_SOURCE
#include <sys/capability.h>

extern int errno;

void whoami(void)
{
    printf("uid=%i  euid=%i  gid=%i\n", getuid(), geteuid(), getgid());
}

void listCaps()
{
    cap_t caps = cap_get_proc();
    ssize_t y = 0;
    printf("The process %d was give capabilities %s\n",(int) getpid(), cap_to_text(caps, &y));
    fflush(0);
    cap_free(caps);
}

int main(int argc, char **argv)
{
    int stat;
    whoami();
    stat = setuid(geteuid());
    pid_t parentPid = getpid();

    if(!parentPid)
    return 1;
    cap_t caps = cap_init();

    // 给进程增加5中能力
    cap_value_t capList[5] ={ CAP_NET_RAW, CAP_NET_BIND_SERVICE , CAP_SETUID, CAP_SETGID,CAP_SETPCAP } ;
    unsigned num_caps = 5;
    cap_set_flag(caps, CAP_EFFECTIVE, num_caps, capList, CAP_SET);
    cap_set_flag(caps, CAP_INHERITABLE, num_caps, capList, CAP_SET);
    cap_set_flag(caps, CAP_PERMITTED, num_caps, capList, CAP_SET);

    if (cap_set_proc(caps)) {
        perror("capset()");

        return EXIT_FAILURE;
    }
    listCaps();

    // 将进程的能力清除
    printf("dropping caps\n");
    cap_clear(caps);  // resetting caps storage
    if (cap_set_proc(caps)) {
        perror("capset()");
        return EXIT_FAILURE;
    }
    listCaps();

    cap_free(caps);
    return 0;
}
```

并执行如下操作：

```shell
gcc capability.c -lcap -o capability

# 需要使用root执行，因为普通用户不能给进程设置能力
sudo ./capability

# 输出如下内容
uid=0  euid=0  gid=0
The process 5044 was give capabilities = cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw+eip
dropping caps
The process 5044 was give capabilities =
```

## example 2 获取进程能力

```c
#undef _POSIX_SOURCE
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <linux/capability.h>
#include <errno.h>

int main()
{
    struct __user_cap_header_struct cap_header_data;
    cap_user_header_t cap_header = &cap_header_data;

    struct __user_cap_data_struct cap_data_data;
    cap_user_data_t cap_data = &cap_data_data;

    cap_header->pid = getpid();
    cap_header->version = _LINUX_CAPABILITY_VERSION_1;

    if (capget(cap_header, cap_data) < 0) {
        perror("Failed capget");
        exit(1);
    }
    printf("Cap data 0x%x, 0x%x, 0x%x\n", cap_data->effective,cap_data->permitted, cap_data->inheritable);
}
```

可以通过capget命令获取进程的能力

```shell
[vagrant@localhost tmp]$ gcc get_capability.c -lcap -o get_capability
# 普通用户默认情况下没有任何能力
[vagrant@localhost tmp]$ ./get_capability
Cap data 0x0, 0x0, 0x0

# root用户默认拥有所有的能力
[vagrant@localhost tmp]$ sudo ./get_capability
Cap data 0xffffffff, 0xffffffff, 0x0
```

## 工具

1. getcap用于获取程序文件所具有的能力。
2. getpcaps用于获取进程所具有的能力。
3. setcap用于设置程序文件所具有的能力。
4. capsh 查看和设置程序的能力

```shell
# 将chown命令授权给普通用户也具备更改文件owner的能力
# 其中eip分别代表cap_effective(e) cap_inheritable(i) cap_permitted(p)
[vagrant@localhost tmp]$ sudo setcap cap_chown=eip /usr/bin/chown
[vagrant@localhost tmp]$ getcap /usr/bin/chown
/usr/bin/chown = cap_chown+eip

# 使用root创建测试文件
[vagrant@localhost tmp]$ sudo touch /tmp/aa
# 普通用户也可以修改root用户创建文件的owner了
[vagrant@localhost tmp]$ chown vagrant:vagrant /tmp/aa

# 清除chown的能力
[vagrant@localhost tmp]$ sudo setcap -r /usr/bin/chown
[vagrant@localhost tmp]$ getcap /usr/bin/chown

# 获取到进程的 capability
[vagrant@localhost tmp]$ cat /proc/95373/status | grep Cap
CapInh:	0000000000000000
CapPrm:	0000000000000000
CapEff:	0000000000000000
CapBnd:	0000003fffffffff
CapAmb:	0000000000000000

# 对上述 capability 进行解码
[vagrant@localhost tmp]$ capsh --decode=0000003fffffffff
0x0000003fffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,35,36,37
```

## runc项目中的应用

runc的容器配置文件`spec.Process.Capabilities`可以定义各个能力集的能力，用来限制容器的能力。

## docker中的应用

docker默认情况下给容器去掉了一些比较危险的capabilities，比如`cap_sys_admin`。

例如在docker中使用gdb命令默认是不允许的，这是因为docker已经将SYS_PTRACE相关的能力给去掉了。

在docker中使用`--cap-add`和`--cap-drop`命令来增加和删除capabilities，

可以使用`--privileged`赋予容器所有的capabilities，该操作谨慎使用。

## ref

* [Linux的capability深入分析(1)](https://blog.csdn.net/wangpengqi/article/details/9821227)
* [Linux的capability深入分析(2)](https://blog.csdn.net/wangpengqi/article/details/9821231)
* [Linux Programmer's Manual CAPABILITIES](http://man7.org/linux/man-pages/man7/capabilities.7.html)
* [如何在Docker内部使用gdb调试器](https://mp.weixin.qq.com/s?__biz=MzI0NjI4MDg5MQ==&mid=2715292188&idx=1&sn=2b7f26203aa594027550e324460bc901&chksm=cd6d15c8fa1a9cde757868fd34c8336433c4877d3e7689ed0a2bd90eb1ef6271bda97aa3bb03&mpshare=1&scene=1&srcid=12045vIwpmKLu97HvFOssitt%23rd)
* [docker run](https://docs.docker.com/engine/reference/commandline/run/)
