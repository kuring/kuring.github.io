---
title: UNIX网络编程读书笔记
Status: public
url: linux_unp
date: 2014-12-22
tags: linux 读书笔记
---

第一遍阅读unpv3后，对书中讲述的内容有了大体的认识，但对书中的具体细节地方却是早已忘记。重新阅读unpv3，这次不希望仍然是阅后即忘，于是通过编写代码的方式对书中的例子和注意事项加深理解。

为了能够将书中的很多细节问题理解清楚并且便于记忆，本文采用了编写书中代码并运行的方式，并将书中容易出错和意想不到的问题记在代码中。

本文的代码实例并未完全按照书中的代码实例，本着单个文件即能编译通过并运行的原则，本文对于很多系统调用并未做防御式编程处理。针对每个版本的程序中缺点和注意事项在代码中已经进行了标注。

鉴于高性能的epoll机制出现比较晚，晚于unp的编写时间，书中并未做介绍。

# TCP客户端程序

客户端函数执行效率情况：select非阻塞式I/O版本>线程化版本>fork版本>select阻塞式I/O版本>停等版本，停等版本的执行效率非常低，在实际生产环境中不建议使用。

其中poll和select机制基本类似，书中并未给出poll版本。

## 停-等版本

最常规的实现思路，但效率非常低，且当程序阻塞在读取要发送内容时，程序是无法收到服务端的状态变化。

```c
/**
 * 停-等版本
 * 该版本缺陷为当服务端发生某些事件时，客户端可能仍然阻塞于fgets调用中
 */
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <signal.h>

#define SERV_PORT 9877
#define MAXLINE 4096	/* max text line length */

void str_cli(FILE *fp, int sockfd)
{
	char sendline[MAXLINE], recvline[MAXLINE];
	while (fgets(sendline, MAXLINE, fp) != NULL)
	{
		/* 当阻塞在fgets函数时将服务器进程关闭时虽然给客户端发送了FIN信号，客户端并不会知道，
		 * 服务端关闭时第一次调用write服务器会返回RST，
		 * 当一个进程向某个收到RST的套接字执行写操作时，内核会向该进程发送一个SIGPIPE信号
		 * 该问题需要使用I/O复用技术来解决，或者使用fork处理的方式来解决
		 * */
		write(sockfd, sendline, strlen(sendline));
		int n = read(sockfd, recvline, MAXLINE);
		if (n == 0)
		{
			printf("str_cli: server terminated prematurely\n");
			exit(1);
		}

		// 向标准输出写内容，既可以使用write也可以使用fputs
		write(STDOUT_FILENO, recvline, n);

		// 使用fputs时需要注意将recvline数组有效内容的后面一位设置为'\0'
//		recvline[n] = '\0';
//		fputs(recvline, stdout);
	}
}

int main(int argc, char **argv)
{
	int sockfd;
	struct sockaddr_in	servaddr;

	if (argc != 2)
	{
		printf("usage: tcpcli <IPaddress>\n");
		exit(1);
	}

	// 当一个进程向某个收到RST的套接字执行写操作时，内核会向该进程发送一个SIGPIPE信号
	// 最好的方式是忽略此信号的处理方式，并在程序下面处理该异常情况
	signal(SIGPIPE, SIG_IGN);

	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port = htons(SERV_PORT);
	inet_pton(AF_INET, argv[1], &servaddr.sin_addr);

	connect(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr));

	str_cli(stdin, sockfd);		/* do it all */

	exit(0);
}
```

## fork版本

```c
/**
 * 阻塞式I/O的fork版本
 */
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <signal.h>

#define SERV_PORT 9877
#define MAXLINE 4096	/* max text line length */

/**
 * 即使服务端已经退出，子进程的read方法仍然能够感知到并且退出while循环，并给父进程发送SIGTERM,父进程对该信号的默认处理方式为退出
 * 优点：代码量比较少，每个进程只处理2个I/O流，从一个复制到另一个
 */
void str_cli(FILE *fp, int sockfd)
{
	char sendline[MAXLINE], recvline[MAXLINE];
	pid_t pid;
	if ((pid = fork()) == 0)
	{
		// child process : server -> stdout
		int n;
		while ((n = read(sockfd, recvline, MAXLINE)) > 0)
		{
			recvline[n] = '\0';
			fputs(recvline, stdout);
		}
		kill(getppid(), SIGTERM);
		exit(0);
	}

	// parent process : stdin -> server
	while (fgets(sendline, MAXLINE, fp) != NULL)
	{
		write(sockfd, sendline, strlen(sendline));
	}
	shutdown(sockfd, SHUT_WR);
	pause();
	return ;
}

int main(int argc, char **argv)
{
	int sockfd;
	struct sockaddr_in	servaddr;

	if (argc != 2)
	{
		printf("usage: tcpcli <IPaddress>\n");
		exit(1);
	}

	// 当一个进程向某个收到RST的套接字执行写操作时，内核会向该进程发送一个SIGPIPE信号
	// 最好的方式是忽略此信号的处理方式，并在程序下面处理该异常情况
	signal(SIGPIPE, SIG_IGN);

	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port = htons(SERV_PORT);
	inet_pton(AF_INET, argv[1], &servaddr.sin_addr);

	if (connect(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) != 0)
	{
		printf("connect error...\n");
		exit(1);
	}

	str_cli(stdin, sockfd);		/* do it all */

	exit(0);
}
```

