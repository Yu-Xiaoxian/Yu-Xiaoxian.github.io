---
title: C++小技巧：RAII 的应用-智能指针和范围锁
tags:
  - C++
categories:
  - Wiki
date: 2021-01-15 21:43:16

updated:: 2021-01-15 22:01:16
---


点击链接查看更多C++ 技巧 ：[Effective C++](https://blog.yu-xiaoxian.me/2019/05/03/wiki/EffectivCpp/)
----

上篇文章介绍了 [RAII](https://blog.yu-xiaoxian.me/2021/01/03/wiki/RAII-cpp-idioms/)，这篇文章介绍下RAII的常见应用：智能指针和范围锁(Scoped Lock)，这两者都利用了RAII的思想来管理资源，将程序员从繁琐又容易出错的资源管理中释放出来，大大降低了程序出错的概率。

本文以 C++11 为例对这两者的应用进行简单介绍，同理其它扩展库 Boost 和 Abseil-Cpp 的用法也类似。

## 智能指针
智能指针主要来管理动态申请的指针，传统的内存管理需要开发者手动申请和释放内存，而智能指针采用了 RAII 的思想，开发者只需要申请内存而不用操心何时释放，因为在该指针离开作用域的时候自动释放。

普通指针用法
```cpp
// 手动申请一块内存
auto ptr = new int[64];
// 手动释放一块内存
delete[] ptr;
```
智能指针用法
```cpp
auto smart_ptr = std::unique_ptr<int[]>(new int[64]);
```
当 smart_ptr 离开作用域后，编译器会自动释放该指针

## 范围锁
范围锁是用来管理互斥量的，传统方法需要手动上锁和解锁互斥量，范围锁采用 RAII 的思想，在需要上锁时获得锁，离开作用域后就会自动释放该互斥量。

直接操作互斥量
```cpp
std::mutex mtx;
mtx.lock();
// do something
mtx.unlock();
```

使用范围锁
```cpp
std::mutex mtx;
{
	std::lock_guard<std::mutex> locker(mtx);
	// do something
}
```
构造 locker 时会将mtx上锁，当 locker 离开作用域后，编译器会自动释放该mutex，为了保证可以及时释放该互斥量，可以像上面代码一样，拿花括号将需要获得锁的代码段包起来。

----

点击链接查看更多C++ 技巧 ：[Effective C++](https://blog.yu-xiaoxian.me/2019/05/03/wiki/EffectivCpp/)
