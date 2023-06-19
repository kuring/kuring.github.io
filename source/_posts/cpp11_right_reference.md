---
title: C++11中的右值引用
Status: public
url: cpp11_right_reference
tags: C++
date: 2015-05-18
toc: yes
---

在C++98中有左值和右值的概念，不过这两个概念对于很多程序员并不关心，因为不知道这两个概念照样可以写出好程序。在C++11中对右值的概念进行了增强，我个人理解这部分内容是C++11引入的特性中最难以理解的了。该特性的引入至少可以解决C++98中的移动语义和完美转发问题，若你还不清楚这两个问题是什么，请向下看。

温馨提示，由于内容比较难懂，请仔细看。C++已经够复杂了，C++11中引入的新特性令C++更加复杂了。在学习本文的时候一定要理解清楚左值、右值、左值引用和右值引用。

# 移动构造函数

首先看一个C++98中的关于函数返回类对象的例子。

```c++
class MyString {
public: 
    MyString() { 
        _data = nullptr; 
        _len = 0; 
        printf("Constructor is called!\n");
    } 

    MyString(const char* p) { 
        _len = strlen (p); 
        _init_data(p); 
        cout << "Constructor is called! this->_data: " << (long)_data << endl;
    } 

    MyString(const MyString& str) { 
        _len = str._len; 
        _init_data(str._data); 
        cout << "Copy Constructor is called! src: " << (long)str._data << " dst: " << (long)_data << endl;
    }
    
    ~MyString() { 
        if (_data)
        {
            cout << "DeConstructor is called! this->_data: " << (long)_data << endl; 
            free(_data);
        }
        else
        {
            std::cout << "DeConstructor is called!" << std::endl; 
        }
    } 

    MyString& operator=(const MyString& str) { 
        if (this != &str) { 
            _len = str._len; 
            _init_data(str._data); 
        } 
        cout << "Copy Assignment is called! src: " << (long)str._data << " dst" << (long)_data << endl; 
        return *this; 
    } 
    
    operator const char *() const {
        return _data;
    }

private: 
    char *_data; 
    size_t   _len; 

    void _init_data(const char *s) { 
        _data = new char[_len+1]; 
        memcpy(_data, s, _len); 
        _data[_len] = '\0'; 
    } 
}; 

MyString foo()
{
    MyString middle("123");
    return middle;
}

int main() { 
    MyString a = foo(); 
    return 1;
}
```

该例子在编译器没有进行优化的情况下会输出以下内容，我在输出的内容中做了注释处理，如果连这个例子的输出都看不懂，建议再看一下C++的语法了。我这里使用的编译器命令为`g++ test.cpp -o main -g -fno-elide-constructors`，之所以要加上`-fno-elide-constructors`选项时因为g++编译器默认情况下会对函数返回类对象的情况作*返回值优化*处理，这不是我们讨论的重点。


```c++
Constructor is called! this->_data: 29483024 // middle对象的构造函数
Copy Constructor is called! src: 29483024 dst: 29483056 // 临时对象的构造，通过middle对象调用复制构造函数
DeConstructor is called! this->_data: 29483024 // middle对象的析构
Copy Constructor is called! src: 29483056 dst: 29483024	// a对象构造，通过临时对象调用复制构造函数
DeConstructor is called! this->_data: 29483056 // 临时对象析构
DeConstructor is called! this->_data: 29483024 // a对象析构
```

在上述例子中，临时对象的构造、复制和析构操作所带来的效率影响一直是C++中为人诟病的问题，临时对象的构造和析构操作均对堆上的内存进行操作，而如果_data的内存过大，势必会非常影响效率。从程序员的角度而言，该临时对象是透明的。而这一问题正是C++11中需要解决的问题。

在C++11中解决该问题的思路为，引入了移动构造函数，移动构造函数的定义如下。

```c++
MyString(MyString &&str) {
    cout << "Move Constructor is called! src: " << (long)str._data << endl;
    _len = str._len;
    _data = str._data;
    str._data = nullptr;
}
```
在移动构造函数中我们窃取了str对象已经申请的内存，将其拿为己用，并将str申请的内存给赋值为nullptr。移动构造函数和复制构造函数的不同之处在于移动构造函数的参数使用*&&*，这就是下文要讲解的右值引用符号。参数不再是const，因为在移动构造函数需要修改右值str的内容。

移动构造函数的调用时机为用来构造临时变量和用临时变量来构造对象的时候移动语义会被调用。可以通过下面的输出结果看到，我们所使用的编译参数为`g++ test.cpp -o main -g -fno-elide-constructors --std=c++11`。

