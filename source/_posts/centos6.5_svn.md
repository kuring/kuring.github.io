---
title: CentOS6.5下安装svn客户端软件
Status: public
url: centos6.5_svn
date: 2014-03-22
---

在用CentOS默认的svn客户端工具访问Windows下搭建的subversion时会提示如下错误：

```
[kuring@localhost 桌面]$ svn checkout https://192.168.100.100/svn/test
svn: 方法 OPTIONS 失败于 “https://192.168.100.100/svn/test: SSL handshake failed: SSL 错误：Key usage violation in certificate has been detected. (https://192.168.100.100)
```

通过执行如下命令可以看到svn是支持https协议的：

```
[kuring@localhost ~]$ svn --version
svn，版本 1.6.11 (r934486)
   编译于 Apr 11 2013，16:13:51

版权所有 (C) 2000-2009 CollabNet。
Subversion 是开放源代码软件，请参阅 http://subversion.tigris.org/ 站点。
此产品包含由 CollabNet(http://www.Collab.Net/) 开发的软件。

可使用以下的版本库访问模块:

* ra_neon : 通过 WebDAV 协议使用 neon 访问版本库的模块。
  - 处理“http”方案
  - 处理“https”方案
* ra_svn : 使用 svn 网络协议访问版本库的模块。  - 使用 Cyrus SASL 认证
  - 处理“svn”方案
* ra_local : 访问本地磁盘的版本库模块。
  - 处理“file”方案
```

这是由于svn客户端在https协议中使用了GnuTLS库造成的，将其更改为使用openssl库即可。通过执行如下命令可以查看svn使用的库：

```
[kuring@localhost bin]$ ldd svn | grep ssl
[kuring@localhost bin]$ ldd svn | grep tls
        libgnutls.so.26 => /usr/lib64/libgnutls.so.26 (0x00007f33004ad000)
```

-------

下面选择重新编译的方式来安装svn。

# 删除subversion

执行：`yum remove subversion`

# 检查openssl安装情况

这里已经安装：

```
[kuring@localhost tmp]$ rpm -qa | grep openssl
openssl-1.0.1e-15.el6.x86_64
openssl-devel-1.0.1e-15.el6.x86_64
```

# 安装neon

这里选择的安装版本为0.29.6，subversion对neon的版本有要求。如果不是subversion的版本，在执行subversion下的configure文件时并不会报错
```
[kuring@localhost software]$ tar zvxf neon-0.29.6.tar.gz
[kuring@localhost software]$ cd neon-0.29.6
./configure --with-ssl=openssl
make
make install
```

# 安装apr

```
tar zvxf apr-1.5.0.tar.gz
cd apr-1.5.0
./configure
make
make install
```

# 安装apr-util

```
tar zvxf apr-util-1.5.3.tar.gz 
cd apr-util-1.5.3
./configure --with-apr=/usr/local/apr
make
make install
```

# 下载sqllite

```
unzip sqlite-amalgamation-3080401.zip
mv sqlite-amalgamation-3080401 sqlite-amalgamation
mv sqlite-amalgamation subversion-1.8.8/	// 将其复制到subversion源码目录下
```

# 安装subversion

```
tar zvxf subversion-1.7.16.tar.gz
./configure --with-ssl --with-neon --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr
make
make install
```

然后再执行`svn --version`命令可以看到已经包含了https协议。

# 参考资料

[SSL handshake failed: SSL error: Key usage violation in certificate has been detected CentOS](http://itekblog.com/key-usage-violation-in-certificate-has-been-detected-centos/)

# 资料下载

[需要的安装包下载](http://pan.baidu.com/s/1qWNhAqS)