## 阻塞式I/O的select版本

```c
/**
 * 阻塞式I/O的select版本
 */
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <signal.h>

#define SERV_PORT 9877
#define MAXLINE 4096	/* max text line length */

/**
 * 缺点：使用了阻塞式I/O，如果在向套接字调用write发送给服务器时，套接字缓冲区已满，write调用会阻塞，从而影响了后续的套接字缓冲区的读取
 */
void str_cli(FILE *fp, int sockfd)
{
	int maxfdp1;
	fd_set rset;
	char sendline[MAXLINE], recvline[MAXLINE];
	int stdineof = 0;
	FD_ZERO(&rset);
	for (; ;)
	{
		// select
		FD_SET(fileno(fp), &rset);
		FD_SET(sockfd, &rset);
		maxfdp1 = (fileno(fp) > sockfd ? fileno(fp) : sockfd) + 1;
		select(maxfdp1, &rset, NULL, NULL, NULL);

		// socket
		if (FD_ISSET(sockfd, &rset))
		{
			int n = read(sockfd, recvline, MAXLINE);
			if (n == 0)
			{
				if (stdineof == 1)
				{
					return ;
				}
				else
				{
					printf("str_cli: server terminated prematurely\n");
					exit(1);
				}
			}
			else if (n == -1)
			{
				exit(1);
			}
			recvline[n] = '\0';
			fputs(recvline, stdout);
//			write(STDOUT_FILENO, recvline, n);
		}

		// input
		if (FD_ISSET(fileno(fp), &rset))
		{
			// 此处不能使用fgets函数，该函数带有缓冲区功能，select跟带有缓冲区的c函数混合使用有问题
//			if (fgets(sendline, MAXLINE, fp) == NULL)
//			{
//				return ;
//			}
			int n = read(fileno(fp), sendline, MAXLINE);
			if (n == 0)
			{
				stdineof = 1;
				shutdown(sockfd, SHUT_WR);	// 关闭写
				FD_CLR(fileno(fp), &rset);
				continue;
			}
			else if (n == -1)
			{
				exit(1);
			}
			write(sockfd, sendline, n);
		}
	}
}

int main(int argc, char **argv)
{
	int sockfd;
	struct sockaddr_in	servaddr;

	if (argc != 2)
	{
		printf("usage: tcpcli <IPaddress>\n");
		exit(1);
	}

	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port = htons(SERV_PORT);
	inet_pton(AF_INET, argv[1], &servaddr.sin_addr);

	if (connect(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) != 0)
	{
		printf("connect error...\n");
		exit(1);
	}

	str_cli(stdin, sockfd);		/* do it all */

	exit(0);
}
```

## 非阻塞式I/O的select版本