```c++
Constructor is called! this->_data: 22872080 // middle对象构造
Move Constructor is called! src: 22872080 // 临时对象通过移动构造函数构造，将middle申请的内存窃取
DeConstructor is called! // middle对象析构
Move Constructor is called! src: 22872080 // 对象a通过移动构造函数构造，将临时对象的内存窃取
DeConstructor is called! // 临时对象析构
DeConstructor is called! this->_data: 22872080 // 对象a析构
```

通过输出结果可以看出，整个过程中仅申请了一块内存，这也正好符合我们的要求了。

# C++98中的左值和右值

我们先来看下C++98中的左值和右值的概念。左值和右值最直观的理解就是一条语句等号左边的为左值，等号右边的为右值，而事实上该种理解是错误的。左值：可以取地址，有名字的值，是一个指向某内存空间的表达式，可以使用&操作符获取内存地址。右值：不能取地址，即非左值的都是右值，没有名字的值，是一个临时值，表达式结束后右值就没有意义了。我想通过下面的例子，读者可以清楚的理解左值和右值了。

```c++
// lvalues:
//
int i = 42;
i = 43; // i是左值
int* p = &i; // i是左值
int& foo();
foo() = 42; // foo()返回引用类型是左值
int* p1 = &foo(); // foo()可以取地址是左值

// rvalues:
//
int foobar();
int j = 0;
j = foobar(); // foobar()是右值
int* p2 = &foobar(); // 编译错误，foobar()是右值不能取地址
j = 42; // 42是右值
```

# C++11右值引用和移动语义

在C++98中有引用的概念，对于`const int &m = 1`，其中m为引用类型，可以对其取地址，故为左值。在C++11中，引入了右值引用的概念，使用*&&*来表示。在引入了右值引用后，在函数重载时可以根据是左值引用还是右值引用来区分。

```c++
void fun(MyString &str)
{
    cout << "left reference" << endl;
}

void fun(MyString &&str)
{
    cout << "right reference" << endl;
}

int main() { 
    MyString a("456"); 
    fun(a); // 左值引用，调用void fun(MyString &str)
    fun(foo()); // 右值引用，调用void fun(MyString &&str)
    return 1;
}
```

在绝大多数情况下，这种通过左值引用和右值引用重载函数的方式仅会在类的构造函数和赋值操作符中出现，被例子仅是为了方便采用函数的形式，该种形式的函数用到的比较少。上述代码中所使用的将资源从一个对象到另外一个对象之间的转移就是移动语义。这里提到的资源是指类中的在堆上申请的内存、文件描述符等资源。

前面已经介绍过了移动构造函数的具体形式和使用情况，这里对移动赋值操作符的定义再说明一下，并将main函数的内容也一起更改，将得到如下输出结果。

```c++
MyString& operator=(MyString&& str) { 
    cout << "Move Operator= is called! src: " << (long)str._data << endl; 
    if (this != &str) { 
        if (_data != nullptr)
        {
            free(_data);
        }
        _len = str._len;
        _data = str._data;
        str._len = 0;
        str._data = nullptr;
    }     
    return *this; 
}

int main() { 
    MyString b;
    b = foo();
    return 1;
}

// 输出结果，整个过程仅申请了一个内存地址
Constructor is called! // 对象b构造函数调用
Constructor is called! this->_data: 14835728 // middle对象构造
Move Constructor is called! src: 14835728 // 临时对象通过移动构造函数由middle对象构造
DeConstructor is called! // middle对象析构
Move Operator= is called! src: 14835728 // 对象b通过移动赋值操作符由临时对象赋值
DeConstructor is called! // 临时对象析构
DeConstructor is called! this->_data: 14835728 // 对象b析构函数调用
```

在C++中对一个变量可以通过const来修饰，而const和引用是对变量约束的两种方式，为并行存在，相互独立。因此，就可以划分为了const左值引用、非const左值引用、const右值引用和非const右值引用四种类型。其中左值引用的绑定规则和C++98中是一致的。

非const左值引用只能绑定到非const左值，不能绑定到const右值、非const右值和const左值。这一点可以通过const关键字的语义来判断。

const左值引用可以绑定到任何类型，包括const左值、非const左值、const右值和非const右值，属于万能引用类型。其中绑定const右值的规则比较少见，但是语法上是可行的，比如`const int &a = 1`，只是我们一般都会直接使用`int &a = 1`了。

