---
title: 一个Linux下的监听脚本程序
Status: public
url: linux_listen_shell
date: 2014-11-06
---

为了避免linux下的控制台程序A死掉，可以通过一个另外一个程序B来监听A程序，当A程序异常退出时将B程序带起来。当然程序设计的最好方式为程序不崩溃，但是程序中存在bug很难避免，该方法还是有一定的实践意义。对于B程序可以通过shell脚本或者单独一个应用程序来解决。本文将通过shell脚本来解决此问题。

# shell脚本的内容

```shell
#!/bin/bash

check_process()
{
	# check parameter
	if [ $1 = "" ];
	then
		return -1
	fi

	# get the running process
	process_names=$(ps -ef | grep $1 | grep -v grep | awk '{print $8}')
	for process_name in $process_names
	do
		if [ $process_name = $1 ] ;
		then
			return 1
		fi
	done

	# not run and run the process
	echo "$(date) : process $1 not run, just run it"
	$1
	return 0
}

while [ 1 ];do
	check_process "/usr/bin/app/process"	# programe path
	sleep 5
done
```

# 将shell脚本在脱离控制台下可以运行

一旦断开了控制台，shell脚本就会由于接收到SIGHUP信号而退出。这里有两种思路来解决该问题，一种是通过系统的crontab来定期调用脚本程序，另外一种是通过神奇的screen程序来解决该问题，我这里通过screen程序来解决该问题，具体screen程序的应用见我的另外一篇文章《》。

# 应用程序为daemon方式运行

为了能够保证该脚本监控多个应用程序，需要将应用程序设置为daemon方式运行，可以调用函数daemon实现。也可以调用单独实现的daemon函数，具体代码如下：

```
void init_daemon(void) 
{ 
    int pid; 
    int i;  

    if(pid=fork()) 
        exit(0);//是父进程，结束父进程 
    else if(pid< 0)  
        exit(1);//fork失败，退出 
    //是第一子进程，后台继续执行 

    setsid();//第一子进程成为新的会话组长和进程组长 
    //并与控制终端分离 
    if(pid=fork()) 
        exit(0);//是第一子进程，结束第一子进程 
    else if(pid< 0)  
        exit(1);//fork失败，退出 
    //是第二子进程，继续 
    //第二子进程不再是会话组长 

    for(i=0;i< NOFILE;++i)//关闭打开的文件描述符 
        close(i); 
    chdir("/tmp");//改变工作目录到/tmp 
    umask(0);//重设文件创建掩模 
    return; 
} 
```
