---
title: 大规模C++程序设计第1部分读书笔记
Status: public
url: large_scale_c++_software_desigin_part_1
date: 2014-04-14
---

# 声明和定义

声明将一个名字引入到了程序中，；定义提供了一个实体（如类型、实例、函数）在程序中的唯一描述，为变量分配存储空间；

在一个给定的作用于中重复一个给定的声明是合法的，而重复定义是非法的。声明仅对当前编译单元有效，仅会在.o文件中加入了一个未定义符号。特定的类型不必与实际的定义类型匹配。

# 内部链接和外部链接

内部链接和外部链接是根据链接过程中一个符号是在编译单元内部还是外部进行划分的，编译单元是按照一个.c或.cpp文件的作用域进行划分的。

内部链接：一个标识符对于编译单元（一个目标文件）来说是局部的，并在链接时与其他编译单元中的标识符不冲突。

内部链接包括：

* static类型的变量、函数
* const类型的变量
* 枚举类型的定义
* typedef定义的类型
* class、struct、union的定义
* inline函数

外部链接：在多文件程序中，链接时这个标识符可以和其他编译单元交互。这些外部符号在程序中必须是唯一的，用来被其他编译单元中未定义的符号访问。

将带有外部链接的定义放在头文件中是错误的。

外部链接包括：

* 类的非内联成员函数
* 非内联且非static函数
* 类的static数据成员

# const类型的变量为内部链接的实例

test1.h内容如下：

```c++
void print1();
```

test1.cpp内容如下：

```c++
#include <stdio.h>

const int max_length = 256;

void print1()
{
	printf("max_length=%d\n", max_length);
}
```
test2.h内容如下：

```c++
void print2();
```

test2.cpp内容如下：

```c++
#include <stdio.h>

const int max_length = 128;

void print2()
{
	printf("max_length=%d\n", max_length);
}
```

main.cpp内容如下：

```c++
#include <stdio.h>

#include "test1.h"
#include "test2.h"

int main()
{
	print1();
	print2();
	return 1;
}
```

从实例可以看出在全局作用域内声明的const类型的变量为内部链接的，在多个文件的作用域中分别定义相同的const类型的变量并不会产生冲突。

# 基本规则

直接访问类的数据成员违反了面向对象中的封装原则，应该将类的数据成员私有化，并提供接口供部访问，类似于java bean规范。

避免使用全局变量，应该将全局变量封装到类中，并提供接口供外部访问。

为避免命名冲突，将全局函数封装到类中。

避免在头文件的作用域中使用enum、typedef和const数据，应该将这些数据封装到头文件的类作用域中。

在头文件中只应该包含如下内容：类、结构体、联合体和运算符函数的声明，类、结构体、联合体和内联函数的定义。
