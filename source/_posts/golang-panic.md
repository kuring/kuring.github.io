---
title: Golang中的panic和recover用法
date: 2019-03-19 23:52:04
tags: golang
---

golang中的panic用于异常处理，个人感觉没有try catch finally方式直观和易用。

`func panic(v interface{})`函数的作用为抛出一个错误信息，同时函数的执行流程会结束，但panic之前的defer语句会执行，之后该goroutine会立即停止执行，进而当前进程会退出执行。

`func recover() interface{}`定义在panic之前的defer语句中，用于将panic()进行捕获，这样触发panic时，当前gotoutine不会被退出。

recover所返回的内容为panic的函数参数，如果没有捕获到panic，则返回nil。

注意：recover仅能定义在defer中使用，在普通语句中无法捕获recover异常。recover可以不跟panic定义在同一个函数中使用。

## example 1

```
import "fmt"

func main() {
	defer fmt.Println(1)
	fmt.Println(2)
	panic(3)
	fmt.Println(4)
	defer fmt.Println(5)
}
```

panic()执行后，会先调用defer函数，然后打印`panic: 3`，当前goroutine退出，后续语句不再执行，程序输出：

```
2
1
panic: 3

goroutine 1 [running]:
main.main()
	/Users/lvkai/src_test/go/panic/panic.go:8 +0xd5
exit status 2
```

## example 2

```
package main

import "fmt"

func main() {
	func() {
		defer func() {
			if r := recover(); r != nil {
				fmt.Println("recover: ", r)
			}
		}()
		defer fmt.Println(1)
		fmt.Println(2)
		panic(3)
		fmt.Println(4)
		defer fmt.Println(5)
	}()
	fmt.Println(6)
}
```

在执行panic后，触发当前函数中的defer中的recover函数，此时panic后的当前函数中的语句同样是不再执行，但当前goroutine不会退出。也就是说panic被recover后，会影响到当前函数中的后续语句的执行，但不影响当前goroutine的继续执行，输出内容如下：

```
2
1
recover:  3
6
```

## example 3

```
package main

import "fmt"

func main() {
	func() {
		defer func() {
			if r := recover(); r != nil {
				fmt.Println("recover: ", r)
			}
		}()
		func() {
			defer fmt.Println(1)
			panic(2)
		}()
		fmt.Println(3)
		defer fmt.Println(4)
	}()
	fmt.Println(5)
}
```

recover跟panic定义在不同的函数中，仍然可以发挥作用。

```
1
recover:  2
5
```
