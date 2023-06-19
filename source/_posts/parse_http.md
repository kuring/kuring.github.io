---
title: 两个通过http获取指定网页内容并解析的简单程序
Status: public
url: parse_http
date: 2013-09-02 19:03:43
---

近段时间写了两个通过http协议来获取指定网页的内容并将内容解析出来的程序。程序一可以解析出目前本博客的内容页面的内容、时间、访问次数参数，采用Qt类库实现；程序二可以解析出新浪博客页面的内容、时间等参数，采用Linux下的tcp相关API实现。均采用C++语言实现。

# 程序一
该程序采用Qt类库实现，其中Http协议的发送和接收采用Qt类库封装的类，网页内容的解析采用Qt封装的解析XML的相关类。
该程序仅能解析标准的Html语言，对于网页中的所有"<>"标签必须有结尾才行。例如本页面源码中的
```
<meta content="black" name="apple-mobile-web-app-status-bar-style" />
```
必须是闭合的。如果是下面这样则无法正确解析网页内容，这是由于采用的Qt类库决定的。
```
<meta content="black" name="apple-mobile-web-app-status-bar-style">
```

# 程序二
该程序的Http协议部分采用Linux的tcp协议api实现，解析网页直接采用搜索字符串的方式实现，较上一种方式要底层，仅能运行在Linux系统下运行。

# 相关下载
[程序一和二的下载链接](http://pan.baidu.com/share/link?shareid=1578018382&uk=3506813023)