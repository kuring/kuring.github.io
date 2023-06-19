---
title: linux中创建新进程的方式
url: linux_create_process_methods
tags: Linux
date: 2013-06-07 16:08:15
---


# system函数
函数包含在C语言的标准库中，在头文件stdlib.h中声明如下：
```
int system(const char *command);
```
该函数会创建一个独立的进程，该进程拥有独立的进程空间，为阻塞函数，只有当新进程执行完毕该函数才返回。

返回值：可以通过返回值来获取新进程的main函数的返回值，返回值保存在int类型的第二个字节即8-15比特，可以通过向右移位或者利用宏WEXITSTATUS(status)来获取新进程的返回值。其中WEXITSTATUS(status)宏包含在头文件<sys/wait.h>中。

这里以调用ls命令为例来展示用法：
```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
int main()
{
    int r=system("ls");
    // 用右移位的方式来获取新进程的返回值
    //printf("%d\n",r>>8&255);
    // 用WEXITSTATUS(status)宏的方式来获取新进程的返回值
    printf("%d\n",WEXITSTATUS(r));
}
```
# popen函数
函数包含在头文件stdio.h中，相关函数如下：
```c
FILE *popen(const char *command, const char *type);
int pclose(FILE *stream);
```
popen函数在父子进程之间建立一个管道，其中type指定管道的类型，可以为"r"或"w"即只读或可写。在shell中的管道符"|"即采用此函数来实现。popen函数为阻塞函数，函数的具体用法如下：
```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
main()
{
    char buf[1024];
    FILE *f=popen("ls","r");
    // 根据管道获取文件描述符
    int fd=fileno(f);  
    int r;     
    while((r=read(fd,buf,1024))>0)
    {   
        buf[r]=0;
        printf("%s\n",buf);
    } 
    close(fd);
    // 关闭管道
    pclose(f);  
}
```
# exec系列函数
该系列函数并不创建新的进程，而是将程序加载到当前进程的代码空间来执行并替换当前进程的代码空间，在exec*函数后面的代码将无法执行。
```c
int execl(const char *path, const char *arg, ...);
int execlp(const char *file, const char *arg, ...);
int execle(const char *path, const char *arg, ..., char * const envp[]);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
```
# fork函数
该函数非常常用。函数原型如下：
```c
pid_t fork(void);
```
调用该函数会产生一个子进程，该子进程不仅复制了父进程的代码空间、堆、栈，而且还复制了父进程的执行位置。之后父子进程同时执行，通常由于操作系统任务调度的原因，子进程会先执行。父进程和子进程之间的并不会共享堆、栈上的数据，可以通过文件或共享内存的方式来通讯。

返回值：该函数父进程返回子进程的id，子进程返回0。通常在代码中通过返回值来判断是子进程还是父进程，用来执行不同的代码。

如果父进程先结束，则子进程会成为孤儿进程，子进程仍然可以继续执行。进程数中的根进程init会成为该子进程的父进程。

如果子进程先结束，则子进程会成为僵尸进程。僵尸进程并不再占用内存和CPU资源，但是会在进程数中看到僵尸进程。因此代码中必须对僵尸进程的情况做处理，通常的处理办法为采用wait函数和信号机制。子进程在结束的时候会向父进程发送一个信号SIGCHLD，整数值为17。父进程扑捉到该信号后通过调用wait函数来回收子进程的资源。其中wait函数为阻塞函数，会一直等待子进程结束。wait函数返回子进程退出时的状态码。下面通过实例演示一下当子进程先结束时父进程怎样回收子进程。
```c
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

void processChildProcess()
{
    printf("receive the child process end\n");
    wait();
}

int main()
{
    int pid;
    pid = fork();
    if (pid > 0)
    {   
        // 父进程
        signal(SIGCHLD, processChildProcess);
        while(1)
        {   
            sleep(1);
        }   
    }   
    else
    {   
        // 子进程
        sleep(1);
        printf("child process end\n");
    }   
}
```
关于fork的详细理解可以看这里：[一个fork的面试题](http://coolshell.cn/articles/7965.html)。