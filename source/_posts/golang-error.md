---
title: Golang中的异常处理机制
date: 2021-03-17 00:24:44
tags:
---

异常处理分为error和defer和recover两类，其中error用来处理可预期的异常，recover用来处理意外的异常。

## error

支持多个返回值，可以将业务的返回值和错误的返回值分开，很多都会返回两个值。如果不使用error返回值，可以用_变量来忽略。
```go
// parseConfig returns a parsed configuration for an Azure cloudprovider config file
func parseConfig(configReader io.Reader) (*Config, error) {
	var config Config

	if configReader == nil {
		return &config, nil
	}

	configContents, err := ioutil.ReadAll(configReader)
	if err != nil {
		return nil, err
	}
	err = yaml.Unmarshal(configContents, &config)
	if err != nil {
		return nil, err
	}

	// The resource group name may be in different cases from different Azure APIs, hence it is converted to lower here.
	// See more context at https://github.com/kubernetes/kubernetes/issues/71994.
	config.ResourceGroup = strings.ToLower(config.ResourceGroup)
	return &config, nil
}
```

<br />error的几种使用方式：

| 使用error的方式 | 说明 | 举例 |
| --- | --- | --- |
| errors.New  | 简单静态字符串的错误，没有额外的信息 | errors.New("shell not specified") |
| [fmt.Errorf](https://golang.org/pkg/fmt/#Errorf) | 用于格式化的错误字符串 | fmt.Errorf("failed to start kubernetes.io/kube-apiserver-client-kubelet certificate controller: %v", err) |
| 实现Error()方法的自定义类型 | 客户段需要检测并处理该错误时使用该方式 | 见下文自定义error |
| Error wrapping | Go 1.13支持的特性 |  |

### errors.New
原则：

- 不要在客户端判断error中的包含字符串信息。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>
```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

func use() {
  if err := foo.Open(); err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```
</td><td>
```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if errors.Is(err, foo.ErrCouldNotOpen) {
    // handle
  } else {
    panic("unknown error")
  }
}
```
</td></tr>
</tbody></table>


当然也可以使用自定义error类型，但此时由于要实现自定义error类型，代码量会增加。

### 自定义error
error是个接口，可以用来扩展自定义的错误处理。
```go
// file: k8s.io/kubernetes/pkg/volume/util/nestedpendingoperations/nestedpendingoperations.go

// NewAlreadyExistsError returns a new instance of AlreadyExists error.
func NewAlreadyExistsError(operationName string) error {
	return alreadyExistsError{operationName}
}

// IsAlreadyExists returns true if an error returned from
// NestedPendingOperations indicates a new operation can not be started because
// an operation with the same operation name is already executing.
func IsAlreadyExists(err error) bool {
	switch err.(type) {
	case alreadyExistsError:
		return true
	default:
		return false
	}
}

type alreadyExistsError struct {
	operationName string
}

var _ error = alreadyExistsError{}

func (err alreadyExistsError) Error() string {
	return fmt.Sprintf(
		"Failed to create operation with name %q. An operation with that name is already executing.",
		err.operationName)
}
```
还可以延伸出更复杂一些的树形error体系：<br />

```go
// package net

type Error interface {
    error
    Timeout() bool   // Is the error a timeout?
    Temporary() bool // Is the error temporary?
}



type UnknownNetworkError string

func (e UnknownNetworkError) Error() string

func (e UnknownNetworkError) Temporary() bool

func (e UnknownNetworkError) Timeout() bool
```

### Error Wrapping
error类型仅包含一个字符串类型的信息，如果函数的调用栈信息为A -> B -> C，如果函数C返回err，在函数A处打印err信息，那么很难判断出err的真正出错位置，不利于快速定位问题。我们期望的效果是在函数A出打印err，能够精确的找到err的源头。<br />
<br />为了解决上述问题，需要error类型在函数调用栈之间传递，有如下解决方法：

- 使用fmt.Errorf()函数来增加额外的信息
- 使用Error Wrapping  
   - golang 1.13支持%w
   - [https://github.com/pkg/errors](https://github.com/pkg/errors)


<br />使用fmt.Errorf()来封装error信息，基于已经存在的error再产生一个新的error类型，需要避免error中包含冗余信息。<br />

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>
```go
// err: failed to call api: connection refused
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "failed to create new store: %s", err)
}
```
</td><td>
```go
// err: call api: connection refused
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "new store: %s", err)
}
```
<tr><td>
```
failed to create new store: failed to call api: connection refused
error中会有很多的冗余信息
```
</td><td>
```
new store: call api: connection refused
error中没有冗余信息，同时包含了调用栈信息
```
</td></tr>
</tbody></table>

但使用fmt.Errorf()来全新封装的error信息的缺点也非常明显，丢失了最初的err信息，已经在中间转换为了全新的err。

## 类型断言
类型转换如果类型不正确，会导致程序crash，必须使用类型判断来判断类型的正确性。<br />

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>
```go
t := i.(string)
```
</td><td>
```go
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```
</td></tr>
</tbody></table>

## panic
用于处理运行时的异常情况。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2021/png/220839/1615386682235-b67733ca-036d-40d3-a895-c6b65a58f9d3.png#align=left&display=inline&height=563&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1060&originWidth=887&size=418096&status=done&style=none&width=471)<br />使用原则

- 不要使用panic，在kubernetes项目中几乎没有使用panic的场景
- 即使使用panic后，一定要使用recover会捕获异常
- 在测试用例中可以使用panic

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func run(args []string) {
  if len(args) == 0 {
    panic("an argument is required")
  }
  // ...
}

func main() {
  run(os.Args[1:])
}
```
</td><td>
```go
func run(args []string) error {
  if len(args) == 0 {
    return errors.New("an argument is required")
  }
  // ...
  return nil
}

func main() {
  if err := run(os.Args[1:]); err != nil {
    fmt.Fprintln(os.Stderr, err)
    os.Exit(1)
  }
}
```
</td></tr>
</tbody></table>

## client-go
client-go利用队列来进行重试<br />
<br />[https://github.com/kubernetes/client-go/blob/master/examples/workqueue/main.go#L93](https://github.com/kubernetes/client-go/blob/master/examples/workqueue/main.go#L93)
## kube-builder
kube-builder为client-go的更上次封装，本质上跟client-go利用队列来进行重试的机制完全一致。

# 发生了错误后该如何处理

- 打印错误日志
- 根据业务场景选择忽略或者自动重试
- 程序自己crash

# 如何避免

- 在编写代码时增加防御式编程意识，不能靠契约式编程。一个比较简单的判断错误处理情况的方法，看下代码中if语句占用的比例。[https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/kubelet_volumes.go](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/kubelet_volumes.go)
- 需求的评估周期中，不仅要考虑到软件开发完成的时间，同时要考虑到单元测试（单元测试用例的编写需要较长的时间）和集成测试的时间
- 单元测试覆盖率提升，测试场景要考虑到各种异常场景