```c
/**
 * 非阻塞式I/O的select版本
 */
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <signal.h>
#include <fcntl.h>

#define SERV_PORT 9877
#define MAXLINE 4096	/* max text line length */
#define max(a,b) ( ((a)>(b)) ? (a):(b) )

/**
 * 优点：速度是最快的，可以防止进程在做任何工作时发生阻塞
 * 缺点：同时管理4个不同的I/O流，每个流都是非阻塞的，需要考虑到4个流的部分读和部分写问题。编码量是最多的，需要引入缓冲区管理机制。
 */
void str_cli(FILE *fp, int sockfd)
{
	// 将socket、标准输入和标准输出描述符设置为非阻塞方式
	int val = fcntl(sockfd, F_GETFL, 0);
	fcntl(sockfd, F_SETFL, val | O_NONBLOCK);

	val = fcntl(STDIN_FILENO, F_GETFL, 0);
	fcntl(STDIN_FILENO, F_SETFL, val | O_NONBLOCK);

	val = fcntl(STDOUT_FILENO, F_GETFL, 0);
	fcntl(STDOUT_FILENO, F_SETFL, val | O_NONBLOCK);

	char to[MAXLINE], fr[MAXLINE];
	char *toiptr, *tooptr, *friptr, *froptr;
	toiptr = tooptr = to;
	friptr = froptr = fr;
	int stdineof = 0;

	int maxfdp1 = max(max(STDIN_FILENO, STDOUT_FILENO), sockfd) + 1;
	fd_set rset, wset;
	for (; ;)
	{
		FD_ZERO(&rset);
		FD_ZERO(&wset);
		if (stdineof == 0 && toiptr < &to[MAXLINE])
		{
			FD_SET(STDIN_FILENO, &rset);
		}
		if (friptr < &fr[MAXLINE])
		{
			FD_SET(sockfd, &rset);
		}
		if (tooptr != toiptr)
		{
			FD_SET(sockfd, &wset);
		}
		if (froptr != friptr)
		{
			FD_SET(STDOUT_FILENO, &wset);
		}
		select(maxfdp1, &rset, &wset, NULL, NULL);	// select函数仍然是阻塞的
		// 标准输入
		if (FD_ISSET(STDIN_FILENO, &rset))
		{
			int n;
			if ((n = read(STDIN_FILENO, toiptr, &to[MAXLINE] - toiptr)) < 0)
			{
				// 对于非阻塞式IO，如果操作不能满足，相应系统调用会返回EWOULDBLOCK错误
				if (errno != EWOULDBLOCK)
				{
					printf("read error on stdin\n");
					exit(1);
				}
			}
			else if (n == 0)
			{
				fprintf(stderr, "EOF on stdin\n");
				stdineof = 1;
				if (tooptr == toiptr)
				{
					shutdown(sockfd, SHUT_WR);	// 缓冲区中没有数据要发送，关闭socket
				}
			}
			else
			{
				fprintf(stderr, "read %d bytes from stdin\n", n);
				toiptr += n;
				FD_SET(sockfd, &wset);
			}
		}

		// 从套接字读
		if (FD_ISSET(sockfd, &rset))
		{
			int n;
			if ((n = read(sockfd, friptr, &fr[MAXLINE] - friptr)) < 0)
			{
				if (errno != EWOULDBLOCK)
				{
					printf("read error on socket\n");
					exit(1);
				}
			}
			else if (n == 0)
			{
				fprintf(stderr, "EOF on socket\n");
				if (stdineof)
				{
					return ;
				}
				else
				{
					printf("server terminated prematurely\n");
					exit(1);
				}
			}
			else
			{
				fprintf(stderr, "read %d bytes from socket\n", n);
				friptr += n;
				FD_SET(STDOUT_FILENO, &wset);
			}
		}

		// 标准输出
		int n;
		if (FD_ISSET(STDOUT_FILENO, &wset) && ((n = friptr - froptr) > 0))
		{
			int nwritten;
			if ((nwritten = write(STDOUT_FILENO, froptr, n)) < 0)
			{
				if (errno != EWOULDBLOCK)
				{
					printf("write error to stdout\n");
					exit(1);
				}
			}
			else
			{
				fprintf(stderr, "wrote %d bytes to stdout\n", nwritten);
				froptr += nwritten;
				if (froptr == friptr)
				{
					froptr = friptr = fr;
				}
			}
		}

		// 向socket写
		if (FD_ISSET(sockfd, &wset) && ((n = toiptr - tooptr)) > 0)
		{
			int nwritten;
			if ((nwritten = write(sockfd, tooptr, n)) < 0)
			{
				if (errno != EWOULDBLOCK)
				{
					printf("write error to socket\n");
					exit(1);
				}
			}
			else
			{
				fprintf(stderr, "wrote %d bytes to socket\n", nwritten);
				tooptr += nwritten;
				if (tooptr == toiptr)
				{
					toiptr = tooptr = to;
					if (stdineof)
					{
						shutdown(sockfd, SHUT_WR);
					}
				}
			}
		}
	}

	return ;
}

/**
 * connect的非阻塞版本
 * 连接建立成功时，描述符变为可写；连接建立错误时，描述符变为即可读又可写
 * 优点：
 * 1、阻塞式的connect调用会消耗CPU时间，非阻塞式connect可以充分利用CPU时间，在等待的过程中可以处理其他工作
 * 2、可以同时建立多个连接，浏览器中会用到此技术
 * 3、阻塞式connect的函数超时过长，可以通过该函数设置超时时间
 * 4、阻塞式的套接字调用connect时，在TCP的三次握手完成之前被某些信号中断时并且connect未设置内核自动重启的标志时，connect将返回EINTR错误
 * 当再次调用connect等待未完成的连接时将会返回EADDRINUSE错误
 */
int connect_nonb(int sockfd, const struct sockaddr *saptr, socklen_t salen, int nsec)
{
	// 将套接字设置为非阻塞状态
	int flags = fcntl(sockfd, F_GETFL, 0);
	fcntl(sockfd, F_SETFL, flags | O_NONBLOCK);

	int error = 0;

	int n;
	if ((n = connect(sockfd, saptr, salen)) < 0)
	{
		// 连接未成功建立，正常情况下返回EINPROGRESS错误，表示操作正在处理
		if (errno != EINPROGRESS)
		{
			// EINPROGRESS表示连接建立已经启动，但是尚未完成
			return -1;
		}
	}
	else if (n == 0)
	{
		// 当服务器和客户端在一台主机上时会立即建立连接
		goto done;
	}

	// 当代码执行到如下过程中时，connect正在建立连接，可以在此位置执行业务相关代码
	// 当然真正使用时，在此位置加入其他代码并不合适，需要根据具体情况重新调整代码
	// 可以参照书中的web客户程序例子

	fd_set rset, wset;
	FD_ZERO(&rset);
	FD_SET(sockfd, &rset);
	wset = rset;

	struct timeval tval;
	tval.tv_sec = nsec;
	tval.tv_usec = 0;
	if ((n = select(sockfd + 1, &rset, &wset, NULL, nsec ? &tval : NULL)) == 0)
	{
		// 发生超时
		close(sockfd);
		errno = ETIMEDOUT;
		return -1;
	}

	// 当连接建立成功时sockfd变为可写，当连接建立失败时sockfd变为即可读又可写
	if (FD_ISSET(sockfd, &rset) || FD_ISSET(sockfd, &wset))
	{
		int len = sizeof(error);
		// 非可移植性函数，连接建立成功返回0，连接建立失败将错误值返回给error
		// 连接建立失败时，有返回-1和返回0的情况
		if (getsockopt(sockfd, SOL_SOCKET, SO_ERROR, &error, &len) < 0)
		{
			// solaris连接建立失败返回-1
			return -1;
		}
	}
	else
	{
		printf("select error:sockfd not set");
		exit(1);
	}
done:
	// 恢复套接字的文件状态标志
	fcntl(sockfd, F_SETFL, flags);
	if (error)
	{
		close(sockfd);
		errno = error;
		return -1;
	}
	return 0;
}

int main(int argc, char **argv)
{
	int sockfd;
	struct sockaddr_in	servaddr;

	if (argc != 2)
	{
		printf("usage: tcpcli <IPaddress>\n");
		exit(1);
	}

	// 当一个进程向某个收到RST的套接字执行写操作时，内核会向该进程发送一个SIGPIPE信号
	// 最好的方式是忽略此信号的处理方式，并在程序下面处理该异常情况
	signal(SIGPIPE, SIG_IGN);

	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port = htons(SERV_PORT);
	inet_pton(AF_INET, argv[1], &servaddr.sin_addr);

	//connect(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr));
	if (connect_nonb(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr), 50) < 0)
	{
		printf("socket connect error\n");
		exit(1);
	}

	str_cli(stdin, sockfd);		/* do it all */

	exit(0);
}
```

