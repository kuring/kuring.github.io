---
title: LNMP开发环境搭建
Status: public
url: build_lnmp
tags: LNMP
date: 2015-07-16
toc: yes
---

这是一篇拿来主义的文章，所有的安装步骤仅为互联网上查找，网络上的教程各种凌乱，这里根据我的实践情况进行了更改，本文仅记录了我的安装过程，由于不同环境可能导致安装步骤不甚相同。

# MAC OS X 10.10

## php

Mac OSX 10.10的系统自带了php、php-fpm，省去了安装的麻烦，可以执行`php -v`查看php的版本。这里需要简单地修改下php-fpm的配置，否则运行php-fpm会报错。

```
sudo cp /private/etc/php-fpm.conf.default /private/etc/php-fpm.conf
vim /private/etc/php-fpm.conf
```

修改php-fpm.conf文件中的error_log项，默认该项被注释掉，这里需要去注释并且修改为error_log = /usr/local/var/log/php-fpm.log。如果不修改该值，运行php-fpm的时候会提示log文件输出路径不存在的错误。

如果系统中存在多个php-fpm.conf，不知道需要编辑哪一个，可以执行`php-fpm -t`命令查看php-fpm要读取的配置文件。

通过`php-fpm -D`来启动php-fpm，可以通过`lsof -Pni4 | grep LISTEN | grep php`命令来查看php-fpm是否监听在9000端口。

## nginx

这里为了简单，直接采用了brew的方式安装。执行

```
brew install nginx
```

nginx的配置文件位于/usr/local/etc/nginx/nginx.conf，默认只能解析html文件，需要配置后才能调用php-fpm解析php文件。下面内容为该修改后的文件全部有效内容：

```
worker_processes  1;
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       8080;
        server_name  localhost;
        root           /Users/kuring/www;	// 页面存放路径
        location / {
            index  index.html index.htm index.php;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        location ~ \.php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    }
}
include servers/*;
```

然后执行`nginx`命令即可启动，默认监听的端口为8080，在浏览器中输入`http://127.0.0.1:8080`即可看到nginx的初始界面。nginx要想监听1024以下端口还需要进一步的配置，8080端口既能满足我需求，不再更改。


## mysql

```
brew install mysql
```

在启动mysql之前可对mysql的配置文件进行更改，我这里需要更改mysql的编码方式，将所有的编码方式都更改为utf8，防止乱码问题的发生。mysql的配置文件为my.cnf，我的位于/usr/local/Cellar/mysql/5.6.25/my.cnf，对文件添加如下内容，有些选项不存在，可手动添加。

```
[mysqld]
character-set-server=utf8

[client]
default-character-set=utf8

[mysqld_safe]
default-character-set=utf8
```

输入`mysqld`命令即可启动mysql，启动mysql后输入`mysql_secure_installation`命令对mysql进行配置，可以设置root用户的密码。

通过`mysql -uroot -p`命令连接到mysql后，输入`status`命令可查看刚才更改的编码是否生效。

由于不需要长期使用mysql，这里不设置mysql自启动命令。

# CentOS6

我首先采用的方案为完全用普通用户安装，尝试失败后采用root安装依赖库普通用户编译程序的方案。

## 普通用户安装依赖库

我首选选择在普通用户kuring下进行安装和运行整个web环境。因此不能使用yum安装方式，必须采用源码安装的方式。通过普通用户安装，最麻烦的地方就在于需要安装很多的依赖库，而依赖的库的安装可能又有需要的库，且库之间存在版本问题。

首先普及几个小知识：

bash查找命令的先后顺序为：

1. alias别名
2. shell中的关键字，如if等
3. shell中的函数
4. shell内置命令，如echo等
5. $PATH环境变量，PATH中的匹配顺序为从前向后的。

程序查找lib库的先后顺序为：

1. 编译程序时指定的链接库路径，g++编译器可以通过`-Wl,-rpath,路径`来指定链接库的路径。
2. 环境变量LD_LIBRARY_PATH指定的搜索路径。
3. /etc/ld.so.conf指定的路径。
4. 默认的系统动态库搜索路径，如/usr/lib64、/usr/local/lib64等。

很多程序采用pkg-config程序来检查库的版本号，pkg-config命令依赖于动态链接库对应的.pc文件，这些.pc文件一般位于系统的/usr/local/lib/pkgconfig目录下。为了能够将安装完成的库通过pkg-config找到对应的.pc文件，需要将.pc文件所在的路径/home/kuring/local/lib/pkg-config设置到环境变量PKG_CONFIG_PATH中。

安装php依赖的libxml2库时提示找不到libtool、autoconf和automake，首先安装libtool。执行`./configure --prefix=/home/kuring/local;make; make install`将其安装到当前用户的local目录下。

用同样的步骤安装autoconf，执行`./configure --prefix=/home/kuring/local;make; make install`。

为了能够将安装的程序起作用，需要将/home/kuring/local目录添加到PATH环境变量中，在.bash_profile文件中添加PATH=$PATH:$HOME/local/bin语句，并执行`source ~/.bash_profile`。

安装之前需要先安装libxml2库，下载地址采用`git clone git://git.gnome.org/libxml2`的方式下载。在执行`sh autogen`产生configure配置文件的过程中，发现提示

```
./configure: line 13094: syntax error near unexpected token `LZMA,liblzma,'
./configure: line 13094: `    PKG_CHECK_MODULES(LZMA,liblzma,'
```

