---
title: ebpf
tags:
author:
---

## 相关术语

1. BCC：BPF 编译器集合，包含了构建 BPF 字节码的编译框架和库，提供了大量的工具。提供了 python、C 语言等接口，

## 什么是 eBPF

## 环境搭建



安装必须的开发组件



在 /etc/apt/sources.list 中增加如下内容：

```
deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy main
deb-src http://apt.llvm.org/jammy/ llvm-toolchain-jammy main
```



```
apt-get install -y make clang llvm libelf-dev libbpf-dev bpfcc-tools libbpfcc-dev linux-tools-$(uname -r) linux-headers-$(uname -r)
```



```
yum install libbpf-devel make clang llvm elfutils-libelf-devel bpftool bcc-tools bcc-devel -y
yum install python-pip3
pip3 install bcc
```

1. 

2. 内核验证并运行BPF 字节码，并将状态保存到 BPF 映射中

3. 用户程序通过 BPF 映射查询BPF 字节码的运行状态

   

### 1. 使用 C 语言开发一个eBPF 程序

使用 C 语言编写 hello.c 的eBPF 程序，内容如下：

```

int hello_world(void *ctx)
{
    bpf_trace_printk("Hello, World!");
    return 0;
}
```

bpf_trace_printk为最常用的BPF辅助函数，可以输出一段字符串，类似于python 的 print 函数。其输出内容会打印到文件 /sys/kernel/debug/tracing/trace_pipe 中。

### 2. 使用 LLVM 将eBPF 程序编译为BPF 字节码

使用 python 语言编写 hello.py 文件，内容如下：

```

#!/usr/bin/env python3
# 1) import bcc library
from bcc import BPF

# 2) load BPF program
b = BPF(src_file="hello.c")
# 3) attach kprobe
b.attach_kprobe(event="do_sys_openat2", fn_name="hello_world")
# 4) read and print /sys/kernel/debug/tracing/trace_pipe
b.trace_print()
```

### 3. 通过bpf 系统调用，将 BPF 字节码提交给内核

在运行时，BCC 会调用LLVM将hello.c 编译为字节码，并加载到内核中执行。执行 `python3 hello.py`，可以看到如下的输出结果：

```

b'           <...>-4792    [001] d...1  3129.123047: bpf_trace_printk: Hello, World!'
b' /usr/local/clou-1583    [000] d...1  3129.167824: bpf_trace_printk: Hello, World!'
```

各个字段含义如下：

- /usr/local/clou-1583: 进程名称和 pid

- `[001]`: 表示 cpu 编号
- `d...`：表示选项
- `3129.123047`：表示时间戳
- `bpf_trace_printk`：表示函数名
- `"Hello, World!"`：为打印结果

上述格式可以通过文件`/sys/kernel/debug/tracing/trace_options`来进行调整。但通过bpf_trace_printk的方式会带来比较大的性能开销，同时所有的 ebpf 均将内容输出到了同一个文件中。因此，该方式很少使用。

### 4. 改进

hello2.c 代码如下：

```c

// 包含头文件
#include <uapi/linux/openat2.h>
#include <linux/sched.h>

// 定义数据结构
struct data_t {
  u32 pid;
  u64 ts;
  char comm[TASK_COMM_LEN];
  char fname[NAME_MAX];
};

// 定义性能事件映射
BPF_PERF_OUTPUT(events);


// 定义kprobe处理函数
int hello_world(struct pt_regs *ctx, int dfd, const char __user * filename, struct open_how *how)
{
  struct data_t data = { };

  // 获取PID和时间
  data.pid = bpf_get_current_pid_tgid();
  data.ts = bpf_ktime_get_ns();

  // 获取进程名
  if (bpf_get_current_comm(&data.comm, sizeof(data.comm)) == 0)
  {
    bpf_probe_read(&data.fname, sizeof(data.fname), (void *)filename);
  }

  // 提交性能事件
  events.perf_submit(ctx, &data, sizeof(data));
  return 0;
}
```

以 `bpf_`开头的为bpf 的内置函数，上述例子中用到了如下函数：

- bpf_get_current_pid_tgid: 获取进程的TGID和 PID
- bpf_ktime_get_ns: 获取系统自启动以来的时间，单位为纳秒
- bpf_get_current_comm: 获取到当前的进程名称
- bpf_probe_read: 类似于 memcpy 函数

hello2.py

```python

from bcc import BPF

# 1) load BPF program
b = BPF(src_file="trace-open.c")
b.attach_kprobe(event="do_sys_openat2", fn_name="hello_world")

# 2) print header
print("%-18s %-16s %-6s %-16s" % ("TIME(s)", "COMM", "PID", "FILE"))

# 3) define the callback for perf event
start = 0
def print_event(cpu, data, size):
    global start
    event = b["events"].event(data)
    if start == 0:
            start = event.ts
    time_s = (float(event.ts - start)) / 1000000000
    print("%-18.9f %-16s %-6d %-16s" % (time_s, event.comm, event.pid, event.fname))

# 4) loop with callback to print_event
b["events"].open_perf_buffer(print_event)
while 1:
    try:
        b.perf_buffer_poll()
    except KeyboardInterrupt:
        exit()
```