非const右值引用不能绑定到任何左值和const右值，只能绑定非const右值。

const右值引用类型仅是为了语法的完整性而设计的， 比如可以使用`const MyString &&right_ref = foo()`，但是右值引用类型的引入主要是为了移动语义，而移动语义需要右值引用是可以被修改的，因此const右值引用类型没有实际意义。

我们通过表格的形式对上文中提到的四种引用类型可以绑定的类型进行总结。

| 引用类型/是否绑定 | 非const左值 | const左值 | 非const右值 | const右值 | 备注|
| ---------------- | -------- | -------- |  -------- | -------- | -------- | 
非const左值引用     | 是 | 否 | 否 | 否 |无 |
const左值引用       | 是 | 是 | 是 | 是 | 全能绑定类型，绑定到const右值的情况比较少见 |
非const右值引用     | 否 | 否 | 是 | 否 | C++11中引入的特性，用于移动语义和完美转发 |
const值引用         | 是 | 否 | 否 | 否 | 没有实际意义，为了语法完整性而存在 |

下面针对上述例子，我们看一下foo函数绑定参数的情况。

如果只实现了`void foo(MyString &str)`，而没有实现`void fun(MyString &&str)`，则和之前一样foo函数的实参只能是非const左值。

如果只实现了`void foo(const MyString &str)`，而没有实现`void fun(MyString &&str)`，则和之前一样foo函数的参数即可以是左值又可以是右值，因为const左值引用是万能绑定类型。

如果只实现了`void foo(MyString &&str)`，而没有实现`void fun(MyString &str)`，则foo函数的参数只能是非const右值。

# 强制移动语义std::move()

前文中我们通过右值引用给类增加移动构造函数和移动赋值操作符已经解决了函数返回类对象效率低下的问题。那么还有什么问题没有解决呢？

在C++98中的swap函数的实现形式如下，在该函数中我们可以看到整个函数中的变量a、b、c均为左值，无法直接使用前面移动语义。

```c++
template <class T> 
void swap ( T& a, T& b )
{
    T c(a); 
    a=b;
    b=c;
}
```

但是如果该函数中能够使用移动语义是非常合适的，仅是为了交换两个变量，却要反复申请和释放资源。按照前面的知识变量c不可能为非const右值引用，因为变量a为非const左值，非const右值引用不能绑定到任何左值。

在C++11的标准库中引入了std::move()函数来解决该问题，该函数的作用为将其参数转换为右值。在C++11中的swap函数就可以更改为了：

```c++
template <class T> 
void swap (T& a, T& b)
{
    T c(std::move(a)); 
    a=std::move(b); 
    b=std::move(c);
}
```

在使用了move语义以后,swap函数的效率会大大提升，我们更改main函数后测试如下:

```c++
int main() { 
    // move函数
    MyString d("123");
    MyString e("456");
    swap(d, e);
    return 1;
}

// 输出结果，通过输出结果可以看出对象交换是成功的
Constructor is called! this->_data: 38469648 // 对象d构造
Constructor is called! this->_data: 38469680 // 对象e构造
Move Constructor is called! src: 38469648 // swap函数中的对象c通过移动构造函数构造
Move Operator= is called! src: 38469680 // swap函数中的对象a通过移动赋值操作符赋值
Move Operator= is called! src: 38469648 // swap函数中的对象b通过移动赋值操作符赋值
DeConstructor is called! // swap函数中的对象c析构
DeConstructor is called! this->_data: 38469648 // 对象e析构
DeConstructor is called! this->_data: 38469680 // 对象d析构
```

# 右值引用和右值的关系

这个问题就有点绕了，需要开动思考一下右值引用和右值是啥含义了。读者会凭空的认为右值引用肯定是右值，其实不然。我们在之前的例子中添加如下代码，并将main函数进行修改如下：

```c++
void test_rvalue_rref(MyString &&str)
{
    cout << "tmp object construct start" << endl;
    MyString tmp = str;
    cout << "tmp object construct finish" << endl;
}

int main() {
    test_rvalue_rref(foo());
    return 1;
}

// 输出结果
Constructor is called! this->_data: 28913680
Move Constructor is called! src: 28913680
DeConstructor is called!
tmp object construct start
Copy Constructor is called! src: 28913680 dst: 28913712 // 可以看到这里调用的是复制构造函数而不是移动构造函数
tmp object construct finish
DeConstructor is called! this->_data: 28913712
DeConstructor is called! this->_data: 28913680
```

