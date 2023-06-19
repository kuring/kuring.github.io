title: cue语言（个人笔记）
date: 2022-05-04 23:24:34
tags:
author:
---
cue为使用golang编写的一款配置语言。

# 安装
mac用户执行：`brew install cue-lang/tap/cue`，其他操作系统用户可以直接使用源码安装：`go install cuelang.org/go/cmd/cue@latest`

# 命令行使用
创建如下文件first.cue

```cue
a: 1.5
a: float
b: 1
b: int
d: [1, 2, 3]
g: {
    h: "abc"
}
e: string
```

- cue fmt first.cue：对代码进行格式化
- cue vet first.cue：校验语法的正确性
- cue eval first.cue：获得渲染结果
- cue export first.cue：将渲染结果以json格式的形式导出，如果指定参数--out yaml，则可以以yaml方式导出。如果要导出某个变量，可以使用-e参数来指定变量。

# 语法

## 基础数据类型
支持的数据类型包括：float、int、string、array、bool、struct、null、自定义数据类型

```yaml
// float
a: 1.5

// int
b: 1

// string
c: "blahblahblah"

// array
d: [1, 2, 3, 1, 2, 3, 1, 2, 3]

// bool
e: true

// struct
f: {
    a: 1.5
    b: 1
    d: [1, 2, 3, 1, 2, 3, 1, 2, 3]
    g: {
        h: "abc"
    }
}

// null
j: null

// 表示两种类型的值，即可以是string，也可以是int
h: string | int

// 表示k的默认值为1，且类型为int
k: *1 | int

// 自定义数据类型
#abc: string

```
更复杂的自定义数据类型如下：
```yaml
#abc: {
  x: int
  y: string
  z: {
    a: float
    b: bool
  }
}
```

## 定义cue模板
文件deployment.cue定义如下内容
```yaml
// 自定义结构体类型
#parameter: {
    name: string
    image: string
}

template: {
    apiVersion: "apps/v1"
    kind:       "Deployment"
    spec: {
        selector: matchLabels: {
            "app.oam.dev/component": parameter.name
        }
        template: {
            metadata: labels: {
                "app.oam.dev/component": parameter.name
            }
            spec: {
                containers: [{
                    name:  parameter.name  // 引用自定义结构体变量parameter
                    image: parameter.image
                }]
            }}}
}

parameter:{
   name: "mytest"
   image: "nginx:v1"
}
```
执行 cue export deployment.cue -e template --out yaml 可获取到渲染结果。

# 引用
- [Getting Started](https://cuelang.org/docs/install/)
- [CUE语言基础入门](https://kubevela.io/zh/docs/platform-engineers/cue/basic)
- [Cuetorials](https://cuetorials.com/zh/introduction/)