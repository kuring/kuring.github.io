---
title: 自动通过跳板机登录到其他服务器
date: 2017-05-17 16:04:01
tags:
---

最近公司需要首先登录跳板机relay，然后通过跳板机才能登录服务器，操作上略显麻烦。为了节省登录服务器的时间，我编写了一个简单的脚本来简化登录操作。

实现效果为在本地terminal下，执行`wrelay $host`，即可自动登录到相应的主机。

# 在relay服务器上增加对其他服务器的免登录命令

在relay服务器上ssh到其他主机时需要输入密码，使用expect命令来登录到其他主机时通过expect脚本来实现自动输入密码并登录的功能。

在/home/$user目录下新建mybin文件夹，并将mybin文件夹添加到$PATH环境变量中，具体修改方法不展开。

在mybin目录下增加gw脚本，内容如下：

```
#!/usr/bin/expect

if {$argc < 1} {
    puts "Usage:cmd <host>"
    exit 1
}

set host [lindex $argv 0]
# 在这里填写要登录的用户
set username "worker"
# 在这里填写要登录的密码
set password "worker"

spawn ssh $username@$host
set timeout 2
expect {
    "*password:" {
        send "$password\n"
    }
    "Are you sure you want to continue connecting (yes/no)?" {
        send "yes\r"
        exp_continue
    }
}
expect "*#"
interact
```

执行`gw 10.1.1.8`，即可登录到对应的主机上。

# 本地主机免登录relay服务器，并自动登录到对应的服务器

在本地自动登录relay主机同样使用expect的方式，脚本名称为wrelay，内容如下：

```
#!/usr/bin/expect

if {$argc < 1} {
    puts "Usage:cmd <remote_host>"
    exit 1
}

# 下面指定relay主机
set host "relay.name"
# 这里输入relay的用户名
set username ""
# 这里输入relay的密码
set password ""
set remote_host [lindex $argv 0]

spawn ssh $username@$host
set timeout 2
expect {
    "*password:" {
        send "$password\n"
    }
    "Are you sure you want to continue connecting (yes/no)?" {
        send "yes\r"
        exp_continue
    }
}
expect "*#"

sleep 0.1
# 在relay上自动登录到其他服务器主机
send "gw $remote_host\n"
interact
```

