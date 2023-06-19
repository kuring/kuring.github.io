---
date: 2013-05-31 17:18:00
title: farbox与jekyll对比
---

前端时间折腾了一段时间的Github博客，终于搞明白了jekyll，开始写了一篇博客发现问题比较多。比如中文编码问题，Github对makedown的支持问题，Github的文章同步问题，网速问题。总体而言，感觉用Github来写博客还不是非常满意。

今天下午偶然想起了前段时间看过faxbox的一篇文章，今天下载下来尝试，果然非常酷。优点如下：

* 博客放在了Dropbox不会丢失。~~当然没有Github酷，Github可以通过版本管理来查看历史版本。~~貌似Dropbox本身有版本管理的功能。
* 写博客的方式简捷。Github的jekyll还需要利用rake命令来创建文章，这样才能保证文章的头部含有YAML标签。而faxbox直接编辑文本文件即可。
* 文本编辑器给力。可以通过实时预览功能查看makedown的解析情况，当然也可以通过在线的makedown编辑器[stackedit](http://benweet.github.io/stackedit/)来编写makedown。
* 网速比Github快。测试一下感觉网速还比较快。
* 可以修改模板，fork模板的方式简捷更酷。
* 对中文的支持好。因为本来就是本土化的软件当然对中文全力支持。
* 代码语法高亮。测试了一下默认的模板支持语法高亮。
* 自带的模板更漂亮。自带的模版虽然不多，但是有说明是适合博客还是相册。
* 网站自带网站分析工具。可以通过简单的浏览网页就可以知道网站的访问情况，虽然farbox在访问量达到1W之后要收费，按照目前的价格，但是我个人觉得比较值，希望farbox能够一直坚持做下去。做的好我愿意付费。

缺点：

* ~~经过一下午的了解，暂时没有发现博客可以分类的功能，不过这个功能我暂时可以不需要。以后准备先写些博客在Github和Faxbox上同步更新，以便多比较比较这两种Geek的写博客方式。~~后来发现是有文章分类功能的，具体操作为将文章放入不同的文件夹中，文件夹的名字即为分类的名字，这种方式比jekyll的YAML语法方式要好用。
* 感觉评论功能有点鸡肋。博客还没写完有事工作，准备续写时发现有人留言指正错误，想回复谢谢，发现无法直接回复。用多说代替之，方法为直接将多说的js脚本存入新建的comment_js.md文件中即可，非常简洁。
* Linux下的用户不太方便。由于farbox软件仅有windows版和mac版，dropbox在linux有相应版本，在linux下的用户不能享受到farbox编辑工具带来的方便。相反在linux下结合git和vim来编写博客却比较方便。

PS：
Github博客地址：[kuring.github.com](kuring.github.com)
Fixbox博客地址：[kuring.faxbox.com](kuring.github.com)