# TCP服务端程序

服务器程序要处理大量并发，在设计时更要注重效率。

## fork版本

```c
/**
 * fork版本
 * PPC(Process per Connection)模型
 */
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <stdio.h>
#include <signal.h>
#include <strings.h>

#define SERV_PORT 9877
#define MAXLINE		4096	/* max text line length */

void sig_chld(int signo)
{
	if (signo != SIGIO)
	{
		return;
	}
	int stat;

	/* 此处不可以使用wait函数，当多个SIGCHLD信号同时发出时会因为信号覆盖而出现僵尸进程的情况
	pid_t pid = wait(&stat);
	printf("child %d terminated\n", pid);	// 非异步信号安全函数，此处不应该调用
	*/

	/* 使用非阻塞的参数WNOHANG来循环处理信号，避免信号丢失问题 */
	pid_t pid;
	while ((pid = waitpid(-1, &stat, WNOHANG)) > 0)
	{
		printf("child %d terminated\n", pid);
	}
}

void str_echo(int sockfd)
{
	ssize_t	n;
	char buf[MAXLINE];

again:
	while ( (n = read(sockfd, buf, MAXLINE)) > 0)
	{
		write(sockfd, buf, n);
	}

	if (n < 0 && errno == EINTR)
		goto again;
	else if (n < 0)
		printf("str_echo: read error\n");
}

/**
 * fork版本
 * 缺点：
 * 		1.fork需要将父进程的内存映像复制到子进程，并在子进程中复制所有的描述符，尽管现在的操作系统已经都实现了写时复制技术，但是耗时仍然比较多
 * 		2.父进程和子进程之间需要IPC机制进行通信，从子进程返回信息到父进程比较麻烦
 */
int main(int argc, char *argv[])
{
	signal(SIGCHLD, sig_chld);
    int listenfd, connfd;
    pid_t childpid;
    struct sockaddr_in cliaddr, servaddr;

    // socket
    listenfd = socket(AF_INET, SOCK_STREAM, 0);
    if (listenfd == -1)
    {
		printf("socket error\n");
		exit(1);
	}

    // bind
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(SERV_PORT);
    if (bind(listenfd,  (struct sockaddr*)&servaddr, sizeof(servaddr)) == -1)
    {
    	printf("bind error\n");
    	exit(1);
    }

    // listen
    // 套接字排队的最大连接数为20
    if (listen(listenfd, 20) == -1)
    {
    	printf("listen error\n");
    	exit(1);
    }

    for ( ; ; )
    {
    	socklen_t clilen = sizeof(cliaddr);
    	// 处理accept被信号中断时返回EINTR错误
		if ((connfd = accept(listenfd, (struct sockaddr*)&cliaddr, &clilen)) < 0)
		{
			if (errno == EINTR)
			{
				continue;
			}
			else
			{
				printf("accept error");
				exit(1);
			}
		}

		if ((childpid = fork()) == 0)
		{
			/* child process */
			close(listenfd); /* close listening socket */
			str_echo(connfd); /* process the request */
			exit(0);
		}
		close(connfd); /* parent closes connected socket */
	}
}
```

