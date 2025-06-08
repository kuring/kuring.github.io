# 基础概念

## WorkSpace

工作空间或者工作区，对应的是一个目录，存放项目的源文件和Bazel 的构建输出，包含如下信息：
1. WOKRSPACE 文件。位于项目的根目录下，可以为空，也可以包含对外部依赖的引用。也可以使用 WORKSPACE.bazel 文件。

WOKRSPACE 在 bazel 8 中已经默认禁用，在 bazel 9 中移除该特性，正在被 Bzlmod 取代。

工作空间、工作区、代码库通常为一个含义。
## Package

软件包包含  BUILD.bazel 或者 BUILD 文件的目录，包含了当前目录中的所有文件以及所有子目录，如果子目录中同样包含 BUILD 文件，则为一个独立的软件包。

## BUILD 文件
BUILD 文件使用Skylark语言，为该语言的一个子集。

### Target 目标

在软件包的 BUILD 文件中定义，目标主要分为文件和规则两大类。

规则用于指定输入和输出之间的关系，类型的声明如下：
```
cc_binary(
    name = "my_app", # 声明目标，每个规则必须有的属性
    srcs = ["my_app.cc"],
    deps = [
        "//absl/base",
        "//absl/strings",
    ],
)
```
### Label 标签

引用一个目标时需要使用的标签。格式如下：
```
@@myrepo//my/app/main:app_binary
```
- `@@myrepo`：代码库名称。如果引用的是当前代码库，则该部分可省略，写为`//my/app/main:app_binary`。`@@` 开头的表示规范代码库名称，`@`开头的表示明显代码库名称。
- `my/app/main`：软件包名称，相对于代码库根目录的软件包路径。如果是引用的同一个软件包，则该部分可省略，写法为 `:app_binary`或者 `app_binary`。
- `app_binary`：未限定的目标名称。如果与软件包路径的最后一个组件匹配，可以忽略，因此 `//my/app/lib`和`//my/app/lib:lib`等价。文件目标的名称为相对于软件包(包含 BUILD 文件)的文件路径。

### 声明依赖
规则里可以声明依赖，大部分规则都有如下三个属性：
1. `srcs`: 输出源文件的规则直接使用的文件。
2. `deps`: 依赖的头文件、库等。
3. `data`: 依赖的一些数据文件。

当引用目录时，使用 `glob(["testdata/**"])` 的方式。

### [目标可见性](https://bazel.build/versions/7.3.0/concepts/visibility?hl=zh-cn)


所有规则都有一个`visibility`，用来指定该目标的可见性。
- `"//visibility:public"`：授予对所有软件包的访问权限。
- `"//visibility:private"`：不会授予任何其他访问权限；只有此软件包中的目标才能使用此目标。
- `"//foo/bar:__pkg__"`：授予对 `//foo/bar`（但不包括其子软件包）的访问权限。
- `"//foo/bar:__subpackages__"`：授予对 `//foo/bar` 及其所有直接和间接子软件包的访问权限。
- `"//some_pkg:my_package_group"`：授予对给定 [`package_group`](https://bazel.build/versions/7.3.0/reference/be/functions?hl=zh-cn#package_group) 中的所有软件包的访问权限。

### 加载扩展程序
可以在 BUILD 文件中加载扩展程序：
```
load("//foo/bar:file.bzl", "some_library")
```
扩展程序的为 `*.bzl`的文件，some_library 为要导入的符号。

## bazel modules

在代码块的根目录下必须有一个 `MODULE.bazel`文件，该文件用于声明名称、版本、直接依赖项列表和其他信息。
```
module(name = "my-module", version = "1.0")

bazel_dep(name = "rules_cc", version = "0.0.1")
bazel_dep(name = "protobuf", version = "3.19.0")
```
版本采用了更为宽松的 SemVer 版本规范：
1. 可以使用任意数量的段，而不是 SemVer 中必须三段格式。
2. 允许每一段中使用字母，而不是 SemVer 中的纯数字。
3. 不存在主版本号、次版本号、补丁版本号必须递增规则。

菱形依赖的问题，采用了 Go Module 中的 MVS 算法。

bazel_dep：用来声明依赖了哪个 bazel 模块和版本，会自动下载这个模块的元数据和代码，且会自动处理依赖的依赖。
use_extension: 加载扩展模块，比如 `python = use_extension("@rules_python//python/extensions:python.bzl", "python")`，用来声明加载 python.bzl 模块中的 python，并将其赋值给变量 python。跟之前 WORKSPACE 中的 load 语法等价。通常会动态生成一些额外的仓库，比如语言的 runtime、第三方的依赖等。
use_repo: 搭配 use_extension 使用，repo 是懒加载模式，将扩展生成的仓库注册到当前的构建环境中。
# 已有 go 项目改造为 bazel 构建

```
go install github.com/bazelbuild/bazel-gazelle/cmd/gazelle@latest
```

# 使用

mac 下使用如下命令安装
```
brew install bazelisk
```
bazelisk 命令会自动解析目录下的 `.bazelversion` 文件，从而下载合适的 bazel 版本来指定。


`bazel query @com_github_google_glog//...` 查询导出了哪些 rule


# bazel central registry
## 上传代码到 bazel central registry


# 常用命令

查询一个仓库下的所有 rule
```
bazel query @foundation_tianji_package-keeper_package-keeper-api_1.0.0//...
```