我想程序运行的结果肯定跟大多数人想到的不一样，“Are you kidding me?不是应该调用移动构造函数吗？为什么调用了复制构造函数？”。关于右值引用和左右值之间的规则是：

> 如果右值引用有名字则为左值，如果右值引用没有名字则为右值。

通过规则我们可以发现，在我们的例子中右值引用str是有名字的，因此为左值，tmp的构造会调用复制构造函数。之所以会这样，是因为如果tmp构造的时候调用了移动构造函数，则调用完成后str的申请的内存自己已经不可用了，如果在该函数中该语句的后面在调用str变量会出现我们意想不到的问题。鉴于此，我们也就能够理解为什么有名字的右值引用是左值了。如果已经确定在tmp构造语句的后面不需要使用str变量了，可以使用std::move()函数将str变量从左值转换为右值，这样tmp变量的构造就可以使用移动构造函数了。

而如果我们调用的是`MyString b = foo()`语句，由于foo()函数返回的是临时对象没有名字属于右值，因此b的构造会调用移动构造函数。

该规则非常的重要，要想能够正确使用右值引用，该规则必须要掌握，否则写出来的代码会有一个大坑。

# 完美转发

前面已经介绍了本文的两大主题之一的移动语义，还剩下完美转发机制。完美转发机制通常用于库函数中，至少在我的工作中还是很少使用的。如果实在不想理解该问题，可以不用向下看了。在泛型编程中，经常会遇到的一个问题是怎样将一组参数原封不动的转发给另外一个函数。这里的原封不动是指，如果函数是左值，那么转发给的那个函数也要接收一个左值；如果参数是右值，那么转发给的函数也要接收一个右值；如果参数是const的，转发给的函数也要接收一个const参数；如果参数是非const的，转发给的函数也要接收一个非const值。

该问题看上去非常简单，其实不然。看一个例子：

```c++
#include <iostream>

using namespace std;

void fun(int &) { cout << "lvalue ref" << endl; } 
void fun(int &&) { cout << "rvalue ref" << endl; } 
void fun(const int &) { cout << "const lvalue ref" << endl; } 
void fun(const int &&) { cout << "const rvalue ref" << endl; }

template<typename T>
void PerfectForward(T t) { fun(t); } 

int main()
{
    PerfectForward(10);           // rvalue ref

    int a;
    PerfectForward(a);            // lvalue ref
    PerfectForward(std::move(a)); // rvalue ref

    const int b = 8;
    PerfectForward(b);            // const lvalue ref
    PerfectForward(std::move(b)); // const rvalue ref

    return 0;
}
```

在上述例子中，我们想达到的目的是PerfectForward模板函数能够完美转发参数t到fun函数中。上述例子中的PerfectForward函数必然不能够达到此目的，因为PerfectForward函数的参数为左值类型，调用的fun函数也必然为`void fun(int &)`。且调用PerfectForward之前就产生了一次参数的复制操作，因此这样的转发只能称之为正确转发，而不是完美转发。要想达到完美转发，需要做到像转发函数不存在一样的效率。

因此，我们考虑将PerfectForward函数的参数更改为引用类型，因为引用类型不会有额外的开销。另外，还需要考虑转发函数PerfectForward是否可以接收引用类型。如果转发函数PerfectForward仅能接收左值引用或右值引用的一种，那么也无法实现完美转发。

我们考虑使用`const T &t`类型的参数，因为我们在前文中提到过，const左值引用类型可以绑定到任何类型。但是这样目标函数就不一定能接收const左值引用类型的参数了。const左值引用属于左值，非const左值引用和非const右值引用是无法绑定到const左值的。

如果将参数t更改为非const右值引用、const右值也是不可以实现完美转发的。

在C++11中为了能够解决完美转发问题，引入了更为复杂的规则：引用折叠规则和特殊模板参数推导规则。

## 引用折叠推导规则

为了能够理解清楚引用折叠规则，还是通过以下例子来学习。

```c++
typedef int& TR;

int main()
{
    int a = 1;
    int &b = a;
    int & &c = a;  // 编译器报错，不可以对引用再显示添加引用
    TR &d = a;     // 通过typedef定义的类型隐式添加引用是可以的
    return 1;
}
```

