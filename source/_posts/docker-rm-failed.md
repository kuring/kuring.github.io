---
title: kubernetes中pod无法删除的问题排查
date: 2019-01-30 20:51:02
tags:
---

## 现象

```
$ cat /etc/redhat-release
CentOS Linux release 7.2.1511 (Core)

$ uname -a
Linux c3-a05-136-45-10.com 3.10.0-327.el7.x86_64 #1 SMP Thu Nov 19 22:10:57 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux

$ docker info | grep "Storage Driver"
Storage Driver: devicemapper
```

在CentOS7.2的系统上，发现有一部分pod在delete后一直处于Terminating状态

```
$ kubectl get pods -o wide
NAME                                 READY     STATUS        RESTARTS   AGE       IP        NODE          NOMINATED NODE
httpserver-prod-1-6cb97dfbcc-25dsh   0/1       Terminating   0          55d       <none>    10.136.45.6   <none>
httpserver-prod-1-6cb97dfbcc-f9flb   0/1       Terminating   0          54d       <none>    10.136.45.4   <none>
httpserver-prod-1-6cb97dfbcc-m7sl4   0/1       Terminating   0          55d       <none>    10.136.45.6   <none>
httpserver-prod-1-6cb97dfbcc-pqpht   0/1       Terminating   0          55d       <none>    10.136.45.6   <none>
httpserver-prod-1-6cb97dfbcc-r987g   0/1       Terminating   0          55d       <none>    10.136.45.4   <none>
httpserver-prod-1-6cb97dfbcc-zghhr   0/1       Terminating   0          54d       <none>    10.136.45.6   <none>
```

查看docker的日志发现有如下报错信息如下，含义为在删除pod时由于/var/lib/docker/overlay/*/merged目录被其他应用占用，从而导致容器无法清除。

```
Jan 30 14:57:47 c3-a05-136-45-4.com dockerd[1510]: time="2019-01-30T14:57:47.704641914+08:00" level=error msg="Error removing mounted layer e6b7378c58a34cb42c6fa7924f7a52b7a19a64b2166d7a56f363e73ecba6e5a9: remove /var/lib/docker/overlay/98a56d695c9e3d0b6a9f3b5e0e60abf7cdb3ce73e976b00e36ca59028e585a36/merged: device or resource busy"
Jan 30 14:57:47 c3-a05-136-45-4.com dockerd[1510]: time="2019-01-30T14:57:47.704772288+08:00" level=error msg="Handler for DELETE /v1.31/containers/e6b7378c58a34cb42c6fa7924f7a52b7a19a64b2166d7a56f363e73ecba6e5a9 returned error: driver \"overlay\" failed to remove root filesystem for e6b7378c58a34cb42c6fa7924f7a52b7a19a64b2166d7a56f363e73ecba6e5a9: remove /var/lib/docker/overlay/98a56d695c9e3d0b6a9f3b5e0e60abf7cdb3ce73e976b00e36ca59028e585a36/merged: device or resource busy"
Jan 30 14:57:48 c3-a05-136-45-4.com dockerd[1510]: time="2019-01-30T14:57:48.228837657+08:00" level=error msg="Error removing mounted layer 2851b80d5c45d1cac3e7384116da0ad022af21701f9aa0d9ba3598efd5723030: remove /var/lib/docker/overlay/0ff0f98e1abf43c10711f2804cae3cf37efd597016d38b4753e2af19c2e27eb9/merged: device or resource busy"
Jan 30 14:57:48 c3-a05-136-45-4.com dockerd[1510]: time="2019-01-30T14:57:48.228953497+08:00" level=error msg="Handler for DELETE /v1.31/containers/2851b80d5c45d1cac3e7384116da0ad022af21701f9aa0d9ba3598efd5723030 returned error: driver \"overlay\" failed to remove root filesystem for 2851b80d5c45d1cac3e7384116da0ad022af21701f9aa0d9ba3598efd5723030: remove /var/lib/docker/overlay/0ff0f98e1abf43c10711f2804cae3cf37efd597016d38b4753e2af19c2e27eb9/merged: device or resource busy"
```

通过`docker ps -a`看到容器的状态为"Removal In Progress"。通过`docker inspect`可以看到容器的进程已经退出了。

```
# docker ps -a
CONTAINER ID        IMAGE                                                                  COMMAND                  CREATED             STATUS                    PORTS               NAMES
e6b7378c58a3        golang-httpserver        "/bin/sh -c 'go ru..."   7 weeks ago         Removal In Progress                           k8s_golang-httpserver_httpserver-prod-1-6cb97dfbcc-f9flb_default_9e3d2cbb-f9d4-11e8-b61c-f01fafd10a1b_0

# docker inspect e6b7378c58a3 --format '{{.State.Pid}}'
0
```

使用`docker rm`命令删除容器会报错

```
# docker rm e6b7378c58a3
Error response from daemon: driver "overlay" failed to remove root filesystem for e6b7378c58a34cb42c6fa7924f7a52b7a19a64b2166d7a56f363e73ecba6e5a9: remove /var/lib/docker/overlay/98a56d695c9e3d0b6a9f3b5e0e60abf7cdb3ce73e976b00e36ca59028e585a36/merged: device or resource busy
```

通过`kubectl delete pods`命令虽然可以强制删除pod，但在宿主机上仍然能看到容器的状态为"Removal In Progress"。

```
# kubectl delete pods  httpserver-prod-1-6cb97dfbcc-f9flb --grace-period=0 --force
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "httpserver-prod-1-6cb97dfbcc-f9flb" force deleted
```

通过搜索挂载目录的信息，可以找到是哪个进程挂载了该目录。可以看到是ntpd服务挂载了该目录。

```
# grep -nr 98a56 /proc/*/mountinfo
/proc/2725007/mountinfo:48:296 183 0:183 / /var/lib/docker/overlay/98a56d695c9e3d0b6a9f3b5e0e60abf7cdb3ce73e976b00e36ca59028e585a36/merged rw,relatime shared:88 - overlay overlay rw,lowerdir=/var/lib/docker/overlay/5e2a5f7af24e555a5afacd6a8faa406b42c51d7f2bb4cde22adcea22e0153583/root,upperdir=/var/lib/docker/overlay/98a56d695c9e3d0b6a9f3b5e0e60abf7cdb3ce73e976b00e36ca59028e585a36/upper,workdir=/var/lib/docker/overlay/98a56d695c9e3d0b6a9f3b5e0e60abf7cdb3ce73e976b00e36ca59028e585a36/work

