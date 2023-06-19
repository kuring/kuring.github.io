---
title: Golang GC
date: 2019-04-01 00:56:25
tags:
---

## 常用垃圾回收方法

### 1.引用计数

类似于C++中的智能指针，每个对象维护一个引用计数，当对象被销毁时引用计数减一，当引用计数为0时立即回收对象。

在PHP、Python中使用，适用于内存比较紧张和实时性比较高的系统。

缺点：

1. 由于频繁更新引用计数，降低了性能。
2. 循环引用问题。解决办法为避免循环引用。

### 2.标记(mark)-清除(sweep)

分为标记和清除两个阶段，算法在70年代就提出了，非常古老的算法。

该算法有一个标记初始的root区域，和一个受控堆区。root区域主要是程序当前的栈和全局数据区域。

从root区域开始遍历所有被引用的对象，所有被访问到的对象被标记为“被引用”，标记完成后对未被引用的对象进行内存回收。

缺点：

标记阶段，都会Stop The World，会大大降低性能。

### 3.分代回收

将堆划分为两个或者多个代空间，新创建的对象存放于新生代，随着垃圾回收的重复执行，生命周期较长的对象会被放到老年代中。新生代和老年代采用不同的垃圾回收策略。

## Go垃圾回收机制

Go中采用三色标记算法，本质上还是标记清除算法，但是对标记阶段有改进。

![https://upload.wikimedia.org/wikipedia/commons/1/1d/Animation_of_tri-color_garbage_collection.gif](https://kuring.oss-cn-beijing.aliyuncs.com/images/golang-gc.gif)

步骤如下：

1. 开始时，所有的对象均为白色
2. 从root开始扫描所有可达对象，标记为灰色，放入待处理队列。root包括了全局指针和goroutine栈上的指针。
3. 从队列取出所有灰色对象，将其引用对象标记为灰色放入队列，自身标记为黑色。
4. 重复3，直到灰色队列为空。
5. 将白色对象进行回收

优点：用户程序和标记操作可以并行执行。

详细过程如下图：

![](https://kuring.oss-cn-beijing.aliyuncs.com/images/golang-gc-2.png)

通过上图可以看出，STW有两个过程：

1. gc开始的时候，需要一些准备工作，如开启write barrier
2. re-scan的过程

每个对象需要一个标记位，go中并未将标记位存放于对象的内存区域中，而是采用非侵入式的标记位。go单独使用了标记位图区域来对应内存中的堆区域。

write barrier: 在运行垃圾回收算法的同时，应用程序在一直执行，上文中提到了写屏障write barrier用于将这些内存的操作记录下来。

## GC日志

GC日志对于定位问题还是比较方便的

### 1.开启GC日志

可以增加环境变量`GODEBUG=gctrace=1`来开启gc日志，gc日志会打印到标准错误中。例如有如下的程序:

```
package main

import "time"

func main() {
	for {
		s := make([]int, 10)
		for i := 0; i < 10000; i++ {
			s = append(s, i)
		}
		time.Sleep(time.Nanosecond)
	}
}
```

执行`GODEBUG=gctrace=1 go run gc.go`即可打印gc日志到标准错误中。

## 2.GC日志的含义

可以参照[runtime](https://godoc.org/runtime)

```
gc # @#s #%: #+#+# ms clock, #+#/#/#+# ms cpu, #->#-># MB, # MB goal, # P
where the fields are as follows:
	gc #        the GC number, incremented at each GC
	@#s         time in seconds since program start
	#%          percentage of time spent in GC since program start
	#+...+#     wall-clock/CPU times for the phases of the GC
	#->#-># MB  heap size at GC start, at GC end, and live heap
	# MB goal   goal heap size
	# P         number of processors used
```

例子:

gc 11557 @179.565s 2%: 0.018+1.2+0.049 ms clock, 0.14+1.1/2.3/0.90+0.39 ms cpu, 13->14->7 MB, 14 MB goal, 8 P

### 3.GC的监控

对于线上GC的监控，基本上读取runtime.MemStats结构中的内容，然后存储到时序数据库中。具体有如下两种获取方式：

```
// 方式1
memStats := &runtime.MemStats{}
runtime.ReadMemStats(memStats)

// 方式2 json格式
expvar.Get("memstats").String()
```

## references

- [Getting to Go: The Journey of Go's Garbage Collector](https://blog.golang.org/ismmkeynote)
- [Go: runtime](https://godoc.org/runtime)
