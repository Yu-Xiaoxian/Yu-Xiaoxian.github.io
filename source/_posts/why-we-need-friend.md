---
title: C++为什么会有“友元”这种破坏封装的功能？
date: 2022-05-07 21:29:47
tags:
  - cpp
---

最近有朋友问了一个问题：C++里的“友元”破坏了封装，让类外的成员能够访问类的私有成员，不是与面向对象的设计思想相违背吗？

首先我们来温习一下什么是友元，如下面代码所示，声明 类Bar 为 类FooB 的友元，那么 Bar 可以访问类 FooB 的私有成员，但 FooA 的私有成员是无法被 Bar 访问到的，编译会报错

```cpp
class Bar;
class FooA {
  public:
  
  private:
    int data_;
};

class FooB {
  public:
    friend class Bar;
  private:
    int data_;
};

class Bar {
  public:
    void set_data(const int data) {
        // accessing foo_a_.data_ is forbidened
        // foo_a_.data_ = data;
        foo_b_.data_ = data;
    }
  
  private:
    FooA foo_a_;
    FooB foo_b_;
};
```

由上可知，友元提供了一种类外访问类私有变量的方法，可是为什么 C++ 要提供这样一种破坏封装的方法呢？这就不得不提 C++ 的设计哲学--“Trust Programmer”，不人为设限，为程序员提供一切可能的操作，由程序员来保证程序的正确性。换句话来说，C++ 不是为某种编程范式服务的，而是为程序员提供一切可能会用到的工具，由程序员来自由的选择和组合，所以才会有“瑞士军刀”的美誉。

明白了 C++ 的这一设计理念，再来看“友元”的设计，是不是就合理了许多呢？