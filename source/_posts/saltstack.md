---
title: SaltStack使用
Status: public
url: saltstack
date: 2015-08-21
toc: yes
---

本文在学习saltstack的过程中编写，内容比较基础，方便使用时查阅命令。

# 安装

为了方便起见，直接采用yum的安装方式，centos源中并没有salt，需要手工添加一下。

## CentOS 7

安装master

```
rpm -Uvh http://ftp.jaist.ac.jp/pub/Linux/Fedora/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
yum install salt-master
```

修改/etc/salt/master配置文件，在其中指定salt文件根目录位置，默认路径为/srv/salt/。

```
file_roots:
  base:
    - /svr/salt/
```

salt在安装的时候已经创建了systemctl命令启动程序需要的service文件，位于/usr/lib/systemd/system/salt-master.service，重启`systemctl restart salt-master.service`生效。


## CentOS 6.5

安装minion

```
rpm -Uvh http://ftp.linux.ncsu.edu/pub/epel/6/i386/epel-release-6-8.noarch.rpm
yum install salt-minion
```

修改/etc/salt/minion配置文件，在其中指定master主机的地址

```
master: 192.168.204.128
```

执行`service salt-minion restart`对服务进行重启。

## 连通性测试

执行`salt-key -L`命令可以看到已认证和未认证的minion，执行`salt-key -a 192.168.204.149`可接收minion。

在master主机中执行`salt '*' test.ping`可测试连接的minion主机。

```
➜  stats  salt-key -L
Accepted Keys:
Denied Keys:
Unaccepted Keys:
192.168.204.149
Rejected Keys:

➜  stats  salt-key -a 192.168.204.149
The following keys are going to be accepted:
Unaccepted Keys:
192.168.204.149
Proceed? [n/Y] y
Key for minion 192.168.204.149 accepted.

➜  stats  salt '*' test.ping
192.168.204.149:
    True
```

# state

可以通过预先定义好的sls文件对被控主机进行管理，这里演示一个简单的文件复制的例子，该例子可以将master主机上的vimrc文件复制到目标主机上。

在master主机的/svr/salt/edit目录下新建vim.sls文件，文件内容如下：

```
/etc/vimrc:
  file.managed:
    - source: salt://edit/vimrc
    - mode: 644
    - user: root
    - group: root
```

另外在edit目录下需要存在一个空的init.sls，以确保state.sls可以找到该目录下的sls文件。同时该目录下还需要存在要复制的vimrc文件。

执行`salt '*' state.sls edit.vim`即可以执行该命令。

如果将vim.sls更改为init.sls文件，执行`salt '*' state.sls edit`命令即可。


# 常用命令

salt '192.168.204.149' cmd.run 'free -m'

salt '192.168.204.149' sys.list_modules  列出minion支持哪些模块，默认已经支持很多模块

salt '192.168.204.149' cp.get_file salt://test_file /root/test_file  将master主机file_roots目录下的文件复制到minion任意目录下，该命令不可以将master主机任意目录下的文件进行复制

salt '192.168.204.149' cp.get_dir salt://test_dir/ /root/ 实验未成功

salt '*' file.mkdir dir_path=/root/test_dir user=root group=root mode=700  在minion主机上创建目录s


# 参考文章

*[saltstack官方文档](http://docs.saltstack.com/en/latest/topics/index.html)

*《python自动化运维技术与最佳实践》