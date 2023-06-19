---
title: Linux中信号处理举例
Status: public
url: linux_signal_deal_example
tags: Hadoop
date: 2013-06-14 23:10:36
---

# Linux处理ctrl+c信号的例子
当按下ctrl+c时如果代码正在执行sleep则会停止睡眠，调用信号处理函数。中断位置可能位于for循环代码段的任意位置，中断位置不可控。
```c
#include <stdio.h>
#include <signal.h>
void h(int s)
{
	printf("抽空处理int信号\n");
}
main()
{
	int sum=0;
	int i;
	signal(SIGINT,h);
	sigset_t sigs;
    
	for(i=1;i<=10;i++)
	{
		sum+=i;
		sleep(1);
	}
	printf("sum=%d\n",sum);
	printf("Over!\n");
}
```

# 信号屏蔽的例子1
当按下ctrl+c时不会调用信号处理函数，当循环执行完毕后会调用信号处理函数，并且printf("Over!\n")会被执行。
```c
#include <stdio.h>
#include <signal.h>

void h(int s)
{
	printf("抽空处理int信号\n");
}

main()
{
    int sum=0;
    int i;
    // 声明信号集合
    sigset_t sigs;
    signal(SIGINT,h);
    // 清空集合
    sigemptyset(&sigs);
    // 加入屏蔽信号
    sigaddset(&sigs,SIGINT);
    // 屏蔽信号
    sigprocmask(SIG_BLOCK,&sigs,0);
    for(i=1;i<=10;i++)
    {
        sum+=i;
        sleep(1);
    }
    printf("sum=%d\n",sum);
    // 消除屏蔽信号
    sigprocmask(SIG_UNBLOCK,&sigs,0);
    // 如果在上面按下ctrl+c，在此句不执行
    printf("Over!\n");
}
```
当在循环中按下ctrl+c后，该函数输出结果为：
```
sum=55
抽空处理int信号
Over!
```
# 信号屏蔽的例子2
```c
#include <stdio.h>
#include <signal.h>
// 信号处理函数
void h(int s)
{
	printf("抽空处理int信号\n");
}
main()
{
	int sum=0;
	int i;
	signal(SIGINT,h);
	sigset_t sigs,sigp,sigq;
	sigemptyset(&sigs);
	sigemptyset(&sigp);
	sigemptyset(&sigq);
	
	sigaddset(&sigs,SIGINT);
	sigprocmask(SIG_BLOCK,&sigs,0);
	for(i=1;i<=10;i++)
	{
		sum+=i;
		sigpending(&sigp);
		if(sigismember(&sigp,SIGINT))
		{
			printf("SIGINT在排队!\n");
			// 是信号SIGINT有效
			sigsuspend(&sigq);
			// 函数调用完毕后信号SIGINT无效
		}
		sleep(1);
	}
	printf("sum=%d\n",sum);
    // 消除屏蔽信号
	sigprocmask(SIG_UNBLOCK,&sigs,0);
	printf("Over!\n");
}
```
该例子可以实现在指定的代码处处理信号。
其中sigsuspend函数原先如下：
```
int sigsuspend(const sigset_t *mask);
```
函数解释：屏蔽新的信号，原来的屏蔽信号失效。是一个阻塞函数，该函数屏蔽mask信号；对非mask信号不屏蔽，信号处理函数调用完毕该函数返回；如果非mask信号没有信号处理函数，则此函数不返回。即返回条件：信号发生且信号为非屏蔽信号且信号必须要调用信号处理函数完毕。