---
title: Linux难点之权限类型
Status: public
url: linux_permission
date: 2014-03-08
---

[TOC]

在Linux操作系统中，每个文件都有一组9个权限位来控制谁能够读写和执行该文件的内容。这组权限对于文件的意义非常容易理解。但是对于目录而言就不是那么容易理解了。要想搞明白权限机制需要了解文件系统中的inode节点和block等概念，并大致了解文件系统的内部实现原理。

# 文件的权限

这部分比较容易理解。比较容易搞错的为：

w位：包含对文件的添加、修改该文件内容的权限，但不包含删除文件或移动该文件的权限，因为文件的文件名信息存储在父目录的block中，而不是在该文件的inode节点中。父目录的block中包含该文件的inode节点和文件名信息，通过父目录中的inode节点找到该文件。

# 目录的权限

r位：表示具有读取该目录列表的权限。

w位：对该文件夹下创建新的文件或目录进行增加、删除、修改操作。

x位：这个稍微难以理解。表示用户能否进入该目录成为工作目录，即是否可以cd到该目录。通常和r位组合一块使用。

## 例子一

假设root用户对testing目录和testing目录下的testing文件拥有的如下权限：

```
[root@localhost tmp]# ls -ald testing/ testing/testing
drwxr--r--. 2 root root 4096 3月   1 11:44 testing/
-rw-------. 1 root root    0 3月   1 11:44 testing/testing
```

kuring用户拥有对该文件夹的读权限，但是没有x权限。当kuring用户访问时会提示如下内容：

```
// 拥有r的权限可以查询文件名，但是没有x权限，不能读取除文件名外的其他信息，产生了问号。
[kuring@localhost tmp]$ ll testing/
ls: 无法访问testing/testing: 权限不够
总用量 0
-????????? ? ? ? ?            ? testing    

// 没有x权限，不能进入该目录
[kuring@localhost tmp]$ cd testing/
bash: cd: testing/: 权限不够
```

## 例子二

在上述例子中，给kuring用户增加对目录testing的rwx权限，却只拥有testing目录下的testing文件的r权限。权限情况如下所示：

```
[kuring@localhost tmp]$ ls -ald testing testing/testing
drwxr--rwx. 2 root root 4096 3月   1 13:29 testing
-rw-r--r--. 1 root root    0 3月   1 13:29 testing/testing
```

kuring用户执行如下操作：

```
// 当对文件进行更改时由于没有对文件的w权限，操作失败
[kuring@localhost tmp]$ echo "world" >> testing/testing
bash: testing/testing: 权限不够

// 当对文件进行删除操作时却可以删除该文件，这是因为文件删除操作的权限是由该文件所在目录的w位决定的
// 文件删除操作会修改父目录中block节点中的文件名内容，而父目录的权限为rx，不可写。
[kuring@localhost tmp]$ rm testing/testing
rm：是否删除有写保护的普通文件 "testing/testing"？y
[kuring@localhost tmp]$ ls testing/
[kuring@localhost tmp]$
```

只有了解了原理，就可以理解在多级目录并且目录的权限不一致的情况下相应的权限问题了。

------------------------

# 进程用户ID

要讲解set uid和set gid，就涉及到进程的用户ID概念。用户ID又可以分为两部分：

* 实际用户ID和实际组ID：标识了究竟是哪个用户执行了该程序，跟命令行中的登录用户一致，可以通过id命令查看。

* 有效用户ID和有效组ID：系统通过有效用户ID、有效组ID和附加组ID来决定进行对系统资源的访问权限。在Linux系统中一个用户可以属于多个组，在/etc/passwd文件中一个用户仅能标识出隶属于一个组ID，该组ID叫做默认的组ID。在/etc/group为文件中可以标识出一个用户隶属于多个组，这多个组中除去默认的组ID叫做附加组ID。

# set uid

* 该权限位仅对二进制文件有效
* 执行者对该程序需要具有x的权限
* 该权限仅在执行该程序过程中有效
* 执行者将具有该程序拥有者的权限

以/usr/bin/passwd命令为例，该程序的权限为：rwsr-xr-x。普通用户可以调用该程序修改自己的密码，而密码文件/etc/shadow未设置任何权限，即只有root用户可以操作该文件。

```
[root@localhost tmp]# ll /usr/bin/passwd
-rwsr-xr-x. 1 root root 30768 2月  22 2012 /usr/bin/passwd
[root@localhost tmp]# ll /etc/shadow
----------. 1 root root 1248 11月 30 10:50 /etc/shadow
```

passwd文件的s标志表明setuid位被设置。

passwd的拥有者为root；当普通用户执行passwd命令时，普通用户具有该命令的执行权限；passwd执行该命令时会暂时获得root用户权限；/etc/shadow就可以被修改。

也许看到这里就会有疑问：那岂不是普通用户可以通过/etc/passwd命令修改其他用户的密码了。之所以不可以这么操作是因为passwd命令内部通过逻辑实现。

# set gid

设置在文件上的情况：

* 该权限位仅对二进制文件有效
* 执行者对该程序需要具有x的权限
* 执行者在运行程序的过程中将会获得该执行群组的权限

设置在目录上的情况：

* 用户对目录具有r和x的权限
* 用户在目录下的有效群组将会变成该目录的群组
* 若用户在此目录下具有w的权限，使用者建立的群组与该目录的群组相同


# 粘附位（sticky bit）

该权限位仅对目录有效，如果在目录上设置了粘附位，只有该目录的属主、该文件的属主或root用户可以删除或重命名该目录文件。

/tmp文件夹的权限如下，其中的t位表示粘附位：

```
[kuring@localhost /]$ ll -ad tmp/
drwxrwxrwt. 25 root root 4096 3月   1 14:51 tmp/
```

如果在/tmp下kuring用户创建了自己的文件kuring_file，并设置权限为777，test用户并不能删除该文件

```
[kuring@localhost tmp]$ touch kuring_file
[kuring@localhost tmp]$ ll kuring_file
-rw-rw-r--. 1 kuring kuring 0 3月   1 15:05 kuring_file
[kuring@localhost tmp]$ chmod 777 kuring_file
[kuring@localhost tmp]$ ll kuring_file
-rwxrwxrwx. 1 kuring kuring 0 3月   1 15:05 kuring_file

[kuring@localhost tmp]$ su - test
密码：
[test@localhost ~]$ cd /tmp/
[test@localhost tmp]$ rm kuring_file
rm: 无法删除"kuring_file": 不允许的操作
```

# 表示方法

如果设置了setuid位，属主的执行权限中的x用s来代替；
如果设置了setgid位，组执行权限中的x用s来代替；
如果设置了sticky位，权限最后的那个字符被设置为t；
如果设置了setuid、setgid或sticky位中一个，又没有设置相应的执行位，这些位显示为S或T。

# 权限设定方式

4为setuid位，2为setgid位，1为sticky位。具体参考《鸟哥的Linux私房菜》。

# 参考文档

《鸟哥的Linux私房菜》
《UNIX/Linux系统管理技术手册》