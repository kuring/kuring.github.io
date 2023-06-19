---
title: 通过python来抓取和解析网页内容
Status: public
url: python_parse_web
tags: python
date: 2014-12-14
---

最近这段时间回顾了下python，距离上次使用python已经超过两年的时间了。

相对于c++语言，python要灵活许多，对于工作中的一些小问题的解决可以通过python来实现比较高效和方便，比如网页的抓取和解析。甚至对于非IT的工作，也可以通过脚本的方式来解决，只要是工作中遇到反复处理的体力活劳动就可以考虑利用编程方式来解决。

本文以我的博客的[文档列表页面](http://kuring.me/archive)为例，利用python对页面中的文章名进行提取。

文章列表页中的文章列表部分的url如下：

```html
<ul class="listing">
    <li class="listing-item"><span class="date">2014-12-03</span><a href="/post/linux_funtion_advance_feature"  title="Linux函数高级特性" >Linux函数高级特性</a>
    </li>
    <li class="listing-item"><span class="date">2014-12-02</span><a href="/post/cgdb"  title="cgdb的使用" >cgdb的使用</a>
    </li>
...
</ul>
```

# requests模块的安装

requests模块用于加载要请求的web页面。

在python的命令行中输入`import requests`，报错说明requests模块没有安装。

我这里打算采用easy_install的在线安装方式安装，发现系统中并不存在easy_install命令，输入`sudo apt-get install python-setuptools`来安装easy_install工具。

执行`sudo easy_install requests`安装requests模块。

# Beautiful Soup安装

为了能够对页面中的内容进行解析，本文使用Beautiful Soup。当然，本文的例子需求较简单，完全可以使用分析字符串的方式。

执行`sudo easy_install beautifulsoup4`即可安装。

# 编码问题

python的编码问题确实是一个很头大的问题，尤其是对于不熟悉python的菜鸟。

python自身的编码问题就已经够头大的了，碰巧requests模块也有一个编码问题的bug，具体的bug见参考文章。

# 代码

```python
#!/usr/bin/env python                                                                                                                                                           
# -*- coding: utf-8 -*-

' a http parse test programe '

__author__ = 'kuring lv'


import requests
import bs4

archives_url = "http://kuring.me/archive"

def start_parse(url) :
    print "开始获取(%s)内容" % url
    response = requests.get(url)
    print "获取网页内容完毕"
    
    soup = bs4.BeautifulSoup(response.content.decode("utf-8"))
    #soup = bs4.BeautifulSoup(response.text);
    
    # 为了防止漏掉调用close方法，这里使用了with语句
    # 写入到文件中的编码为utf-8
    with open('archives.txt', 'w') as f :
        for archive in soup.select("li.listing-item a") :
            f.write(archive.get_text().encode('utf-8') + "\n")
            print archive.get_text().encode('utf-8')

# 当命令行运行该模块时，__name__等于'__main__'
# 其他模块导入该模块时，__name__等于'parse_html'
if __name__ == '__main__' :
    start_parse(archives_url)
```

# 参考文章

* [廖雪峰的python教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)

* [Beautiful Soup 4.2.0 文档](http://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html)

* [使用 Python 轻松抓取网页](http://wuchong.me/blog/2014/04/24/easy-web-scraping-with-python/)

* [Python+Requests抓取中文乱码改进方案](http://www.au92.com/archives/python-requests-chinese-improve-random-code.html)
