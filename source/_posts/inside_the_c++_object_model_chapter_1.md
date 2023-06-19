---
title: 深度探索C++对象模型读书笔记_第一章：关于对象
Status: public
url: inside_the_c++_object_model_chapter_1
tags: C++
date: 2013-07-20
toc: yes
---

C++对象模型是深入了解C++必须掌握知识，而《深度探索C++对象模型》一书基本是理解C++对象模型的必须之作。可惜本书看起来更像是作者Stanley B.Lippman的随笔，语言诙谐，跟作者的另外一本经典之作《C++ Primer》有着天壤之别，侯捷的翻译也是晦涩难懂，跟侯捷翻译的其他作品也有一定差距（这两位大师还真是凑到一块了），所以这本书看起来还是很吃力的。这里挑选文中重点记录笔记，忽略扯淡部分，以备忘。

# C++的额外开销

C++相比C语言多出了封装、继承和多态等特性，新特性的增加必然会以牺牲一部分性能为代价。额外开销主要是由virtual引起的，包括：

* 虚函数机制：需要在运行期间动态绑定。
* 虚基类：多次出现在继承中的基类有一个单一且被共享的实体。

# C++对象模型

C++类成员变量包括：静态成员变量和非静态成员变量。成员函数包括：静态成员函数、非静态成员函数和虚函数。

## 单一类的对象模型

```c++
class Point {
    public:
        Point(float xval);
        virtual ~Point();
        float x() const;            /* 非静态成员函数 */                                                                                                                         
        static int PointCount();    /* 静态成员函数 */  

    protected:
        virtual ostream& print(ostream &os) const;  /* 虚函数 */  
        float _x;                   /* 非静态成员变量 */  
        static int _point_count;    /* 静态成员函数 */  
}
```
该类的c++对象模型如下图：
![Image Title](/ref/c++/c++_object_model/chapter1_1.PNG)

通过图中可以看出：

* 非静态数据成员直接放到了类的对象里面。
* 静态数据成员放到所有的类对象的外面，即静态存储区。
* 静态和非静态的成员函数放在类对象之外，即代码区。
* 如果类中存在虚函数，则会产生一个虚函数表（vtbl），表中的表项存储了指向虚函数的指针。在类对象的开始位置添加一个指向虚函数表的指针（vptr）。vptr的赋值由类的构造函数和赋值运算符自动完成。
* 虚函数表的第一项指向用来作为动态类型识别用的type_info对象。

# C++支持的编程范式(programming paradigms)

* 程序模型：通俗的理解成C语言的面向过程编程方式。
* 抽象数据类型模型：通过类封装成为了一种新的数据类型，该数据类型有别于基本数据类型。
* 面向对象模型：利用封装、继承和多态的特性。

# C++支持多态的方式

* 隐含的转换操作，例如通过父类的指针指向子类的对象。`shape *ps = new circle();`
* 通过虚函数机制。
* 通过dynamic_cast强制类型转换。如`if (circle *pc = dynamic_cast<circle*>(ps))`。

# 类对象的内存构成

* 非静态数据成员。
* 由于内存对齐而添加到非静态数据成员中的空白。
* 为了支持虚机制（包括：虚函数和虚继承）而额外占用的内存。

# 利用工具查看对象模型

查看C++类的对象模型有两种比较简便的方式，一种是使用Virtual Studio在调试模式下查看对象的组成，但是往往显示的对象不全;另外一种是通过Virtual Studio中cl命令来静态查看。

本文选择使用cl工具的方式，cl命令位于C:\Program Files (x86)\Microsoft Visual Studio 9.0\VC\bin目录下，为了方便使用可以将该变量添加到Path环境变量中。在命令行中执行cl命令，提示“计算机中丢失mspdb80.dll”，该文件位于C:\Program Files (x86)\Microsoft Visual Studio 9.0\Common7\IDE目录下，将msobj80.dll,mspdb80.dll,mspdbcore.dll,mspdbsrv.exe四个文件复制到C:\Program Files (x86)\Microsoft Visual Studio 9.0\VC\bin目录下即可。

通过`cl xxx.cpp /d1reportSingleClassLayoutXX`命令即可查看文件中类的对象模型，其中该命令最后的XX需要更换为要查看的类名，中间没有空格。

执行上述命令时提示：无法打开文件“LIBCMT.lib”。该文件位于C:\Program Files (x86)\Microsoft Visual Studio 9.0\VC\lib目录下，将该目录添加到环境变量lib中。重新打开命令行执行cl提示：无法打开文件“kernel32.lib”，将C:\Program Files\Microsoft SDKs\Windows\v6.0A\Lib添加lib环境变量中。