经过发现是由于找不到PKG_CHECK_MODULES造成的，正常情况下该函数定义在aclocal.m4文件，而该情况下aclocal.m4文件中并不存在该函数。之所以不存在是由于aclocal命令找不到pkg.m4文件造成的，可以通过`aclocal --print`命令查看查找的pkg.m4文件的路径。我这里的解决思路为直接从其他机器上复制一个pkg.m4文件过来。

在产生了configure命令后，执行`./configure --prefix=/home/kuring/local`命令后发现提示找不到Python.h命令的错误。

鉴于遇到了如此之多的错误，本着不浪费生命的原则还是采用yum来安装依赖库吧。

## php

这里的mysql直接采用了yum命令安装的。

在执行php的`./configure`命令后提示libxml2找不到错误，直接`yum install libxml-devel`命令安装libxml-devel即可。然后执行`./configure --enable-fpm --prefix=/home/kuring/php5.5 --with-mysqli=/usr/bin/mysql_config;make; make install;`。在编译php的时候要加上php-fpm选项来安装php-fpm命令。

安装后配置~/.bash_profile文件的$PATH环境变量的值为：PATH=$HOME/bin:$HOME/php5.5/bin:$HOME/php5.5/sbin:$PATH。

此时即可通过`php-fpm -D`命令来启动php-fpm命令了。

安装完成后通过phpinfo()函数查看里面有MySQLi的选项，但是实际程序运行的时候居然不支持mysqli的一些力函数，说明mysqli的扩展安装不成功。在/home/kuring/php5.5/include/php/ext/mysqli目录中找到了对应的.h文件，却没有找到mysqli.so的动态链接库文件。大概是由于在编译php时mysql的路径配置有些问题造成的，因为mysql是通过yum安装的，路径比较乱一些。

为了能够产生mysqli.so文件，采用单独编译的方式，在php的源码目录中已经包含了mysqli的源码，进入mysqli源码目录下执行`phpize;./configure --prefix=/home/kuring/php5.5/mysqli --with-php-config=/home/kuring/php5.5/bin/php-config --with-mysqli=/usr/bin/mysql_config;make;make install；`。将mysqli.so文件安装到了/home/kuring/php5.5/lib/php/extensions/no-debug-non-zts-20121212目录下，不知道为什么目录末尾还要加个这么长的文件夹名，直接将文件复制到上一级目录下。

在/home/kuring/php5.5目录下没有找到php.ini文件，通过`php --ini`命令查看php的配置文件路径为/home/kuring/php5.5/lib，直接从php的源码文件中复制一个php.ini文件到该目录下。并将php.ini中的增加如下内容：

```
extension_dir = "/home/kuring/php5.5/lib/php/extensions"
extension=mysqli.so
```

再运行程序，发现mysqli的系列函数已经支持了，好一段折腾。

## php-fpm

执行`cp $HOME/php5.5/etc/php-fpm.conf.default $HOME/php5.5/etc/php-fpm.conf`来增加配置文件。


## nginx

首先安装pcre库，该库为正则表达式库。下载后通过

下载源码后执行`./configure --prefix /home/kuring/nginx;make;make install;`即可安装完成。

修改nginx的配置文件为如下内容：

```
worker_processes  1;
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       8080;
        server_name  localhost;
        root           /home/kuring/www;   
        location / {
            index  index.html index.htm index.php;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        location ~ \.php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    }
}
include servers/*;
```


# 常见操作

## nginx

* nginx -s stop：关闭
* nginx -t：检测nginx的配置是否正确

## mysql

* mysqld_safe：启动mysql
* mysqladmin shutdown -u root -p：关闭mysql
* create user kuring identified by 'kuring_pass'：mysql创建用户（我尝试过几次，每次创建的用户密码都为空）
* drop user kuring：删除一个用户
* grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option ：允许mysql的root用户通过远程登录

### 创建用户的操作

使用`create user kuring identified by 'kuring_pass'`命令创建用户kuring。默认创建完成的用户在本机无法登陆，但是远程却可以登陆。 这是因为mysql数据库中的user表中存在一条记录造成的。

```
use mysql;
select user,host,password from user;

// 在表中存在一条用户名为空的记录
+---------+-----------+-------------------------------------------+
| User    | host      | password                                  |
+---------+-----------+-------------------------------------------+
| root    | localhost | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| root    | 127.0.0.1 | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| root    | ::1       | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
|         | localhost |                                           |
| root    | %         | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| report1 | %         | *884CAA4D6FA1C3F7E4849C8DAF1B5B37FCB3EC0B |
+---------+-----------+-------------------------------------------+

// 将mysql中的为空的记录删除掉，这样就可以通过创建的用户连接了
```

在mysql命令行中执行`grant all privileges on kuring_db.* to kuring identified by 'kuring_pass'`命令可以给刚创建的用户对数据库的权限。

### 修改mysql用户密码

mysql将用户名和密码存放到了mysql数据库的user表中，在mysql命令行中执行`use mysql;update user set password=password("new password") where user="username";flush privileges;`即可更新相应用户的密码。

## php-fpm

* php-fpm -D：启动php-fpm，如果需要指定php.ini文件，可以使用-c参数
* php-fpm -t：检查php-fpm的配置文件
* kill -USR2: 重启php-fpm
* kill -INT: 停止php-fpm 

## php

* php --ini：显示php.ini文件路径



# 参考文章

1. [运用Autoconf和Automake生成Makefile的学习之路](http://www.cnblogs.com/hnrainll/archive/2013/01/06/2847069.html)