## select版本

```c
/**
 * select版本
 * 缺点：
 * 		1. 有最大并发数限制，一个进程最多打开FD_SETSIZE个文件描述符，FD_SETSIZE往往是1024或2048字节
 * 		2. select每次调用都会线性扫描全部的FD集合，这样效率就会呈现线性下降，把FD_SETSIZE改大的后果就是所有FD处理都慢慢来
 * 		3. 内核/用户空间内存拷贝问题，内核把FD消息通知给用户空间采取了内存拷贝方法
 */
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <stdio.h>
#include <signal.h>
#include <sys/select.h>
#include <strings.h>

#define SERV_PORT 9877
#define MAXLINE		4096	/* max text line length */

/**
 * 使用select的需要维护client数组和allset的描述符集
 */
int main(int argc, char *argv[])
{
    struct sockaddr_in cliaddr, servaddr;

    // socket
    int listenfd = socket(AF_INET, SOCK_STREAM, 0);
    if (listenfd == -1)
    {
    	printf("socket error\n");
    	exit(1);
    }
    printf("finish socket...\n");

    // bind
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(SERV_PORT);
    if (bind(listenfd,  (struct sockaddr*)&servaddr, sizeof(servaddr)) == -1)
    {
    	printf("bind error\n");
    	exit(1);
    }
    printf("finish bind...\n");

    // listen
    // 套接字排队的最大连接数为20
    if (listen(listenfd, 20) == -1)
    {
    	printf("listen error\n");
    	exit(1);
    }
    printf("finish listening...\n");

    int maxfd = listenfd;
    int maxi = -1;
    int client[FD_SETSIZE];
    for (int i=0; i<FD_SETSIZE; i++)
    {
    	client[i] = -1;
    }
    fd_set allset;
    FD_ZERO(&allset);
    FD_SET(listenfd, &allset);

    for ( ; ; )
    {
    	fd_set rset = allset;
    	int nready = select(maxfd + 1, &rset, NULL, NULL, NULL);
    	if (FD_ISSET(listenfd, &rset))
    	{
    		// 设置client数组
    		socklen_t clilen = sizeof(cliaddr);
    		// 调用select时有个问题，见书中16.6节
    		// 如果调用accept时客户端已经关闭连接，此时accept会阻塞并直到新的客户端连接到来
    		// 为了解决该问题可以将套接字设置为非阻塞再调用accept
			int connfd = accept(listenfd, (struct sockaddr*)&cliaddr, &clilen);
			printf("accept one client:%d...\n", connfd);
			int i = 0;
			for (; i<FD_SETSIZE; i++)
			{
				if (client[i] < 0)
				{
					client[i] = connfd;
					break;
				}
			}

			FD_SET(connfd, &allset);

			if (i == FD_SETSIZE)
			{
				printf("too many clients");
				exit(-1);
			}
			if (connfd > maxfd)
			{
				maxfd = connfd;
			}
			if (i > maxi)
			{
				maxi = i;
			}
			if (--nready <= 0)
			{
				continue;
			}
    	}

    	// 检测所有客户端的数据
    	for (int i=0; i<=maxi; i++)
    	{
    		if (client[i] < 0)
    		{
    			continue;
    		}
    		if (FD_ISSET(client[i], &rset))
    		{
    			int n;
    			char buf[MAXLINE];
    			printf("start reading form one client...\n");
    			if ((n = read(client[i], buf, MAXLINE)) == 0)
				{
    				close(client[i]);
    				FD_CLR(client[i], &allset);
    				client[i] = -1;
				}
    			else
    			{
    				write(client[i], buf, n);
    			}
    			if (--nready <= 0)
    			{
    				break;
    			}
    		}
    	}
	}
}
```

