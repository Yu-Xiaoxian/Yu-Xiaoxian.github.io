---
title: C++是一个语言联邦
tags:
  - C++
categories:
  - Wiki
date: 2020-05-26 21:43:16

updated:: 2020-05-26 22:01:16
---

点击链接查看更多C++ 技巧 ：[Effective C++](https://blog.yu-xiaoxian.me/2019/05/03/wiki/EffectivCpp/)

--------------------------------

C++ 是一个强大的编程语言，但他的风格并不统一，这是由于C++是一个语言联邦，由以下四部分组成，每部分都有自己的特点。

- C语言：
- 面向对象的C++：包括类，类的派生和继承
- 模板元编程的C++：模板元编程是图灵完全的语言，也有自己的风格特点
- STL库：STL是官方提供的标准库

## C 语言
C++ 最早就是 C 语言的预编译器，兼容了C语言的所有特性，然而C语言的类型转换不够安全，C++中做了安全的类型转换，导致C++和C的特性有一定的差别，在编程时要区分是C 还是 C++
## 面向对象的C++
这部分的C++主要是类的派生、继承、聚合
## 模板元编程的C++
模板元编程已经被证明是图灵完全的语言，风格特点和传统C++又有所不同
## STL
STL是官方提供的标准库，为保证标准库的高效性，库的开发者使用了部分C的特性，因此会使得STL有一点像C，也有一点像C++，需要仔细甄别

----

点击链接查看更多C++ 技巧 ：[Effective C++](https://blog.yu-xiaoxian.me/2019/05/03/wiki/EffectivCpp/)