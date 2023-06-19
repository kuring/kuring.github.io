---
title: 深度探索C++对象模型读书笔记_第五章：构造、析构、复制语意学
Status: private
url: inside_the_c++_object_model_chapter_5
tags: C++
date: 2013-09-23
---

# 无继承情况下的对象构造
在《Unix环境高级编程》的7.6节中提到C程序的内存空间可以分为正文段、初始化数据段、非初始化数据段、栈、堆。其中初始化数据段包含程序中需明确赋初值的变量，如C语言中的全局变量`int maxcount = 99;`。非初始化数据段又称为bss（block started by symbol）段，在程序开始之前，内核将此段初始化为0或空指针，如出现在函数外面的`long sum[1000];`，该变量没有明确赋初值，因此放到了bss段中。
而在C++语言中，将所有的全局对象当做初始化过的数据来对待，因此不会将全局变量放到bss段中。

## POD数据类型
书中提到Plain ol' data，查了下应该叫Plain Old Data,简称POD，指C风格的struct结构体定义的数据结构，其中struct结构体中只能定义常规数据类型，不可以含有自定义数据类型。

```
typedef struct Point
{
	float x, y, z;
} Point;

Point global;

Point foobar()
{
	Point local;
	Point *heap = new Point();
	*heap = local;
	delete heap;
	return local;
}
```
首先看全局变量global，按照常规的理解，在程序启动的时候编译器会调用Point的合成的默认构造函数来初始化global变量，在程序退出时会调用Point的合成的析构函数来销毁global变量。实际上，C++编译器会将Point看成是一个POD对象，既不会调用合成的构造函数也不会调用合成的析构函数，但C++编译器会将global当成初始化过的数据来对待，不放入BSS段。

foobar函数中的local局部变量不会自动初始化，意味着local.x中的值是不可控的，但是local变量分配了栈空间。

`*heap = local;`执行时仅简单执行按字节复制操作，不会产生赋值操作符，因为Point是一个POD类型。

`return local;`同样仅通过字节复制操作产生一个临时对象。

## 抽象数据类型
这次将上面的Point类型从struct变换为class
```
class Point
{
public:
	Point(float x=0.0, float y=0.0, float z=0.0) : _x(x), _y(y), _z(z){}

private:
	float _x, _y, _z;
};
```
在上节中的foobar函数中，各个对象的默认复制构造函数、赋值操作符和析构函数仍然不会调用，因为调用是没有意义的，因此编译器干脆就不产生。
## 为继承做准备
再次更改Point类，引入虚函数。
```
class Point
{
public:
	Point(float x=0.0, float y=0.0) : _x(x), _y(y) {}
	virtual float z();
private:
	float _x, _y;
};
```
引入虚函数后，类对象就需要一个vtbl来存放虚函数的地址，类对象中需要添加vptr指针。而vptr的初始化是在对象构造的时候，因此对象初始化的时候需要调用构造函数，同时默认构造函数和赋值构造函数会自动在构造函数的最前面插入初始化vptr的代码。

# 继承体系下的对象构造
C++时会自动扩充类的每一个构造函数。扩充步骤如下：
1. 如果类含有虚基类，则所有虚基类的构造函数被调用，调用顺序为从左到右，从最深到最浅。
2. 如果类含有基类，则基类构造函数会被调用，以基类的声明顺序为顺序。
3. 如果类对象中含有vptr，必须在初始化类的成员变量之前为vptr指定初值，使其指向vtbl。
4. 将成员初始化列表中数据成员的初始化操作放入构造函数内部，并且按照成员在类中的声明顺序。
5. 如果类成员变量不在构造函数的初始化列表中，但是成员变量含有默认构造函数，则默认构造函数必须被调用。
 
## 虚拟继承
本小节将学习一下引入了虚继承机制之后构造函数的生成是什么样子的。
```
class Point
{
public:
	Point(float x=0.0, float y=0.0) : _x(x), _y(y) {}
	virtual float z();
private:
	float _x, _y;
};

class Point3d : public virtual Point
{
public:
	Point3d(float x=0.0, float y=0.0, float z=0.0)
		: Point(x, y), _z(z) {}
	~Point3d();

	virtual float z() {return _z;}
protected:
	float _z;
};

class Vertex : virtual public Point 
{
	// 不是重点忽略
};

class Vertex3d : public Point3d, public Vertex
{
	// 不是重点忽略
};

class PVertex : public Vertex3d
{
	// 不是重点忽略
};
```
类之间的继承关系如下图所示，已经属于最复杂的继承模型了。
![Image Title](/ref/c++/c++_object_model/chapter5_1.PNG)
如果要构造Vertex3d的实例，在内存中必须仅能有一个Point类型的对象，而如果在Point3d和Vertex基类中都构造一个Point实例显然是不合适的。答案是编译器会在Vertex3d的构造函数中生成Point的对象，在Point3d和Vertex的构造函数中均不会生成Point的对象。Vertex3d和Point3d的构造函数伪码如下面所示，Vertex构造函数的伪码和Point3d类似，这里就不再列出。
```
Point3d* Point3d::Point3d(Point3d *this, bool __most_derived, float x, float y, float z)
{
	// 如果子类初始化基类则本构造函数不需要初始化基类
	if (__most_derived != false)
	{
		this->Point::Point(x, y);
	}
	this->__vptr_Point3d = __vtbl_Point3d;	// 初始化指向本类的vptr
	this->__vptr_Point3d_Point = __vtbl_Point3d_Point;	// 初始化指向基类的vptr
	this->_z = z;
	return *this;
}

Vertex3d* Vertex3d::Vertex3d(Vertex3d *this, bool __most_derived, float x, float y, float z)
{
	if (__most_derived != false)
	{
		this->Point::Point(x, y);
	}
	this->Point3d::Point3d(false, x, y, z);
	this->Vertex::Vertex(false, x, y);
	// 初始化vptr
	// 用户代码
	return this;
}
```
编译器在类的构造函数中增加了一个bool变量来判断本类是否需要初始化基类，虚基类的初始化始终在继承最底层的类构造函数中初始化。对于PVertex类来说，Point类的构造函数在该类的构造函数中调用。

## vptr初始化语意学
vptr的在构造函数中的初始化时机为：在基类构造函数调用操作之后，在成员初始化列表和构造函数中显式代码之前。
构造函数的执行先后顺序为：
1. 所有虚基类、基类的构造函数会被调用。
2. 对象的vptr初始化，指向相关的vtbl。
3. 在构造函数内展开成员的初始化列表。
4. 执行显式代码。
# 对象复制语意学