在C++中，不可以在程序中对引用再显示添加引用类型，对于`int & &c`的声明变量方式，编译器会提示错误。但是如果在上下文中（包括使用模板实例化、typedef、auto类型推断等）出现了对引用类型再添加引用的情况，编译器是可以编译通过的。具体的引用折叠规则如下，可以看出一旦引用中定义了左值类型，折叠规则总是将其折叠为左值引用。这就是引用折叠规则的全部内容了。另外折叠规则跟变量的const特性是没有关系的。

```
A& & => A&
A& && => A&
A&& & => A&
A&& && => A&&
```

## 特殊模板参数推导规则

下面我们再来学习特殊模板参数推导规则，考虑下面的模板函数，模板函数接收一个右值引用作为模板参数。

```c++
template<typename T>
void foo(T&&);
```

说白点，特殊模板参数推导规则其实就是引用折叠规则在模板参数为右值引用时模板情况下的应用，是引用折叠规则的一种情况。我们结合上文中的引用折叠规则，

1. 如果foo的实参是上文中的A类型的左值时，T的类型就为A&。根据引用折叠规则，最后foo的参数类型为A&。
2. 如果foo的实参是上文中的A类型的右值时，T的类型就为A&&。根据引用折叠规则，最后foo的参数类型为A&&。

## 解决完美转发问题

我们已经学习了模板参数为右值引用时的特殊模板参数推导规则，那么我们利用刚学习的知识来解决本文中待解决的完美转发的例子。

```c++
#include <iostream>

using namespace std;

void fun(int &) { cout << "lvalue ref" << endl; }
void fun(int &&) { cout << "rvalue ref" << endl; }
void fun(const int &) { cout << "const lvalue ref" << endl; }
void fun(const int &&) { cout << "const rvalue ref" << endl; }

//template<typename T>
//void PerfectForward(T t) { fun(t); }

// 利用引用折叠规则代替了原有的不完美转发机制
template<typename T>
void PerfectForward(T &&t) { fun(static_cast<T &&>(t)); }

int main()
{
    PerfectForward(10);           // rvalue ref，折叠后t类型仍然为T &&

    int a;
    PerfectForward(a);            // lvalue ref，折叠后t类型为T &
    PerfectForward(std::move(a)); // rvalue ref，折叠后t类型为T &&

    const int b = 8;
    PerfectForward(b);            // const lvalue ref，折叠后t类型为const T &
    PerfectForward(std::move(b)); // const rvalue ref，折叠后t类型为const T &&

    return 0;
}
```

例子中已经对完美转发的各种情况进行了说明，这里需要对PerfectForward模板函数中的static_cast进行说明。static_cast仅是对传递右值时起作用。我们看一下当参数为右值时的情况，这里的右值包括了const右值和非const右值。

```c++
// 参数为右值，引用折叠规则引用前
template<int && &&T>
void PerfectForward(int && &&t) { fun(static_cast<int && &&>(t)); }

// 引用折叠规则应用后
template<int &&T>
void PerfectForward(int &&t) { fun(static_cast<int &&>(t)); }
```

可能读者仍然没有发现上述例子中的问题，“不用static_cast进行强制类型转换不是也可以吗？”。别忘记前文中仍然提到一个右值引用和右值之间关系的规则，`如果右值引用有名字则为左值，如果右值引用没有名字则为右值。`。这里的变量t虽然为右值引用，但是是左值。如果我们想继续向fun函数中传递右值，就需要使用static_cast进行强制类型转换了。

其实在C++11中已经为我们封装了std::forward函数来替代我们上文中使用的static_cast类型转换，该例子中使用std::forward函数的版本变为了：

```c++
template<typename T>
void PerfectForward(T &&t) { fun(std::forward<T>(t)); }
```

对于上文中std::move函数的实现也是使用了引用折叠规则，实现方式跟std::forward一致。

# 引用

1. 《深入理解C++11-C++11新特性解析与应用》
2. [C++11 标准新特性: 右值引用与转移语义](http://www.ibm.com/developerworks/cn/aix/library/1307_lisl_c11/)
3. [如何评价 C++11 的右值引用（Rvalue reference）特性？](http://www.zhihu.com/question/22111546)
4. [C++11 完美转发](http://blog.bitdewy.me/blog/2013/07/08/cpp11-perfect-forward/)
5. [C++ Rvalue References Explained](http://thbecker.net/articles/rvalue_references/section_01.html#section_01)
6. [详解C++右值引用](http://jxq.me/2012/06/06/%E8%AF%91%E8%AF%A6%E8%A7%A3c%E5%8F%B3%E5%80%BC%E5%BC%95%E7%94%A8/) （对C++ Rvalue References Explained的翻译）