title: linux auditd
date: 2022-03-28 11:57:28
tags:
author:
---
## audit简介

audit为linux内核安全体系的重要组成部分，用来记录内核的系统调用，文件修改等事件，用于审计目的。

![](https://kuring.oss-cn-beijing.aliyuncs.com/common/linux_audit.png)

 - auditctl: 面向用户的工具，类似于iptables命令
 - auditd: 负责将审计信息写入到/var/
 
## 启动auditd服务

auditd作为单独的服务运行在系统上，Redhat系统使用`systemctl start auditd`启动服务，启动后通过 `ps -ef | grep auditd`查看进程是否启动成功。
 
## auditctl

查看auditd的运行状态
```
$ auditctl -s
enabled 1
failure 1
pid 638
rate_limit 0
backlog_limit 8192
lost 0
backlog 0
loginuid_immutable 0 unlocked
```

查看当前环境规则

```
$ auditctl -l
-w /tmp/hosts -p rwxa
-w /proc/sys/net/ipv4/tcp_retries1 -p rwxa
-w /proc/sys/net/ipv4/tcp_retries2 -p rwxa
-w /proc/sys/net/ipv4/tcp_retries2 -p wa
```

删除所有的audit规则
```
$ auditctl -D
No rules
```

## 实践

### 监控文件变化

1. 执行 auditctl -w $file -p wa 来监控文件，比如监控内核参数 auditctl -w /proc/sys/net/ipv4/tcp_retries2 -p wa，其中-p指定了监控文件的行为，支持rwxa。
2. 查看文件 cat /proc/sys/net/ipv4/tcp_retries2。
3. 使用vim打开文件 vim /proc/sys/net/ipv4/tcp_retries2。
4. 执行 ausearch -f /proc/sys/net/ipv4/tcp_retries2 命令查看，可以看到如下的日志

```
$ ausearch -f /proc/sys/net/ipv4/tcp_retries1
----
time->Mon Mar 28 12:44:48 2022
type=PROCTITLE msg=audit(1648442688.159:6232591): proctitle=76696D002F70726F632F7379732F6E65742F697076342F7463705F7265747269657331
type=PATH msg=audit(1648442688.159:6232591): item=1 name="/proc/sys/net/ipv4/tcp_retries1" inode=46629229 dev=00:03 mode=0100644 ouid=0 ogid=0 rdev=00:00 objtype=NORMAL cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
type=PATH msg=audit(1648442688.159:6232591): item=0 name="/proc/sys/net/ipv4/" inode=8588 dev=00:03 mode=040555 ouid=0 ogid=0 rdev=00:00 objtype=PARENT cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
type=CWD msg=audit(1648442688.159:6232591):  cwd="/root"
type=SYSCALL msg=audit(1648442688.159:6232591): arch=c000003e syscall=2 success=yes exit=3 a0=11687a0 a1=241 a2=1a4 a3=7ffe33dc14e0 items=2 ppid=8375 pid=8629 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts1 ses=250225 comm="vim" exe="/usr/bin/vim" key=(null)
```

### 监控文件夹变化

监控文件夹同样采用跟上述文件相同的方式，但有个问题是如果文件夹下内容较多，会一起监控，从而导致audit的log内容过多。

### 监控系统定期reboot

执行如下命令：

```
auditctl -w /bin/systemctl -p rwxa -k systemd_call
auditctl -a always,exit -F arch=b64 -S reboot -k reboot_call
```

待系统重启后执行如下命令：

```
ausearch -f reboot
```

## 参考文档
[RedHat auditd文档](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/chap-system_auditing)