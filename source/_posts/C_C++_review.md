---
title: C/C++程序员面试宝典读书笔记
Status: private
url: C_C++_review
tags: C++
date: 2013-07-16 11:30:56
---

本文摘录《C/C++程序员面试宝典》一书中我认为需要注意的地方。

# 数组指针与指针数组的区别
数组指针即指向数组的指针，定义数组指针的代码如`int (*ap)[2]`，定义了一个指向包含两个元素的数组的数组指针。
如果数组的每一个元素都是一个指针，则该数组为指针数组。实例代码如下：
```
char *chararr[] = {"C", "C++", "Java"};
```
`int (*ap)[2]`和`int *ap[2]`的区别就是前一个是数组指针，后一个是指针数组。因为"[]"的优先级高于"*"，决定了这两个表达式的不同。

# public、protected、private
## 修饰成员变量或成员函数
* public：可以被该类中的函数、子类的函数、友元函数和该类的对象访问。
* protected：可以被该类中的函数、子类的函数和友元函数访问，不能被该类的对象访问。
* private：只能由该类中的函数或友元函数访问。
* 默认为private权限。
## 用在继承中
* public：基类成员保持自己的访问级别，基类的public成员为派生类的public成员，基类的protected成员为派生类的protected成员。
* protected：基类的public和protected成员在派生类中未protected成员。
* private：基类的所有成员在派生类中为private成员。

