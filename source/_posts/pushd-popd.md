---
title: pushd和popd命令的用法
date: 2018-08-10 18:09:56
tags: shell
---

# pushd和popd命令的用法

在编写shell的时候，经常会在目录之间进行切换，如果使用cd命令经常会切换错误，pushd和popd使用栈的方式来管理目录。

## dirs

用于显示当前目录栈中的所有记录。

## pushd

将目录加入到栈顶部，并将当前目录切换到该目录。若不加任何参数，该命令用于将栈顶的两个目录进行对调。

## popd

删除目录栈中的目录。若不加任何参数，则会首先删除目录栈顶的目录，并将当前目录切换到栈顶下面的目录。

命令格式：`pushd  [-N | +N]   [-n]`

- `+N` 将第N个目录删除（从左边数起，数字从0开始）
- `-N` 将第N个目录删除（从右边数起，数字从0开始）
- `-n` 将目录出栈时，不切换目录


## example

```
[root@localhost tmp]# mkdir /tmp/dir{1,2,3,4}
[root@localhost tmp]# pushd /tmp/dir1
/tmp/dir1 /tmp
[root@localhost dir1]# pushd /tmp/dir2
/tmp/dir2 /tmp/dir1 /tmp
[root@localhost dir2]# pushd /tmp/dir3
/tmp/dir3 /tmp/dir2 /tmp/dir1 /tmp
[root@localhost dir3]# pushd /tmp/dir4
/tmp/dir4 /tmp/dir3 /tmp/dir2 /tmp/dir1 /tmp
# dirs的显示内容跟pushd完成后的输出一致
[root@localhost dir4]# dirs
/tmp/dir4 /tmp/dir3 /tmp/dir2 /tmp/dir1 /tmp

[root@localhost dir4]# popd
/tmp/dir3 /tmp/dir2 /tmp/dir1 /tmp

# 带有数字比较容易出错
[root@localhost dir3]# popd +1
/tmp/dir3 /tmp/dir1 /tmp

# 清除目录栈
[root@localhost dir3]# dirs -c
[root@localhost dir3]# dirs
/tmp/dir3
```
