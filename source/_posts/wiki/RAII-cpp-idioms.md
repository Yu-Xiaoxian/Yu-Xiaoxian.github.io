---
title: RAII 目前最好的资源管理策略
tags:
  - C++
categories:
  - Wiki
date: 2021-01-03 21:43:16

updated:: 2021-01-03 22:01:16
---


点击链接查看更多C++ 技巧 ：[Effective C++](https://blog.yu-xiaoxian.me/2019/05/03/wiki/EffectivCpp/)
----

RAII(Resource Aquisition Is Initialization，翻译过来就是“资源获取即初始化”)，是C++之父 Bjarne Stroustrup 提出的一种编程用例，也是目前最好的资源管理方案。为什么说是最好，我们需要先了解一下传统的资源管理有哪些？

传统的资源管理主要有两种：

 - 以C/C++为首的 malloc/new 方法，由程序员手动管理资源，资源保存在堆上
 - 以Java为首的垃圾回收机制（Garbage Collection），由运行环境和管理资源，资源保存在栈上

那么这两种方案各有什么优劣呢？
- C/C++ 的方法，程序员可以精确控制资源获取和释放的时机，充分利用系统的性能。风险就是资源获取与释放必须成对出现，如果忘记释放会占用系统资源，而过早释放有可能导致内存泄漏
- Java 的方法将程序员从繁琐的资源管理中释放出来，但也导致程序员无法准确判断资源获取和释放的时机，有可能导致资源的浪费。

这里有一种方式，可以在保证资源控制精度的同时，避免大量的繁琐操作，那就是RAII(Resource Aquisition Is Initialization)，他巧妙利用了C++作用域的特性和类的析构函数，将资源的申请和释放交给编译器去完成，从而将程序员从繁琐的资源管理中释放出来。下面就来详细介绍RAII是怎样实现的。

请看下面一段代码，申请一段内存，在使用完成后将其释放，这里需要我们来手动管理资源
```cpp
int * arr = new int[100];
// do something
delete[] arr;
```

如果将资源的申请和释放分别放在类的构造和析构函数中，我们可以达到同样的目的
```cpp
class Arr{
public:
	Arr(int size) { ptr_ = new int[size]; }
	~Arr() { if(ptr_ != nullptr; ) delete[] ptr_; }
private:
	int *ptr_ = nullptr;
}

{
	Arr arr(100);
	// do somethin
}
// When leaving the zoen, the array will be released automatically
```

我们实例化了arr，根据 C++ 语言的特性，当 arr 离开 {} 所限制的作用域后，析构函数 Arr::~Arr() 会被自动调用，从而释放申请的内存。

这样的设计就省去了自动管理资源的麻烦，这里只是举了一个简单的例子，在更复杂的工程中，使用RAII的方式，帮我们节省的精力是十分巨大的。比如 C++11 中智能指针 shared_ptr 和 unique_ptr，就是 RAII 的经典应用，大家感兴趣的话可以点击链接进行了解[智能指针](https://blog.csdn.net/flowing_wind/article/details/81301001)。

大家可以关注我的公众号「于振洋」，学习更多C++技巧
![公众号二维码](https://img-blog.csdnimg.cn/20200722012125256.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1X3hpYW94aWFuXzIwMTg=,size_16,color_FFFFFF,t_70#pic_center)

----

点击链接查看更多C++ 技巧 ：[Effective C++](https://blog.yu-xiaoxian.me/2019/05/03/wiki/EffectivCpp/)
