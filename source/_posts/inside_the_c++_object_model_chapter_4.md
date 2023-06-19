---
title: 深度探索C++对象模型读书笔记_第四章：函数语意学
Status: public
url: inside_the_c++_object_model_chapter_4
tags: C++
date: 2013-07-25
toc: yes
---

# 成员函数的各种调用方式

## 非静态成员函数的调用方式

在C++中必须要保证类的非静态成员函数必须和非成员函数的执行效率一致，在编译的过程中，编译期已经将类的非静态成员函数编译为了非成员函数。

```c++
class  Test
{
public:
	int sum() const
	{
		return a + b;
	}

	int add()
	{
		return a;		
	}
	
	int add(int size)
	{
	   return a + size;
	}
public:
	 int a;
	 int b;
};
```

在上述Test类中，编译器会在编译阶段对类中的成员函数做一些转换。下面列出了编译器可能会做出的变换，不同的编译器实现不太一致。

```c++
int sum_TestFv(const Test * const this)
{
    return this->a + this->b;
}

int add_TestFv(Test * const this)
{
	return this->a;
}

int add_Testi(int size, Test * const this)
{
	return this->a + size;
} 
```

通过上面的变换可以总结出如下规律：

* 将成员函数重新改写为一个外部函数，并且对函数的名字进行处理，使在程序中唯一。一种可能的处理办法就是将函数名更改为：函数名_类名_函数参数。这样即解决了类之间函数名相同的问题，又解决了类之间函数重载的问题。
* 在函数的参数末尾添加额外的this指针参数。对于const函数添加的this指针为双const类型，对于非const函数则添加的this指针为指向的内容可变的const指针。
* 在函数内对成员函数的存取采用this指针来实现。

## 虚成员函数

虚函数如果是通过指针类型访问，需要在运行时动态决定指针指向的类型，因此需要访问虚函数表才能够获取正确的虚函数地址。访问虚函数的方式为`(*ptr->vptr[i])(ptr)`，其中i代表要调用的虚函数在虚函数表中的索引，最后一个ptr代表要调用虚函数的编译器添加的this指针参数。

关于虚成员函数的更详细问题会在下一个节中进行讨论。

## 静态成员函数

静态成员函数中没有this指针，可以理解成带类作用域的全局函数，执行效率跟全局函数一致。

# 虚成员函数

这部分内容是本书的核心内容，可以参考陈皓的博客相关文章，已经对C++中的虚成员函数和虚成员变量进行了说明。

* [C++ 虚函数表解析](http://blog.csdn.net/haoel/article/details/1948051)
* [C++ 对象的内存布局(上)](http://blog.csdn.net/haoel/article/details/3081328)
* [C++ 对象的内存布局(下)](http://blog.csdn.net/haoel/article/details/3081385)

## 单一继承下的虚函数

```c++
class Point
{
public:
	virtual ~Point(){}
	virtual Point& mult(float) = 0;
	float x() const {return _x;}
	virtual float y() const {return 0;}
	virtual float z() const {return 0;}
protected:
	Point(float x=0.0) {_x = x;}
	float _x;
};

class Point2d : public Point
{
public:
	Point2d(float x=0.0, float y=0.0) : Point(x), _y(y) {}
	~Point2d(){}

	Point2d& mult(float){return *this;}
	float y() const {return _y;}
protected:
	float _y;
};

class Point3d : public Point2d
{
public:
	Point3d(float x=0.0, float y=0.0, float z=0.0) : Point2d(x, y), _z(z) {}
	~Point3d(){}

	Point3d & mult(float) {return *this;}
	float z() const {return _z;}
protected:
	float _z;
};
```
三个类对应的虚函数表会转化成下图

![Image Title](/ref/c++/c++_object_model/chapter4_1.PNG)

通过图中可以看出每个函数在虚函数表中的位置无论在基类还是在子类中位置总是固定的。图中的Point的实例应该是不存在的，因为类中含有纯虚函数mult。

要想调用ptr->z()就变得非常容易，可以在编译器就可以确定虚函数的调用。虽然ptr所指向的对象在编译器并不能确定，但是编译器可以将其转化成为(*ptr->vptr[4])(ptr)。因为z()函数总是在虚函数表中的第四个位置，唯一需要在执行期确定的就是ptr所指的对象的实际类型。

## 多重继承下的虚函数

避免重复造轮子，参考上面博文。

## 虚拟继承下的虚函数

避免重复造轮子，参考上面博文。

# 函数的效率

非成员函数、静态成员函数、非静态成员函数都被转换成为了完全相同的形式。
inline函数的执行效率最高。
虚函数的效率最低。

# 指向成员函数的指针

这里学习到一个新的语法，之前没有接触过。即指向类成员函数的指针及使用方法。
```
#include <iostream>
using namespace std;
class Point
{
public:
	virtual ~Point() {}
	float x() {return _x;}
public:
	Point(float x=0.0)
	{
		_x = x;
	}
	float _x;
};

int main()
{
	Point point(1.0);
	float (Point::*p)();   // 定义指向成员函数的指针
	p = &Point::x;        // 为指向成员函数的指针赋值
	cout << (point.*p)();  // 调用指向类成员函数的指针
	return 1;
}
```
如果成员函数的指针并不用于虚函数、多重继承、虚基类等情况，则成员函数的指针效率跟非成员函数指针的效率一致。

## 指向虚成员函数的指针

书中对于函数取地址的语法在gcc和vs2008下我试验不成功，语法错误。

## 多重继承下指向成员函数的指针

依赖于编译器的实现，用到的情况比较少，没仔细看。

## 指向成员函数指针的效率

在引入了虚函数、多重继承、虚基类等情况后，指向成员函数的指针效率有所下降。

# 内联函数

内联只是一个请求，编译器并不一定会将函数内联的展开。

## 形式参数

内联时每一个形参都会被对应的实参取代。

```c++
inline int min(int i, int j)
{
	return i < j ? i : j;
}

inline int bar()
{
	int minval;
	int val1 = 1024;
	int val2 = 2048;
	minval = min(val1, val2);
	minval = min(1024, 2048);
	minval = min(foo(), bar() + 1);
}
```

`minval=min(val1, val2)`会被内联展开成`minval = val1 < val2 ? val1 : val2`。
`minval = min(1024, 2048)`会被扩展为`minval = 1024`。
`minval = min(foo(), bar() + 1)`需要引入临时对象，被扩展为
```
int t1;
int t2;
minval = (t1 = foo()) , (t2 = bar() + 1), t1 < t2 ? t1 : t2;
```

## 局部变量

```c++
inline int min(int i, int j)
{
	int minval = i < j ? i : j;
	return minval;
}

inline int bar()
{
	int minval;
	int val1 = 1024;
	int val2 = 2048;
	minval = min(val1, val2);
}
```
在内联函数中引入局部变量，内联函数在内联的时候局部变量会拥有一个唯一的名称。代码中的`minval = min(val1, val2)`会被内联为：

```c++
int _min_lv_minval;
minval = (_min_lv_minval = val1 < val2 ? val1 : val2), _min_lv_minval;
```

内联函数可以代替C语言中的#define宏定义，但是当内联函数调用次数过多，会产生大量的扩展代码，使程序的大小变大。
