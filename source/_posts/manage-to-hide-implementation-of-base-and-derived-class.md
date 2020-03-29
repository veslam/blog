---
title: 隐藏继承关系中父类子类的实现 manage-to-hide-implementation-of-base-and-derived-class
date: 2016-11-07 18:46:27
tags: [coding, c++, multiple inheritance, Diamond of Death]
---
### 缘起 ###
最近一段时间的主要工作是制作一个*iOS Static Library*，那么必然要做到的就是实现的隐藏，只外露一些包含公开方法的头文件，随*.a*文件一同交付，无论动机是代码保密，或是尽量简化对外接口。
如何做到呢？

### 分析 ###
通过搜索找到一种称作[C++桥接设计模式(impl)](http://blog.csdn.net/just_kong/article/details/10060461)的方法，看起来比较复杂一些，并未深入研究。
现在，想象一下，如果在外露的头文件里用户看到的是类X，而库的实现中拥有私密成员和方法的其实是类Y，两者是什么关系呢？
大致无非两种：X私藏一个Y的指针，把所有方法指派给Y去做，即X *has-a* Y；或者，Y继承自X，并把X承诺的接口都实现了，即Y *is-a* X。两者孰优孰劣应该比较明显：前者X每新增一个方法，都要在Y增加一个相应的方法，并X实现中手动调用，有些繁琐，不是长久之计，而且从直觉上不满足XY地位平等。其实，后者采用的*继承*也正是对于这些疑问的完美解决方案，无需在X，Y中两份声明，重载和虚函数也能提供更大的灵活性和可能性。

##### 简单版需求：要隐藏某一个类#####

为了和后文统一，就称这个类名为**Base**。那么要做的是：首先将其隐姓埋名，改称**BaseImpl**（Impl是implementation的简称），再建立一个冒名顶替的**Base**类作为他的爸爸。
![BaseImpl继承Base](https://raw.githubusercontent.com/veslam/blog/master/res/20161107_01_HideImpl.png)

具体实现方式：**继承** + **Factory Method** + **纯虚方法** / **GetImpl() downcasting** （二选一）
参考链接：[Hiding-Implementation-Details-in-C](http://www.codeproject.com/Articles/42466/Hiding-Implementation-Details-in-C)最后一段。作者把BaseImpl的定义和实现都放进了Base.cpp，但我做了些改动以使结构清晰，将两个类的定义和实现拆了出来放到各自的.h/.cpp中。

1) 采用**纯虚方法**（个人推荐）：
![继承 + Factory Method + 纯虚方法 (灰色箭头为#import关系)](https://raw.githubusercontent.com/veslam/blog/master/res/20161107_02_HideImpl.png)
>The client no longer sees any of the implementation details. The cost is that we are now required to have a factory method and virtual methods.

``` C 
// Define a public interface in foo.h
class Foo
{
public:
   virtual void SetPosition(float x, float y) = 0; // 纯虚
   static Foo* Create();
protected:
   Foo() {}    // hide
};
```
``` C
// foo.cpp
// Define factory method
Foo* Foo::Create()
{
    return new FooImpl();
}

```
``` C
// Define a private implementation in fooImpl.h
class FooImpl : public Foo
{
public:
   FooImpl() {}
   void SetPosition(float x, float y);
private:
   float m_x, m_y;      // private data
   void PrivateMethod() {}  // private method
};
```
``` C
// fooImpl.cpp
void FooImpl::SetPosition(float x, float y)
{
    m_x = x; m_y = y;
}
```

2) 采用 **GetImpl() downcasting 方法**：
![继承 + Factory Method + GetImpl() downcasting (灰色箭头为#import关系)](https://raw.githubusercontent.com/veslam/blog/master/res/20161107_03_HideImpl.png)
>By downcasting in the class methods, we get access to the implementation data without virtual methods.

``` C 
// Define a public interface in foo.h
class Foo
{
public:
   virtual void SetPosition(float x, float y);
   static Foo* Create();
protected:
   Foo() {}    // hide
};
```
``` C
// foo.cpp
// Define factory method
Foo* Foo::Create()
{
    return new FooImpl();
}

// downcasting methods
inline FooImpl * GetImpl(Foo* ptr) { return (FooImpl *)ptr; }
inline const FooImpl * GetImpl(const Foo* ptr) { return (const FooImpl *)ptr; }

// Define public method
void Foo::SetPosition(float x, float y)
{
    FooImpl * f = GetImpl(this);
    f->SetPosition(x, y);
}
```
*fooImpl.h fooImpl.cpp* 与方法1)中无异。

到此为止，已经实现了对单个类的隐藏。如果原本没有涉及继承的话，至此已满足需求。
这样实现的好处是，原来你在**foo**中实现的各种私有内容，比如各种*private, friend class, enum, typedef function*等，都原封不动的转移到**fooImpl**的实现就行，就像前面说的，**fooImpl**实际上就是原来的**foo**。

---

然而我的最终目标是**隐藏一对本来已经存在继承关系的父子类**，在依照上述方法2)拓展时遭遇无法解决的问题。
结构应该拓展为下图的关系，这样既满足外露的**Base**和**Derived**类的父子关系，又满足**BaseImpl**和**DerivedImpl**类的父子关系。
![继承关系](https://raw.githubusercontent.com/veslam/blog/master/res/20161107_04_HideImpl.png)
看到**DerivedImpl**继承了两个同源类，不好的预感已经产生……没错，就是*菱形继承 (Diamond of Death)* 之前还没有在实践中遇到过类似问题，这次花费的很多时间作为补课。

>Base* d = Derived::Create();
d->SetPosition(5, 10);

第二句就崩，查看变量里面的内容，各种值都是随机值，看来是指针类型转换导致内存混乱。
尝试分析一下：第一句，创建了**DerivedImpl**向上转换为**Derived***再向上转换为**Base***，第二句调用方法时，又向下转换为**BaseImpl***。

[How can I avoid the Diamond of Death when using multiple inheritance?](http://stackoverflow.com/a/139329/2918210)其中的答案解释了一些疑惑：

>A practical example:
>>class A {};
class B : public A {};
class C : public A {};
class D : public B, public C {};

>Notice how class D inherits from both B & C. But both B & C inherit from A. That will result in 2 copies of the class A being included in the vtable.

也就是说，**DerivedImpl**对象里会保存2份**Base**，然而他的父亲和爷爷都不是这个结构，指针强制类型转换后出问题也就有可能了。那么正确的做法是什么呢？
>To solve this, we need virtual inheritance. It's class A that needs to be virtually inherited. So, this will fix the issue:
>>class A {};
class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};

啊哈，就是让中间两个父类**虚继承**爷爷类！那么扩展方法1)，我的实现结构应该如下：
![结构图 Base和Derived，BaseImpl和DerivedImpl分别共用.h和.cpp](https://raw.githubusercontent.com/veslam/blog/master/res/20161107_05_HideImpl.png)
举个栗子🌰
``` C 
// Define a public interface in Base.h
class Base
{
public:
   static Base* Create();
   virtual void setPosition(float x, float y) = 0;
   virtual Point getPosition() = 0;
   virtual float getArea() = 0;
};

class Derived : virtual public Base
{
public:
   static Derived* Create();
   virtual void setSomething(float s) = 0;
};
```
``` C
// Base.cpp
Base* Base::Create()
{
    return new BaseImpl();
}

Derived* Derived::Create()
{
    return new DerivedImpl();
}
```
``` C
// BaseImpl.h
class BaseImpl : virtual public Base
{
   friend class xxx;
public:
   BaseImpl() {}
   void setPosition(float x, float y);
   Point getPosition();
   virtual float getArea();
private:
   float m_x, m_y, area2D;      // private data
   void privateMethod();        // private method
};

class DerivedImpl : public Derived, public BaseImpl
{
public:
   void setSomething(float s);
   virtual float getArea();     // overwrite
private:
   float area3D;
};
```
``` C
// BaseImpl.cpp
void BaseImpl::setPosition(float x, float y)
{
    m_x = x; m_y = y;
}
Point BaseImpl::getPosition()
{
    return Point(m_x, m_y);
}
float BaseImpl::getArea()
{
    return area2D;
}
void BaseImpl::privateMethod()
{ // do something. }

void DerivedImpl::setSomething(float s)
{
  // blablabla
}
float DerivedImpl::getArea()
{
    return area3D;
}
```

---
好啦！现在我们已经实现了对外的实现隐藏，然而最后还要解决一个问题。
用户Create的是**Base**和**Derived**，库内部我们自己写的函数需要访问的是**BaseImpl**和**DerivedImpl**，因此需要从B/D到BImpl/Dimpl的转换，如何做呢？答案是__*dynamic_cast*__。
在问题[C++ cannot convert from base A to derived type B via virtual base A](http://stackoverflow.com/a/3749251/2918210)的解答中有述：
>This is what **static_cast** do: a static cast is dubbed static because the computation of what is necessary for the cast is done at compile-time, be it pointer arithmetic or conversions (*).
>
However, when **virtual** inheritance kicks in things tend to become a bit more difficult. The main issue is that with **virtual** inheritance all subclasses share a same instance of the subobject. In order to do that, **B** will have a pointer to a **A**, instead of a **A** proper, and the A base class object will be instantiated outside of **B**.
Therefore, it's impossible at compilation time to be able to deduce the necessary pointer arithmetic: it depends on the runtime type of the object.
>
In summary:
 - compile-time downcast: **static_cast**
 - run-time downcast: **dynamic_cast**

即，**static_cast**只能用于在编译时已可确定对象类型的情况，而涉及**虚继承**的情况，需要动态转化类型，所以要用**dynamic_cast**。

终于大功告成！😃

---
### 知识补充 ###
__*纯虚函数*__ 声明后有“＝0”的函数，如*virtual void function() = 0;* 纯虚函数不可实现。
__*纯虚类*__ 定义中含有*纯虚函数*的类。纯虚类不能通过构造函数new出一个实例，但可以有指针对象。
__*virtual static function()*__不可存在 因为*static*的目的是生成只与类有关的全局固态方法，而*virtual*的目的是根据具体对象的类别，动态灵活指向函数，两者互斥。
