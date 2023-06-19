---
title: Linux Seccomp
date: 2018-12-08 17:21:40
tags:
---

seccomp是secure computing mode的缩写，是Linux内核中的一个安全计算工具，机制用于限制应用程序可以使用的系统调用，增加系统的安全性。可以理解为系统调用的防火墙，利用BPF来规律系统调用。

在/proc/${pid}/status文件中的Seccomp字段可以看到进程的Seccomp。

## prctl

下面程序使用prctl来设置程序的seccomp为strict模式，仅允许read、write、_exit和sigreturn四个系统调用。当调用未在seccomp白名单中的系统调用后，应用程序会被kill。

```c
#include <stdio.h>         /* printf */
#include <sys/prctl.h>     /* prctl */
#include <linux/seccomp.h> /* seccomp's constants */
#include <unistd.h>        /* dup2: just for test */

int main() {
  printf("step 1: unrestricted\n");

  // Enable filtering
  prctl(PR_SET_SECCOMP, SECCOMP_MODE_STRICT);
  printf("step 2: only 'read', 'write', '_exit' and 'sigreturn' syscalls\n");

  // Redirect stderr to stdout
  dup2(1, 2);
  printf("step 3: !! YOU SHOULD NOT SEE ME !!\n");

  // Success (well, not so in this case...)
  return 0;
}
```

执行上述程序后会输出如下内容：

```c
step 1: unrestricted
step 2: only 'read', 'write', '_exit' and 'sigreturn' syscalls
Killed
```

## 基于BPF的seccomp

上述基于prctl系统调用的seccomp机制不够灵活，在linux 3.5之后引入了基于BPF的可定制的系统调用过滤功能。

需要先安装依赖包：`yum install libseccomp-dev`

```
#include <stdio.h>   /* printf */
#include <unistd.h>  /* dup2: just for test */
#include <seccomp.h> /* libseccomp */

int main() {
  printf("step 1: unrestricted\n");

  // Init the filter
  scmp_filter_ctx ctx;
  ctx = seccomp_init(SCMP_ACT_KILL); // default action: kill

  // setup basic whitelist
  seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(rt_sigreturn), 0);
  seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit), 0);
  seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(read), 0);
  seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 0);

  // setup our rule
  seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(dup2), 2,
                        SCMP_A0(SCMP_CMP_EQ, 1),
                        SCMP_A1(SCMP_CMP_EQ, 2));

  // build and load the filter
  seccomp_load(ctx);
  printf("step 2: only 'write' and dup2(1, 2) syscalls\n");

  // Redirect stderr to stdout
  dup2(1, 2);
  printf("step 3: stderr redirected to stdout\n");

  // Duplicate stderr to arbitrary fd
  dup2(2, 42);
  printf("step 4: !! YOU SHOULD NOT SEE ME !!\n");

  // Success (well, not so in this case...)
  return 0;
}
```

输入如下内容：

```
step 1: unrestricted
step 2: only 'write' and dup2(1, 2) syscalls
step 3: stderr redirected to stdout
Bad system call
```

## docker中的应用

通过如下方式可以查看docker是否启用seccomp：

```
# docker info --format "{{ .SecurityOptions }}"
[name=seccomp,profile=default]
```

docker每个容器默认都设置了一个seccomp profile，启用的系统调用可以从[default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)中看到。

docker会将seccomp传递给runc中的sepc.linux.seccomp。

可以通过`—security-opt seccomp=xxx`来设置docker的seccomp策略，xxx为json格式的文件，其中定义了seccomp规则。

也可以通过`--security-opt seccomp=unconfined`来关闭docker引入默认的seccomp规则的限制。

## ref

- [Introduction to seccomp: BPF linux syscall filter](https://blog.yadutaf.fr/2014/05/29/introduction-to-seccomp-bpf-linux-syscall-filter/)
- [如何在Docker内部使用gdb调试器](https://mp.weixin.qq.com/s?__biz=MzI0NjI4MDg5MQ==&mid=2715292188&idx=1&sn=2b7f26203aa594027550e324460bc901&chksm=cd6d15c8fa1a9cde757868fd34c8336433c4877d3e7689ed0a2bd90eb1ef6271bda97aa3bb03&mpshare=1&scene=1&srcid=12045vIwpmKLu97HvFOssitt%23rd)
