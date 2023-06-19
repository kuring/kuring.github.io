---
title: Linux下父进程定期杀死超时子进程的例子
Status: public
url: linux_kill_child_process
date: 2014-05-21
---

在Linux下会父进程通过fork()出的子进程可能会由于某种原因死锁或睡眠而无法终止，这时候需要父进程杀死子进程。本程序是父进程检测到子进程运行一段时间后杀死子进程的例子。

父进程的检测代码如下：

```c
#include <stdio.h>

#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(int argc, char *argv[])
{
    int pid = fork();
    if (pid > 0)
    {
        // parent process
        int count = 0;
        while (1)
        {
            pid_t result = waitpid(pid, NULL, WNOHANG);
            if (result == 0)
            {
                printf("child process is running\n");
                count++;
                if (count >= 60 * 60)   // one hour
                {
                    kill(pid, 9);   // kill child process
                }
            }
            else if (result == pid)
            {
                printf("child process has exit\n");
                break;
            }
            else
            {
                printf("result=%d\n", result);
            }
            sleep(1);
        }
    }
    else if (pid == 0)
    {
        // child process
        execv("/home/kuring/source/child", NULL);
        _exit(1);
    }
    else
    {
        printf("fork() error\n");
        return -1;
    }
    return 0;   
}
```

子进程调用execv()函数执行的child代码如下：

```c
#include <stdio.h>

#include <unistd.h>

int main(int argc, char *argv[])
{
    while (1)
    {
        sleep(1);
    }
    return 1;
}
```

例子比较简单，不作过多解释。