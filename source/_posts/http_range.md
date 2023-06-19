---
title: 一个实例讲解HTTP的断点续传
Status: public
url: http_range
date: 2014-11-06
---

本文以从服务器下载一个文件为例，讲解HTTP的断点续传功能。

客户端IP地址为：192.168.1.2
服务器IP地址为：192.168.1.3

# 客户端向服务器发送请求

客户端向服务器发送的请求为：

```
GET /deepc.a HTTP/1.1
Host: 192.168.100.189
Connection: keep-alive
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.14 Safari/537.36
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
```

从中可以看出请求的文件名为deepc.a文件。

# 客户端向服务器发送具体请求

```
GET /deepc.a HTTP/1.1
Host: 192.168.100.189
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.14 Safari/537.36
Accept: */*
Referer: http://192.168.100.189/deepc.a
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Range: bytes=0-32767
```

Range字段表示请求文件的范围为0-32767。

# 服务器响应

第一次服务器的HTTP响应报文如下：

```
GET /deepc.a HTTP/1.1
Host: 192.168.1.3
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.14 Safari/537.36
Accept: */*
Referer: http://192.168.1.3/deepc.a
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Range: bytes=0-32767

HTTP/1.1 206 Partial Content
Date: Sun, 04 May 2014 05:14:54 GMT
Server: Apache/2.4.4 (Win32) PHP/5.4.16
Last-Modified: Sat, 03 May 2014 00:43:22 GMT
ETag: "7efce6-4f8742f6ed9b2"
Accept-Ranges: bytes
Content-Length: 32768
Content-Range: bytes 0-32767/8322278
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
```

HTTP的状态为206，表示服务器已经处理了部分HTTP相应。其中Content-Range字段表示服务器已经响应了0-32767个字节的文件内容。8322278表示文件的总长度为8322278字节。

# 客户端继续向服务器发送请求

客户端根据上次HTTP报文中服务器已经返回给的客户端的数据情况继续向服务器发送请求报文，向服务器发送的请求报文内容如下：

```
GET /deepc.a HTTP/1.1
Host: 192.168.100.189
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.14 Safari/537.36
Accept: */*
Referer: http://192.168.100.189/deepc.a
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Range: bytes=32768-8322277
If-Range: "7efce6-4f8742f6ed9b2"

HTTP/1.1 206 Partial Content
Date: Sun, 04 May 2014 05:14:54 GMT
Server: Apache/2.4.4 (Win32) PHP/5.4.16
Last-Modified: Sat, 03 May 2014 00:43:22 GMT
ETag: "7efce6-4f8742f6ed9b2"
Accept-Ranges: bytes
Content-Length: 8289510
Content-Range: bytes 32768-8322277/8322278
Keep-Alive: timeout=5, max=98
Connection: Keep-Alive
```

Content-Range的内容表示客户端向服务器请求文件中32768-8322277之间的字节数据。

# 第二次服务器的HTTP响应报文如下：

```
HTTP/1.1 206 Partial Content
Date: Sun, 04 May 2014 05:14:54 GMT
Server: Apache/2.4.4 (Win32) PHP/5.4.16
Last-Modified: Sat, 03 May 2014 00:43:22 GMT
ETag: "7efce6-4f8742f6ed9b2"
Accept-Ranges: bytes
Content-Length: 8289510
Content-Range: bytes 32768-8322277/8322278
Keep-Alive: timeout=5, max=98
Connection: Keep-Alive
```

表示服务器已经相应完成了32768-8322277之间的数据。