title: 正向代理
date: 2022-03-12 20:42:50
tags:
author:
---
正向代理通常用在远程访问某个环境中的。常见的正向代理工具包括squid、nginx、3proxy。

## squid

老牌的正向代理工具。

安装：yum install squid && systemctl start squid

squid默认会监听在3128端口号。

缺点：如果修改了本地的/etc/hosts文件，则需要重启squid后才可以更新。

## 3proxy

官方并没有提供yum的安装方式，比较简单的运行方式是以docker的形式。

执行如下的命令，即可开启3128端口作为http代理，3129端口作为sock5代理。

```
mkdir /etc/3proxy
cat > /etc/3proxy/3proxy.cfg <<EOF
log /var/log/3proxy.log D
logformat "- +_L%t.%. %N.%p %E %U %C:%c %R:%r %O %I %h %T"
rotate 7
auth none
flush
allow somepu
maxconn 200

# starting HTTP proxy with disabled NTLM auth ( -n )
proxy -p3128 -n

# starting SOCKS proxy
socks -p3129 -n
EOF
docker run -d --restart=always -p 3128:3128 -p 3129:3129 --net=host -v /var/log:/var/log -v /etc/3proxy/3proxy.cfg:/etc/3proxy/3proxy.cfg --name 3proxy 3proxy/3proxy
```

## 设置代理

### 终端设置代理

shell支持如下的代理环境变量:
```
export http_proxy=http://localhost:1080
export https_proxy=http://localhost:1080
```

如果是 socks5 代理同样可以使用上述两个环境变量：
```
export http_proxy=socks5://localhost:1080
export https_proxy=socks5://localhost:1080
```

## 相关链接

- https://github.com/3proxy/3proxy