## poll版本

```c
/**
 * poll版本
 * poll版本的解决了select文件描述符限制问题，但是仍然具备select的缺点中的2和3
 */
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <stdio.h>
#include <signal.h>
#include <strings.h>
#include <poll.h>
#include <stropts.h>

#define SERV_PORT 9877
#define MAXLINE		4096	/* max text line length */
#define OPEN_MAX 1024  // 该宏已经从limit.h中移除，用来表示一个进程可以打开的最大描述符数目

/**
 * 使用select的缺点为需要维护client数组
 */
int main(int argc, char *argv[])
{
    struct sockaddr_in cliaddr, servaddr;

    // socket
    int listenfd = socket(AF_INET, SOCK_STREAM, 0);
    if (listenfd == -1)
    {
    	printf("socket error\n");
    	exit(1);
    }
    printf("finish socket...\n");

    // bind
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(SERV_PORT);
    if (bind(listenfd,  (struct sockaddr*)&servaddr, sizeof(servaddr)) == -1)
    {
    	printf("bind error\n");
    	exit(1);
    }
    printf("finish bind...\n");

    // listen
    // 套接字排队的最大连接数为20
    if (listen(listenfd, 20) == -1)
    {
    	printf("listen error\n");
    	exit(1);
    }
    printf("finish listening...\n");

    struct pollfd client[OPEN_MAX];
    client[0].fd = listenfd;
    client[0].events = POLLIN;
    for (int i=1; i<OPEN_MAX; i++)
    {
    	client[i].fd = -1;
    }

    int maxi = 0;	// 当前client正在使用的最大下标

    for ( ; ; )
    {
    	int nready = poll(client, maxi + 1, -1);
    	if (client[0].revents & POLLIN)
    	{
    		// 设置client数组
    		socklen_t clilen = sizeof(cliaddr);
			int connfd = accept(listenfd, (struct sockaddr*)&cliaddr, &clilen);
			printf("accept one client:%d...\n", connfd);
			int i = 1;
			for (; i<OPEN_MAX; i++)
			{
				if (client[i].fd < 0)
				{
					client[i].fd = connfd;
					break;
				}
			}

			if (i == OPEN_MAX)
			{
				printf("too many clients");
				exit(-1);
			}
			client[i].events = POLLIN;
			if (i > maxi)
			{
				maxi = i;
			}
			if (--nready <= 0)
			{
				continue;
			}
    	}

    	// 检测所有客户端的数据
    	for (int i=0; i<=maxi; i++)
    	{
    		if (client[i].fd < 0)
    		{
    			continue;
    		}
    		if (client[i].revents & (POLLIN | POLLERR))
    		{
    			int n;
    			char buf[MAXLINE];
    			printf("start reading form one client...\n");
    			if ((n = read(client[i].fd, buf, MAXLINE)) < 0)
				{
    				if (errno == ECONNRESET)
    				{
    					close(client[i].fd);
    					client[i].fd = -1;
    				}
    				else
    				{
    					printf("read client error\n");
    					exit(-1);
    				}
				}
    			else if (n == 0)
    			{
    				printf("client %d close\n", client[i].fd);
    				close(client[i].fd);
    				client[i].fd = -1;
    			}
    			else
    			{
    				write(client[i].fd, buf, n);
    			}
    			if (--nready <= 0)
    			{
    				break;
    			}
    		}
    	}
	}
}
```

## 多线程版本

```c
/**
 * 多线程版本
 * TPC(Thread Per Connection)模型
 * 线程的开销虽然比进程小，但是仍然有比较大开销，因此并发数不是很高
 */
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <stdio.h>
#include <signal.h>
#include <strings.h>
#include <pthread.h>

#define SERV_PORT 9877
#define MAXLINE		4096	/* max text line length */

void str_echo(int sockfd)
{
	ssize_t	n;
	char buf[MAXLINE];

again:
	while ( (n = read(sockfd, buf, MAXLINE)) > 0)
	{
		write(sockfd, buf, n);
	}

	if (n < 0 && errno == EINTR)
		goto again;
	else if (n < 0)
		printf("str_echo: read error\n");
}

static void *doit(void *arg)
{
	pthread_detach(pthread_self());
	str_echo((int)arg);
	close((int)arg);
	printf("close socket...\n");
	return NULL;
}

int main(int argc, char *argv[])
{
    int listenfd, connfd;
    struct sockaddr_in cliaddr, servaddr;

    // socket
    listenfd = socket(AF_INET, SOCK_STREAM, 0);
    if (listenfd == -1)
    {
		printf("socket error\n");
		exit(1);
	}

    // bind
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(SERV_PORT);
    if (bind(listenfd,  (struct sockaddr*)&servaddr, sizeof(servaddr)) == -1)
    {
    	printf("bind error\n");
    	exit(1);
    }

    // listen
    // 套接字排队的最大连接数为20
    if (listen(listenfd, 20) == -1)
    {
    	printf("listen error\n");
    	exit(1);
    }

    for ( ; ; )
    {
    	socklen_t clilen = sizeof(cliaddr);
    	// 处理accept被信号中断时返回EINTR错误
		if ((connfd = accept(listenfd, (struct sockaddr*)&cliaddr, &clilen)) < 0)
		{
			if (errno == EINTR)
			{
				continue;
			}
			else
			{
				printf("accept error");
				exit(1);
			}
		}
		printf("receive new client...\n");
		pthread_t tid;
		pthread_create(&tid, NULL, &doit, (void *)connfd);
	}
}
```

