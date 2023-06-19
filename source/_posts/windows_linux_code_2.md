---
title: 再谈Windows和Linux之间的中文编码问题
Status: public
url: windows_linux_code_2
date: 2013-11-10
---

上次文章《[Windows和Linux之间的中文编码问题](http://kuring.me/post/windows_linux_code)》中提到的在Windows下的源代码程序放到Linux下出现中文编码问题，解决方法为通过iconv工具转换源代码文件的编码为UTF8格式。最近多学习了些字符编码的知识，发现了解决此问题的另外一种办法。

# 基础知识
我们在编译程序的时候会涉及到几个编码问题，包括C++源文件的编码、C++程序的内码和运行环境编码，其中C++程序的内码较难理解。

C++程序的内码是指在可执行文件中字符串常量是以什么编码形式存放的，其中字符串常量为窄字符形式。在Windows系统中C++的内码通常为GB18030，在Linux下的gcc/g++使用的内码默认为utf8，可以通过fexec-charset参数来修改。

运行环境编码即为操作系统的编码，通常情况下，简体中文Windows操作系统编码为GB18030，而Linux下默认为UTF8。

# gcc命令的参数
gcc有两个参数可以用来解决编码问题。
-finput-charset：用来指定源文件的编码。
-fexec-charset：用来指定生成的可执行文件的编码。

如果这两个参数均未指定，则GCC不会对编码进行转换。
以上这两个参数就可以用来在不修改源文件编码的基础上来达到正确的效果，达到和上篇文章中解决问题同样的效果。

# 关于Unicode编码
一直对Unicode编码比较糊涂，Unicode只是编码方法规范，而不是具体的存储方法。
常用的Unicode又分为UCS-2和UCS-两种编码，其中UCS-2采用固定的2个字节存储，UCS-4采用固定的4个字节存储。
通常情况下提到的Unicode编码即为UCS-2编码，比如Windows记事本中的保存为Unicode编码，其实就是保存为了UCS-2编码，由于每个字符均为2个字节，所以下次读取的时候仍然可以通过存储格式还原出来。

# 参考文章
[字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
[字符编码详解](http://www.crifan.com/files/doc/docbook/char_encoding/release/html/char_encoding.html)
[关于c++的一些编码问题](http://www.vip-tarena.com/C__peixun/876.html)