# ps -ef | grep 2725007
ntp      2725007       1  0 Jan07 ?        00:00:02 /usr/sbin/ntpd -u ntp:ntp -g

# ntpd进程的启动时间在容器启动之后
# ps -ef | grep ntpd
root     1179644   18205  0 19:52 pts/1    00:00:00 grep --color=auto -d skip -i ntpd
ntp      3853149       1  0 Jan07 ?        00:00:02 /usr/sbin/ntpd -u ntp:ntp -g
```

查看ntpd.service文件内容如下，其中`PrivateTmp=true`，该选项用于控制服务是否使用单独的tmp目录：

```
[Unit]
Description=Network Time Service
After=syslog.target ntpdate.service sntp.service

[Service]
Type=forking
EnvironmentFile=-/etc/sysconfig/ntpd
ExecStart=/usr/sbin/ntpd -u ntp:ntp $OPTIONS
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

## 问题复现

```
# 在系统上启动一个容器，此时ntpd必须处于running状态
$ docker run -d httpserver:1 /bin/sh -c "while : ; do sleep 1000 ; done"

# 启动容器
$ docker run -d httpserver:1 /bin/sh -c "while : ; do sleep 1000 ; done"
200222b438aac43bbe32a6c54e31ced0848482b9dec3e519d2f847c70c1ce801

# 重启ntpd
$ systemctl restart ntpd

$ docker stop 200222b438aa

# 此时容器的相关信息还存在
$ docker ps -a
CONTAINER ID        IMAGE                                               COMMAND                  CREATED              STATUS                       PORTS               NAMES
200222b438aa        httpserver:1    "/bin/sh -c 'while..."   About a minute ago   Exited (137) 7 seconds ago                       hardcore_yalow

# 强制删除容器失败
$ docker rm -f 200222b438aa
Error response from daemon: driver "devicemapper" failed to remove root filesystem for 200222b438aac43bbe32a6c54e31ced0848482b9dec3e519d2f847c70c1ce801: remove /var/lib/docker/devicemapper/mnt/e53342aa9cf5f43e73b6596f88939b8d3fdefaf1ca03ee95a24d867e1de6c522: device or resource busy


# 此时容器处于Removal In Progress状态
$ docker ps -a
CONTAINER ID        IMAGE                                               COMMAND                  CREATED             STATUS                PORTS               NAMES
200222b438aa        httpserver:1    "/bin/sh -c 'while..."   2 minutes ago       Removal In Progress                       hardcore_yalow

# 再次重启ntpd进程
$ systemctl restart ntpd

# 强制删除成功
$ docker rm 200222b438aa
200222b438aa
```

经在如下版本的CentOS7系统实验，该问题不存在。

```
$ uname -a
Linux localhost.localdomain 3.10.0-862.9.1.el7.x86_64 #1 SMP Mon Jul 16 16:29:36 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

$ cat /etc/redhat-release
CentOS Linux release 7.5.1804 (Core)

$ docker info | grep "Storage Driver"
Storage Driver: overlay2
```

## 问题产生原因

此问题为Systemd启用PrivateTmp选项后，导致mount namespace的一处内核bug。

## 处理方式

在/usr/lib/systemd/system/docker.service的[Service]中增加`MountFlags=slave`，并重新启动docker服务，注意重启docker后，容器会重启。

当然也可以通过重启ntpd服务的方式来临时解决问题，但当下次删除容器时还需要重启ntpd。

还有一种办法是修改ntpd.service中的`PrivateTmp=true`，然后重启ntpd服务。

## ref

[Docker 故障（device or resource busy）](http://blog.kissingwolf.com/2017/09/09/Docker-%E6%95%85%E9%9A%9C%EF%BC%88device-or-resource-busy%EF%BC%89/)