# UDP

由于udp比较简单，书中并未将udp协议当做重点来讲解。

## UDP客户端程序

```c
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <stdio.h>
#include <signal.h>
#include <strings.h>

#define SERV_PORT 9877
#define MAXLINE		4096	/* max text line length */

// sendto、recvfrom方式
void dg_cli(FILE *fp, int sockfd, const struct sockaddr *pservaddr, socklen_t servlen)
{
	char sendline[MAXLINE], recvline[MAXLINE + 1];
	struct sockaddr *preply_addr = (struct sockaddr *)malloc(servlen);

	while (fgets(sendline, MAXLINE, fp) != NULL)
	{
		sendto(sockfd, sendline, strlen(sendline), 0, pservaddr, servlen);
		int len = servlen;
		int n = recvfrom(sockfd, recvline, MAXLINE, 0, preply_addr, &len);
		// 为了防止接收到其他进程的数据，通过条件判断去除
		if (len != servlen || memcmp(pservaddr, preply_addr, len) != 0)
		{
			printf("reply from others (!ignore)\n");
			continue;
		}
		recvline[n] = 0;
		fputs(recvline, stdout);
	}
}

// connect、write、read方式
void dg_cli2(FILE *fp, int sockfd, const struct sockaddr *pservaddr, socklen_t servlen)
{
	char sendline[MAXLINE], recvline[MAXLINE + 1];

	connect(sockfd, (struct sockaddr *)pservaddr, servlen);

	while (fgets(sendline, MAXLINE, fp) != NULL)
	{
		write(sockfd, sendline, strlen(sendline));
		int n = read(sockfd, recvline, MAXLINE);
		recvline[n] = 0;
		fputs(recvline, stdout);
	}
}

int main(int argc, char *argv[])
{
	if (argc != 2)
	{
		printf("usage: tcpcli <IPaddress>\n");
		exit(1);
	}

	struct sockaddr_in servaddr;
	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port = htons(SERV_PORT);
	inet_pton(AF_INET, argv[1], &servaddr.sin_addr);

	int sockfd = socket(AF_INET, SOCK_DGRAM, 0);
	dg_cli2(stdin, sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr));
	exit(0);
}
```

## UDP服务端程序

```c
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <stdio.h>
#include <signal.h>
#include <strings.h>

#define SERV_PORT 9877
#define MAXLINE		4096	/* max text line length */

void dg_echo(int sockfd, struct sockaddr *pcliaddr, socklen_t clilen)
{
	char mesg[MAXLINE];
	for (;;)
	{
		socklen_t len = clilen;
		int n;
		bzero(mesg, MAXLINE);
		if ((n = recvfrom(sockfd, mesg, MAXLINE, 0,  pcliaddr, &len)) < 0)
		{
			close(sockfd);
			printf("recvfrom error, error=%m\n");
			exit(1);
		}
		printf("recv %s\n", mesg);
		sendto(sockfd, mesg, n, 0, pcliaddr, len);
	}
}

int main(int argc, char *argv[])
{
	int sockfd = socket(AF_INET, SOCK_DGRAM, 0);
	struct sockaddr_in servaddr, cliaddr;
	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);
	if (bind(sockfd,  (struct sockaddr*)&servaddr, sizeof(servaddr)) < 0)
	{
		printf("bind error\n");
		exit(1);
	}

	dg_echo(sockfd, (struct sockaddr*)&cliaddr, sizeof(cliaddr));
}
```

## UDP服务端信号驱动式I/O版本

信号驱动式I/O：进程执行I/O系统调用告知内核启动某个I/O操作，内核启动I/O操作后立即返回到进程。进程在I/O操作发生期间继续执行。当操作完成或遇到错误时，内核以进程在I/O系统调用中指定的方式通知进程。

