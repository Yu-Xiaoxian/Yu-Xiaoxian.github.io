---
title: C++ const 详解
date: 2022-07-02 21:29:47
tags:
  - cpp
categories:
  - Wiki
---

# 编译器不按套路出牌，该怎么办？const 成员函数深度剖析

### 一、const 成员函数

首先来复习一下 const 成员函数，我们自己实现一个字符串 MyStr，内部使用 char* 指针保存原始数据，使用 get_length() 函数获取数组的长度，由于 get 函数不会修改成员变量，可以使用 const 关键字修饰 get_length() 函数，编译期就会帮我们检查是否修改了成员变量，如果有修改，就会在编译期报错，这样就可以保护成员变量不被修改。
```
class MyStr {
public:
  MyStr() {...}
  ~MyStr() {...}
  size_t get_length() const {
    // _length = 1;  编译出错
    return _length;
  }
  
private:
  char* _data = nullptr;
  size_t _length = 0;
}
```
### 二、logic const 和 bitwise const
编译器是怎么实现 const 成员检查的呢？在编译时它会检查该函数有没有对成员变量进行修改，比如上面代码里的赋值操作 `_length = 1` 就明显修改了成员变量，发现这样的操作编译器就会报警，这种校验就是 bitwise const。

然而只有 bitwise const 是不够的，设想 MyStr 中的 `_data` 使用了高速缓存，还有别的操作会修改字符串的长度，`get_length()` 函数需要改成下面这样。

```
size_t get_length() const {
  _length = strlen(_data); // 编译器会报错
  return _length;
}
```
更新字符串长度并没有改变 `_data` 的内容，从逻辑（logic）来讲符合 const 函数的定义，但编译器并不这样认为，我们可以使用 `mutable` 字段让编译器不再阻拦我们，像下面这样
```
MyStr {
public:
  MyStr() {...}
  ~MyStr() {...}
  size_t get_length() const {
    _length = strlen(_data); // 编译器不会报错
    return _length;
  }
  
private:
  char* _data = nullptr;
  mutable size_t _length = 0;
}
```
这种逻辑上不改变数据的 const 函数就叫做 logic const。

### 三、编译器检查不出怎么办
上面讲了一种绕过编译器检查的情况，下面来看一种编译器检查不出来的问题，当类的成员变量为指针时，只改变指针指向的数据而不改变指针，编译器是不会认为违背 const 原则的。比如我们提供 `operater[]` 操作符来访问 MyStr 的元素
```
class MyStr {
public:
  ...
  char& operator[](const size_t index) const {
    return _data[index];
  }
private:
  char* _data = nullptr;
  size_t _length = 0;
}

MyStr my_str;
my_str[0] = 'a'; // 这样就通过const函数改变了成员变量指针指向的内容，并且编译器不会报错
```
这种情况怎么办呢？我们一定要记得把 const 函数的返回值设置为 const
```
class MyStr {
public:
  ...
  const char& operator[](const size_t index) const {
    return _data[index];
  }
private:
  char* _data = nullptr;
  size_t _length = 0;
}

MyStr my_str;
my_str[0] = 'a'; // 这样编译器就会报错，因为我们修改了常引用
```

### 总结
最后把前面讲的总结为两点：

1. 想要绕过编译器 const 检查时，对成员变量增加 mutable 修饰
2. 常量函数的返回值（特别是引用和指针）一定要加 const 修饰