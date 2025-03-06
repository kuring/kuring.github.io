---
title: pyenv 的使用
permalink: /pyenv
date: 2025-03-05
---

通过 pyenv 命令可以管理本地的 python 版本和 python 环境（各类安装包）。

# 安装 python 版本

mac 下目前已经无法通过 brew 命令安装 python2，可以使用 pyenv 工具来安装 python 2。

```
brew install pyenv
pyenv install 2.7.18
```

下载下来的 python 版本位于 ~/.pyenv/versions 目录下。

# 创建 virtualenv 环境

需要先安装插件 pyenv-virtualenv：

```shell
brew install pyenv-virtualenv
```
并在 ~/.zshrc 文件中新增加如下内容，并新打开终端窗口，确保插件被加载：
```
# 添加 pyenv 和 pyenv-virtualenv 初始化
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

创建新的 python 环境 python2，并激活该环境：
```
pyenv virtualenv 2.7.18 python2
pyenv activate python2
```

创建的环境会存在于 `~/.pyenv/versions` 目录下。