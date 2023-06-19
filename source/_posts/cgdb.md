---
title: cgdb的使用
Status: public
url: cgdb
date: 2014-02-02
---

长期以来在使用gdb调试代码的时候都会因为调试代码的时候查看代码不方便而烦恼，gdb的list命令不够好用。而且网上的教程中也确实不容易发现可以替代gdb的好的终端下的调试工具，对于图形界面的集成开发环境（比如Eclipse CDT）和图形界面的调试工具（比如DDD）不在本文讨论的范围内，毕竟很多时候连接linux的时候还是终端方式的居多。

# tui模式

直到后来偶然间发现了gdb的`-tui`参数，该参数通过文本用户界面模式进行调试代码，使用起来确实方便了许多，再也不用边调试边list代码了，该模式已经满足了我的边调试边查看代码的需求。另外，gdbtui命令也可完成相同的功能。一个tui调试模式的界面如下：

![gdb -tui](http://beej.us/guide/bggdb/gdbwinss.png)

虽然，tui模式大大的提供了调试的友好性，但是仍然有一些缺点。比如显示的代码无法语法高亮，虽然会很影响用户体验，但是我毕竟是一名后台开发的程序员，这点可以忽略不计。源码布局和gdb命令布局之间切换不够方便，这点也不要紧，毕竟可以切换，只是需要输入两个单词就可以切换了。最最有问题的就是，该命令使用的时curses库，当用ssh通过windows下的SecureCRT或者putty连接进行调试时，源码布局部分往往不能够自动刷新，需要手工输入`CTRL+L`来刷新，具体的原理我没有去深究。

# cgdb

主要是本着解决gdb tui中的源码布局不能自动刷新的问题，本文的重点`cgdb`命令终于闪亮登场了。该命令不仅解决了源码布局自动刷新问题，同时也支持了语法高亮，同时源码查看支持vi的部分命令。功能基本跟vimgdb相近，但是安装更容易，在ubuntu下只需执行`sudo apt-get install cgdb`即可。

![cgdb](https://cgdb.github.io/images/screenshot_debugging.png)

下面是一些常用命令：

ESC：切换焦点到源码模式，在该界面中可以使用vi的常用命令

i：切换焦点到gdb模式

o：打开文件对话框，选择要显示的代码文件，按ESC取消

空格：在当前行设置一个断点

# 参考文章

[Beej's Quick Guide to GDB](http://beej.us/guide/bggdb/)
[cgdb](https://cgdb.github.io/)