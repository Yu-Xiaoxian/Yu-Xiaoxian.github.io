---
title: RAII 目前最好的资源管理策略
tags:
  - C++
categories:
  - Wiki
date: 2021-01-19 21:43:16

updated:: 2021-01-19 22:01:16
---


点击链接查看更多C++ 技巧 ：[Effective C++](https://blog.yu-xiaoxian.me/2019/05/03/wiki/EffectivCpp/)
----

今天遇到一个bug，记录一下，简而言之就是：绝对不要显式调用局部变量的析构函数。为什么呢，下面详细介绍一下

首先来看下面一段代码，可以不看结果，先想一下输出结果是什么？
```cpp
#include <stdio.h>

class Mocker{
public:
  Mocker(const int size):size_(size){
    printf("Construct Mocker\n");
  }
  ~Mocker(){
    printf("Deconstruct %d\n", size_);
  }
private:
  int size_ = 0;
};

int main(){
  {
    Mocker m1(1);
    m1.~Mocker();
  }
  return 0;
}
```

不要看结果，先想想输出是什么？

-----

想好了吗，现在揭晓答案

```shell
Construct Mocker
Deconstruct 1
Deconstruct 1
```

很奇怪，竟然输该出了两个“Deconstructor 1”，析构函数被调用了两次。这是由于 C++ 会保证局部变量离开作用域时，该变量的析构函数被第一时间调用。所以就没有必要再手动调用析构函数了。

上面这个例子中，调用两次析构函数，不会造成不良的后果。但如果是下面这种情况，类Mocker中申请了一块内存，在析构函数中被释放，如果析构函数被调用两次，就会导致程序异常终止。

```cpp
class Mocker{
public:
  Mocker(const int size):size_(size){
    data_ = new int[size_];
    printf("Construct Mocker\n");
  }
  ~Mocker(){
    delete[] data_;
    printf("Deconstruct %d\n", size_);
  }
private:
  int size_ = 0;
  int* data_ = nullptr;
};
```

所以，使用 C++ 的时候切记一点，**绝对不要**手动调变量用**局部变量**的析构函数。

----

点击链接查看更多C++ 技巧 ：[Effective C++](https://blog.yu-xiaoxian.me/2019/05/03/wiki/EffectivCpp/)
