---
title: Linux设备驱动程序实例之hello world
Status: public
url: linux_driver_hello_world
date: 2014-04-02
---

本文为Linux设备驱动程序的入门实践文章，编写一个hello world程序，并在Linux上执行。

# 编写驱动程序

驱动程序hello.c文件内容如下：

```
#ifndef __KERNEL__
#  define __KERNEL__
#endif
#ifndef MODULE
#  define MODULE
#endif

#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/init.h>

MODULE_LICENSE("GPL");

int hello_init()
{
    printk(KERN_WARNING "Hello kernel!\n");
    return 0;
}


void hello_exit()
{
    printk("Bye, kernel!\n");
}

module_init(hello_init);
module_exit(hello_exit)
```

# 编写Makefile

Makefile文件的写法可以采用传统的make方式，也可以采用kbuild的方式。

采用传统的make方式的写法如下：

```
ifeq ($(KERNELRELEASE),)
KERNELDIR ?= /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

modules:
        $(MAKE) -C $(KERNELDIR) M=$(PWD) modules

modules_install:
        $(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install

clean:
        rm -rf *.o *~ core .depend .*.cmd *.ko *.mod.c .tmp_versions

.PHONY: modules modules_install clean
else
        obj-m := hello.o
endif
```

采用kbuild方式的Makefile内容如下：

```
obj-m := hello.o

all :
        $(MAKE) -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
        $(MAKE) -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

# 编译

将hello.c和Makefile文件放在任意目录中，执行make命令编译。

# 安装

执行`insmod hello.ko`命令安装驱动程序，通过`lsmod`命令即可看到驱动程序已经安装。

通过查看/var/log/messages文件即可看到hello驱动程序打印的内容。

# 卸载

执行`rmmod hello.ko`命令即可卸载驱动程序模块。

# 参考文章

《深入理解Linux设备驱动程序》
《Linux那些事之我是USB》
[Ubuntu12.10 内核源码外编译 linux模块--编译驱动模块的基本方法](http://www.cnblogs.com/QuLory/archive/2012/10/23/2736339.html)