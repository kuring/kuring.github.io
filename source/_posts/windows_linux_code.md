---
title: Windows和Linux之间的中文编码问题
Status: public
url: windows_linux_code
date: 2013-09-30 21:50:16
---

在开发Linux程序的时候通常会在Windows下编码，然后拿到Linux下编译调试。而两个操作系统之间的默认编码往往有差别。

# 文件编码问题
在Windows下查看文件编码可以使用记事本打开文件，然后点击“另存为”在右下角即可看到当前文件的编码方式。如果显示为ANSI编码，在简体中文系统下，ANSI 编码代表 GB2312 编码。不同 ANSI 编码之间互不兼容，ANSI是American National Standards Institute的缩写， 记事本默认是以ANSI编码保存文本文档的。

在Linux可以通过vi命令查看文件编码，用vi打开文件，然后输入`:set encoding`即可显示文件编码。

在VS2008中创建文件的默认编码是根据当前系统的编码格式确定的。VS2008编译器可以同时支持GB2312和UTF-8两种编码。

为了解决在Linux下的乱码问题，Linux下的编码格式为utf8编码，这里采用在Windows下将gb2312编码更改为utf8的方式来解决。iconv是一个可以转换文件编码的工具，编写一个批处理脚本来实现批量转换文件编码的功能。批处理文件的内容如下：
```
@ECHO OFF
FOR /R %%F IN (*.h,*.cpp) DO (
echo %%~nxF
iconv.exe -f GB2312 -t UTF-8 %%F > %%F.utf8
move %%F.utf8 %%F >nul
)
PAUSE
```
本脚本来自网络，不是我自己写的。
注意：在使用文件编码之前一定要备份文件，防止意外发生，否则后果自负。

# 文件名编码问题
Windows的中文系统下文件名的编码默认为gbk，在Linux默认编码为UTF-8。如果将Windows下的中文文件名的文件复制到Linux下肯定会出现乱码的问题。可以利用convmv工具来解决编码的问题。

具体执行操作为：在Linux系统下的要转换编码的目录下执行命令：`convmv -f GBk -t UTF-8 --notest -r *`，这样就会将该文件夹下的所有文件递归的转换编码为UTF-8。

convmv的帮助文档点[这里](http://www.j3e.de/linux/convmv/man/)。

# 相关下载
[脚本和iconv程序下载链接](http://pan.baidu.com/s/19UbCG)