---
title: sync.Cond的例子
date: 2018-08-29 00:31:14
tags: golang
---

sync.Cond类似于pthread中的条件变量，但等待的为goroutine，而不是线程。比较难理解的为Wait函数，在调用该函数时必须L为Lock状态，调用Wait函数后，goroutine会自动解锁，并等待条件的到来，等条件到来后会重新加锁。

代码量并不多，下面是去掉注释后的代码。

```golang
package sync

import (
	"sync/atomic"
	"unsafe"
)

type Cond struct {
	noCopy noCopy

	// L is held while observing or changing the condition
	L Locker

	notify  notifyList
	checker copyChecker
}

func NewCond(l Locker) *Cond {
	return &Cond{L: l}
}

func (c *Cond) Wait() {
	c.checker.check()
	t := runtime_notifyListAdd(&c.notify)
	c.L.Unlock()
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock()
}

func (c *Cond) Signal() {
	c.checker.check()
	runtime_notifyListNotifyOne(&c.notify)
}

func (c *Cond) Broadcast() {
	c.checker.check()
	runtime_notifyListNotifyAll(&c.notify)
}

// copyChecker holds back pointer to itself to detect object copying.
type copyChecker uintptr

func (c *copyChecker) check() {
	if uintptr(*c) != uintptr(unsafe.Pointer(c)) &&
		!atomic.CompareAndSwapUintptr((*uintptr)(c), 0, uintptr(unsafe.Pointer(c))) &&
		uintptr(*c) != uintptr(unsafe.Pointer(c)) {
		panic("sync.Cond is copied")
	}
}

// noCopy may be embedded into structs which must not be copied
// after the first use.
//
// See https://github.com/golang/go/issues/8005#issuecomment-190753527
// for details.
type noCopy struct{}

// Lock is a no-op used by -copylocks checker from `go vet`.
func (*noCopy) Lock() {}
```

具体的使用例子如下：

```golang
package main

import (
	"fmt"
	"sync"
)

func main() {
	mutex := &sync.Mutex{}
	cond := sync.NewCond(mutex)

	wg := &sync.WaitGroup{}

	wait := func(i int, c chan int) {
		defer wg.Done()
		fmt.Println("start chan ", i)
		cond.L.Lock()
		defer cond.L.Unlock()
		fmt.Printf("chan %d wait before\n", i)
		c <- i
		// Wait是理解起来稍微麻烦的点，Cond.Wait会自动释放锁等待信号的到来，当信号到来后，第一个获取到信号的Wait将继续往下执行并从新上锁
		cond.Wait()
		fmt.Printf("chan %d wait end\n", i)
	}

	signal := func(count int, c chan int) {
		defer wg.Done()
		for i := 0; i < count; i++ {
			fmt.Printf("read chan %d ready\n", <-c)
		}
		fmt.Println("call signal")
		cond.Signal()
	}

	broadcast := func(count int, c chan int) {
		defer wg.Done()
		for i := 0; i < count; i++ {
			fmt.Printf("read chan %d ready\n", <-c)
		}
		fmt.Println("call broadcast")
		cond.Broadcast()
	}

	c := make(chan int)
	wg.Add(2)
	go wait(0, c)
	go signal(1, c)
	wg.Wait()
	fmt.Println("signal test finished\n\n")

	count := 3
	for i := 0; i < count; i++ {
		wg.Add(1)
		go wait(i, c)
	}
	wg.Add(1)
	go broadcast(count, c)
	wg.Wait()
}
```
