title: json patch
date: 2023-01-29 17:04:14
tags:
author:
---
# json patch
该规范定义在 [RFC 6902](https://datatracker.ietf.org/doc/html/rfc6902/)，定义了修改 json 格式的规范，同时还可以配合 http patch 请求一起使用，实例如下：
```json
   PATCH /my/data HTTP/1.1
   Host: example.org
   Content-Length: 326
   Content-Type: application/json-patch+json
   If-Match: "abc123"

   [
     { "op": "test", "path": "/a/b/c", "value": "foo" },
     { "op": "remove", "path": "/a/b/c" },
     { "op": "add", "path": "/a/b/c", "value": [ "foo", "bar" ] },
     { "op": "replace", "path": "/a/b/c", "value": 42 },
     { "op": "move", "from": "/a/b/c", "path": "/a/b/d" },
     { "op": "copy", "from": "/a/b/d", "path": "/a/b/e" }
   ]
```
支持add、remove、replace、move、copy和 test 六个patch动作。
## 协议规范
### add
格式如下：
```json
{ "op": "add", "path": "/hello", "value": [ "foo" ] }
```
规范：

1. 如果原始 json 中不存在 key "/hello"，则会全新创建 key。
2. 如果原始 json 存在 key "/hello"，则会直接覆盖；即使"/hello"为数组，也不会在原先的基础上追加，而是直接强制覆盖；

原始 json 如下：
```json
{
  "hello": ["123"]
}
```
执行后结果如下：
```json
{
    "hello": [
        "world"
    ]
}
```
### remove
用来删除某个 key，格式如下：
```json
[{ "op": "remove", "path": "/hello" }]
```
### replace
用来替换某个 key，跟 add 动作的差异是，如果 key 不存在，则不会创建 key。
```json
{ "op": "replace", "path": "/hello", "value": 42 }
```
如果原始 json 格式为: `{}`，执行完成后，输出 json 格式仍然为：`{}`。
### move
用来修改 key 的名称，格式如下：
```json
{ "op": "move", "from": "/hello", "path": "/hello2" }
```
如果 key 不存在，则不做任何修改。
### copy
用来复制某个 key，格式如下：
```json
{ "op": "copy", "from": "/hello", "path": "/hello2" }
```
如果原始 key 不存在，则不复制；如果目标 key 已经存在，则仍然会复制。

原始 json 如下：
```json
{
 "hello": "world",
 "hello2": "world2"
}
```
执行完成后的 json 如下：
```json
{
    "hello": "world",
    "hello2": "world"
}
```
### test
用来测试 key 对应的 value 是否相等，该操作并不常用
```json
{ "op": "test", "path": "/a/b/c", "value": "foo" }
```

## 工具

- [JSON Patch Builder Online](https://json-patch-builder-online.github.io/) 在线工具，可根据原始 json 和 patch 完成后的 json，产生 json patch
- [jsonpatch.me](https://jsonpatch.me/) 在线工具，可根据原始 json 和 json patch，产生 patch 完成后的 json

## 总结
通过上述协议可以发现如下缺点：

1. 对于数组的处理不是太理想，如果要删除数组中的某个元素，或者在数组中追加某个元素，则无法表达。
2. 该协议对于人类并不友好。

# json merge patch
定义在 [RFC 7386](https://www.rfc-editor.org/rfc/rfc7386)，由于patch 能力比较有限，使用场景较少。

同样可以配合 http patch 方法一起使用，http 请求如下：
```json
       PATCH /target HTTP/1.1
       Host: example.org
       Content-Type: application/merge-patch+json

       {
         "a":"z",
         "c": {
           "f": null
         }
       }
```

下面结合具体的实例来说明 json merge patch 的功能。
原始 json 格式如下：
```json
{
  "title": "Goodbye!",
  "author" : {
    "givenName" : "John",
    "familyName" : "Doe"
  },
  "tags":[ "example", "sample" ],
  "content": "This will be unchanged"
}
```
patch json 格式如下：
```json
{
  "title": "Hello!",
  "phoneNumber": "+01-123-456-7890",
  "author": {
    "familyName": null
  },
  "tags": [ "example" ]
}
```
其中 null 用来表示该 key 需要删除。对于数组类型，则直接覆盖数组中的值。
patch 完成后的 json 如下：
```json
{
  "title": "Hello!",
  "author" : {
    "givenName" : "John"
  },
  "tags": [ "example" ],
  "content": "This will be unchanged",
  "phoneNumber": "+01-123-456-7890"
}
```

通过上述实例可以发现如下的功能缺陷：

1. 如果某个 json 的 key 对应的值为 null，则无法表达，即不可以将某个 key 对应的value 设置为 null。
2. 对于数组的处理非常弱，是直接对数组中所有元素的替换。

# k8s strategic merge patch
该协议的资料较少，官方参考资料只有两篇文章，最好结合着 k8s 的代码才能完全理解：

- [Update API Objects in Place Using kubectl patch](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/update-api-object-kubectl-patch/)
- [Strategic Merge Patch](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md)
- [Pod Spec 定义](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.26/#podspec-v1-core)

## 背景
无论是 json patch，还是 json merge patch 协议，对于数组元素的支持都不够友好。
比如对于如下的 json：
```json
spec:
  containers:
    - name: nginx
      image: nginx-1.0
```
期望能够 patch 如下的内容
```json
spec:
  containers:
    - name: log-tailer
      image: log-tailer-1.0
```
从而可以实现 containers中包含两个元素的情况，无论是 json patch 还是 json merge patch，其行为是对数组元素的直接替换，不能实现追加的功能。

## 协议规范
为了解决 json merge patch 的功能缺陷，strategic merge patch 通过如下两种方式来扩展功能：

1. json merge patch 的 json 语法增强，增加一些额外的指令
2. 通过增强原始 json 的 struct 结构实现，跟 golang 语言强绑定，通过 golang 中的 struct tag 机制实现。这样的好处是不用再扩充 json merge patch 的 json 格式了。支持如下 struct tag：
   1. patchStrategy： 指定策略指令，支持：replace、merge 和 delete。默认的行为为 replace，保持跟 json merge patch 的兼容性。
   2. patchMergeKey: 数组一个子 map 元素的主键，类似于关系型数据库中一行记录的主键。

支持如下指令：

1. replace
2. merge
3. delete

### replace
支持 go struct tag 和 在 json patch 中增加指令两种方式。
replace 是默认的指令模式，对于数组而言会直接全部替换数组内容。

如下指令用来表示，
```json
$patch: replace  # recursive and applies to all fields of the map it's in
containers:
- name: nginx
  image: nginx-1.0
```
### delete
删除数组中的特定元素，下面例子可以删除数组中包含 name: log-tailer 的元素。
```json
containers:
  - name: nginx
    image: nginx-1.0
  - $patch: delete
    name: log-tailer  # merge key and value goes here
```
删除 map 的特定 key，如下实例可以删除 map 中的 key rollingUpdate。
```json
rollingUpdate:
  $patch: delete
```
### merge
该指令仅支持 go struct tag 模式，格式为：`$deleteFromPrimitiveList/<keyOfPrimitiveList>: [a primitive list]`。

#### deleteFromPrimitiveList
删除数组中的某个元素
go struct 定义如下：
```go
Finalizers []string `json:"finalizers,omitempty" patchStrategy:"merge" protobuf:"bytes,14,rep,name=finalizers"`
```
原始 yaml 如下：
```yaml
finalizers:
- a
- b
- c
- b
```
patch yaml 如下，用来表示删除finalizers中的所有元素 b 和 c
```yaml
# The directive includes the prefix $deleteFromPrimitiveList and
# followed by a '/' and the name of the list.
# The values in this list will be deleted after applying the patch.
$deleteFromPrimitiveList/finalizers:
- b
- c
```
最终得到结果：
```yaml
finalizers:
- a
```
#### setElementOrder
用于数组中的元素排序

##### 简单数组排序例子
原始内容如下：
```yaml
finalizers:
- a
- b
- c
```
设置排序顺序：
```yaml
# The directive includes the prefix $setElementOrder and
# followed by a '/' and the name of the list.
$setElementOrder/finalizers:
- b
- c
- a
```
最终得到排序顺序：
```yaml
finalizers:
- b
- c
- a
```
##### map 类型数组排序例子
其中 patchMergeKey 为 name 字段
```yaml
containers:
  - name: a
    ...
  - name: b
    ...
  - name: c
    ...
```
patch 指令的格式：
```yaml
# each map in the list should only include the mergeKey
$setElementOrder/containers:
- name: b
- name: c
- name: a
```
最终获得结果：
```yaml
containers:
  - name: b
    ...
  - name: c
    ...
  - name: a
    ...
```
#### retainKeys
用来清理 map 结构中的 key，并指定保留的 key
原始内容：
```yaml
union:
  foo: a
  other: b
```
patch 内容：
```yaml
union:
  retainKeys:
    - another
    - bar
  another: d
  bar: c
```
最终结果，可以看到 foo 和 other 因为不在保留列表中已经被清楚了。同时新增加了字段 another 和 bar，新增加的是字段是直接 patch 的结果，同时这两个字段也在保留的列表内。
```yaml
union:
  # Field foo and other have been cleared w/o explicitly set them to null.
  another: d
  bar: c
```

# strategic merge patch 在 k8s 中应用
kubectl patch 命令通过--type 参数提供了几种 patch 方法。
```json
--type='strategic': The type of patch being provided; one of [json merge strategic]
```

1. json：即支持 json patch 协议，例子：`kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path": "/spec/containers/0/image", "value":"newimage"}]`
2. merge：对应的为 json merge patch 协议。
3. stategic：k8s 特有的 patch 协议，在 json merge patch 协议基础上的扩展，可以解决 json merge patch 的缺点。

对于 k8s 的 CRD 对象，由于不存在 go struct tag，因此无法使用 stategic merge patch。

TODO：待补充具体例子

# kubectl patch、replace、apply之间的区别
## patch
kubectl patch 命令的实现比较简单，直接调用 kube-apiserver 的接口在 server 端实现 patch 操作。
## replace
如果该命令使用 --force=true 参数，则会先删除对象，然后再提交，相当于是全新创建。
## apply
apply 的实现相对比较复杂，逻辑也比较绕，可以实现 map 结构中的字段增删改操作，数组中数据的增删改操作。实现上会将多份数据进行merge 后提交，数据包含：

1. 要重新 apply 的 yaml
2. 对象的annotation kubectl.kubernetes.io/last-applied-configuration 包含的内容
3. 运行时的 k8s 对象

具体的操作步骤：

1. 要重新 apply 的 yaml 跟annotation kubectl.kubernetes.io/last-applied-configuration 包含的内容比较，获取到要删除的字段。
2. 要重新 apply 的 yaml 跟运行时的 k8s 对象进行比较，获取到要增加的字段。
3. 上述两个结果再进行一次 merge 操作，最终调用 kube-apiserver 的接口实现 patch 操作。

为什么一定需要用到kubectl.kubernetes.io/last-applied-configuration的数据呢？
在 yaml 对象提交到 k8s 后，k8s 会自动增加一些字段，也可以通过额外的方式修改对象增加一些字段。如果 patch 内容仅跟运行时结果比较，会导致一些运行时的k8s 自动增加的字段或者手工更新的字段被删除掉。

| 试验 | 上次提交 last-applied-configuration | 运行时 | patch 内容 | 结果 | 结果分析 |
| --- | --- | --- | --- | --- | --- |
| 试验一 | label1: first | label1: first | label2: second | label2: second | 1. patch 内容跟上次内容比较，发现要删除字段 label1<br>2. patch 内容跟运行时比较，发现新增加了字段 label2  <br>3. 最终 label1 被删除，仅保留 label2 |
| 试验二 | label1: first | label1: first <br>label2: second | label1: first | label1: first <br>label2: second | 1. patch 内容跟上次内容比较，发现结果无变化<br>2. patch 内容跟运行时比较，发现要新增加字段 label2<br>3. 最终新增加字段 label2 |

# 引用

- [json patch RFC 规范](https://datatracker.ietf.org/doc/html/rfc6902/)
- [https://jsonpatch.com/](https://jsonpatch.com/)
