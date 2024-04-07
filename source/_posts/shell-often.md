title: 常用shell命令（持续更新）
date: 2022-03-28 11:20:03
tags:
author:
---
## curl

- --local-port：指定源端口号
- --proxy：指定本地代理，例如：http://127.0.0.1:52114
- -d：指定body，如果body比较小，可以直接指定`-d 'login=emma＆password=123'`，也可以通过指定文件的方式 `-d '@data.txt'`

## history

bash会将历史命令记录到文件.bash_history中，通过history命令可以查看到历史执行的命令。但history在默认情况下，仅会显示命令，不会展示出执行命令的时间。history命令可以根据环境变量HISTTIMEFORMAT来显示时间，要想显示时间可以执行如下的命令：

```
HISTTIMEFORMAT='%F %T ' history
```

## lrzsz

CentOS rpm包地址：https://rpmfind.net/linux/centos/7.9.2009/os/x86_64/Packages/lrzsz-0.12.20-36.el7.x86_64.rpm

## ps

最常用的为`ps -ef`和`ps aux`命令，两者的输出结果差不多，其中`ps -ef`为System V Style风格，`ps aux`为BSD风格，现在ps命令两者均支持。

```
$ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
www-data       1  0.0  0.0    212     8 ?        Ss   Apr09   0:00 /usr/bin/dumb-init -- /nginx-ingress-controller
www-data       6  0.9  0.3 813500 100456 ?       Ssl  Apr09  67:20 /nginx-ingress-controller --publish-service
www-data      33  0.0  0.7 458064 242252 ?       S    Apr09   0:53 nginx: master process /usr/local/nginx/sbin/nginx -c /etc/nginx/nginx.conf
www-data   46786  0.0  0.7 459976 239996 ?       S    17:57   0:00 rollback logs/eagleeye.log interval=60 adjust=600
www-data   46787  1.4  0.8 559120 283328 ?       Sl   17:57   2:07 nginx: worker process
www-data   46788  1.1  0.8 558992 284772 ?       Sl   17:57   1:38 nginx: worker process
www-data   46789  0.0  0.7 452012 237152 ?       S    17:57   0:01 nginx: cache manager process
www-data   46790  0.0  0.8 490832 267600 ?       S    17:57   0:00 nginx: x
www-data   47357  0.0  0.0  60052  1832 pts/2    R+   20:21   0:00 ps aux
```

每个列的值如下：
- %MEM：占用内存百分比
- VSZ: 进程使用的虚拟内存量（KB）
- RSS：进程占用的固定内存量，驻留在页中的（KB）
- STAT：进程的状态
- TIME：进程实际使用的cpu运行时间


## pssh

该工具的定位是在多台主机上批量执行pssh命令。

1. 将文件存放到文件中 /tmp/hosts 中，文件格式如下：
```
192.168.1.1
192.168.1.2
```
2. 批量执行shell命令：pssh -h /tmp/hosts -A -i 'uptime'。

具体参数说明如下：
- -A: 手工输入密码模式，如果未打通ssh免密，可以在执行pssh命令的时候手工输入主机密码，但要求所有主机密码必须保持一致

## rsync

常用命令：
1. `rsync -avz -e 'ssh -p 50023' ~/git/arkctl root@100.67.27.224:/tm`

常用参数：
- `--delete`: 本地文件删除时，同步删除远程文件
- `-e 'ssh -p 50023'`: 指定 ssh 端口号
- `--exclude=.git`: 忽略同步本地的 git 目录

## scp

- -P：指定端口号

## strace

跟踪进程的系统调用

- -p：指定进程
- -s：指定输出的字符串的最大大小
- -f：跟踪由fork调用产生的子进程

## wget

- -P: 当下载文件时，可以指定本地的下载的目录

## split

split [-bl] file [prefix]

- -b, --bytes=SIZE：对file进行切分，每个小文件大小为SIZE。可以指定单位b,k,m。

- -C,--bytes=SIZE：与-b选项类似，但是，切割时尽量维持每行的完整性。

- prefix：分割后产生的文件名前缀。

拆分文件：split -b 200m ip_config.gzip ip_config.gzip

文件合并：cat ip_config.gzip* > ip_config.gzip

## python

快速开启一个http server `python -m SimpleHTTPServer 8080`。

在python3环境下该命令变更为：`python3 -m http.server 8080`

格式化 json: `cat 1.json | python -m json.tool`

## awk

按照,打印出一每一列 `awk -F, '{for(i=1;i<=NF;i++){print $i;}}'`

## docker registry

- 列出镜像：`curl http://127.0.0.1:5000/v2/_catalog?n=1000`
- 查询镜像的tag: `curl http://127.0.0.1:5000/v2/nginx/tags/list`，如果遇到镜像名类似`aa/bb`的情况，需要转移一下 `curl http://127.0.0.1:5000/v2/aa\/bb/tags/list`

## socat

- 向本地的 socket 文件发送数据：`echo "test" | socat - unix-connect:/tmp/unix.sock`
- 通过交互的方式输入命令：`socat - UNIX-CONNECT:/var/lib/kubelet/pod-resources/kubelet.sock`

## git

- 删除远程分支 `git push origin --delete xxx`
- 强制更新远程分支：`git push --force-with-lease origin feature/statefulset`
- 删除本地分支：`git branch -D local-branch`
- 拉取远程分支并切换分支：`git checkout -b develop origin/remote-branch` develop为本地分支，origin/remote-branch为远程分支

## rpm

- `rpm -ivh xx.rpm`：用来安装一个 rpm 包
- `rpm -qa`：查看已经安装的包
- `rpm -ql`: 查看已经安装的 rpm 的文件内容
- `rpm -qpR *.rpm`: 查看rpm包的依赖
- `rpm -e *`：要删除的rpm包

## cloc

用来统计代码行数

- `cloc .`： 用来统计当前的代码行数
- `cloc . --exclude-dir vendor`：忽略目录 vendor，`--exclude-dir` 仅能支持一级目录
- `cloc . --fullpath --not-match-d=pkg/apis/`：用来忽略目录 pkg/apis 下的文件