```c
/**
 * 信号驱动式I/O在TCP套接字用途不大，该信号产生的过于频繁，它的出现并未指示发生的事情
 */
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <stdio.h>
#include <signal.h>
#include <strings.h>
#include <fcntl.h>

#define SERV_PORT 9877
#define MAXLINE		4096	/* max text line length */

static int sockfd;

#define MAXDG 4096


typedef struct
{
	void *dg_data;				// 实际数据
	size_t dg_len;				// 实际数据长度
	struct sockaddr *dg_sa;		// 包含客户端地址
	socklen_t dg_salen;			// 客户端地址长度
} DG;

#define QSIZE 8
static DG dg[QSIZE];			// 存放数据的环形缓冲区

static long cntread[QSIZE + 1];
// 需要处理的下一个数据元素的下标
static int iget;
// 存放数据元素的下一个位置
static int iput;
static int nqueue;				// 队列中的数据个数
static socklen_t clilen;

static void sig_hup(int signo)
{
	int i=0;
	for (; i <= QSIZE; i++)
	{
		printf("cntread[%d = %ld\n", i, cntread[i]);
	}
}

static void sig_io(int signo)
{
	int nread;
	// 为了解决非实时信号不排队问题，采用循环读取方式
	for (nread = 0; ; )
	{
		// 检查队列是否已满
		if (nread >= QSIZE)
		{
			printf("receive overflow\n");
			exit(1);
		}
		DG *ptr = &dg[iput];
		ptr->dg_salen = clilen;
		ssize_t len = recvfrom(sockfd, ptr->dg_data, MAXDG, 0, ptr->dg_sa, &ptr->dg_salen);
		if (len < 0)
		{
			if (errno == EWOULDBLOCK)
			{
				break;
			}
			else
			{
				printf("recvfrom error\n");
				exit(1);
			}
		}
		ptr->dg_len = len;
		nread++;
		nqueue++;
		if (++iput >= QSIZE)
		{
			iput = 0;
		}
	}
	cntread[nread]++;
}

void dg_echo(int sockfd_arg, struct sockaddr *pcliaddr, socklen_t clilen_arg)
{
	sockfd = sockfd_arg;
	clilen = clilen_arg;
	int i = 0;
	for (; i<QSIZE; i++)
	{
		dg[i].dg_data = malloc(MAXDG);
		dg[i].dg_sa = (struct sockaddr *)malloc(clilen);
		dg[i].dg_salen = clilen;
	}
	iget = iput = nqueue = 0;
	signal(SIGHUP, sig_hup);

	// 在启动信号I/O前设置信号处理函数
	signal(SIGIO, sig_io);

	// 设置接收信号通知的进程，让本进程接收SIGIO信号
	fcntl(sockfd, F_SETOWN, getpid());

	// 为了能够在得到I/O事件后重复执行I/O操作，需要将文件描述符设置为非阻塞方式
	// O_ASYNC表示在文件描述符上使用信号驱动I/O
	int flags = fcntl(sockfd, F_GETFL);
	fcntl(sockfd, F_SETFL, flags | O_ASYNC | O_NONBLOCK);

	sigset_t zeromask, newmask, oldmask;
	sigemptyset(&zeromask);
	sigemptyset(&newmask);
	sigemptyset(&oldmask);

	// 设置新的信号掩码，阻塞SIGIO信号
	sigaddset(&newmask, SIGIO);
	sigprocmask(SIG_BLOCK, &newmask, &oldmask);
	for (; ;)
	{
		while (nqueue == 0)
		{
			// 挂起进程直到收到任何信号，该函数返回后SIGIO继续被阻塞
			sigsuspend(&zeromask);
		}
		// 解除SIGIO的阻塞
		sigprocmask(SIG_SETMASK, &oldmask, NULL);
		sendto(sockfd, dg[iget].dg_data, dg[iget].dg_len, 0, dg[iget].dg_sa, dg[iget].dg_salen);
		if (++iget >= QSIZE)
		{
			iget = 0;
		}
		// 为了能够修改nqueue的值，阻塞SIGIO信号
		sigprocmask(SIG_BLOCK, &newmask, &oldmask);
		nqueue--;
	}
}

int main(int argc, char *argv[])
{
	int sockfd = socket(AF_INET, SOCK_DGRAM, 0);
	struct sockaddr_in servaddr, cliaddr;
	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
	servaddr.sin_port = htons(SERV_PORT);
	if (bind(sockfd,  (struct sockaddr*)&servaddr, sizeof(servaddr)) < 0)
	{
		printf("bind error\n");
		exit(1);
	}

	dg_echo(sockfd, (struct sockaddr*)&cliaddr, sizeof(cliaddr));
	return 1;
}
```

# 相关下载

本文中的实例，代码采用eclipse CDT编写，可以直接导入eclipse中运行。

[下载实例](http://pan.baidu.com/s/1o6zCNOm)


