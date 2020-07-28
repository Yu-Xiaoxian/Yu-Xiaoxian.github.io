---
title: C++小技巧：const 和 constexpr
date: 2020-07-28 22:50:26
tags: cpp
---

太长不看版：

- constexpr 表示常量，告诉编译器这个变量可以尽情优化
- const 表示该变量是只读

---

下面正文开始，在介绍 const 和 constexpr 的区别之前，请允许我卖个关子，介绍一下编译器对常量的优化，先看下面一段简单的除法例程

```cpp
int main(){
    const int a = 123;
    int b = 3;
    float c = b/a;
}
```
经过编译器优化生成的汇编代码如下
```c
main:
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], 123
        mov     DWORD PTR [rbp-8], 3
        // 除法运算开始
        mov     ecx, DWORD PTR [rbp-8]
        mov     edx, 558694933
        mov     eax, ecx
        imul    edx
        sar     edx, 4
        mov     eax, ecx
        sar     eax, 31
        sub     edx, eax
        mov     eax, edx
        pxor    xmm0, xmm0
        cvtsi2ss        xmm0, eax
        movss   DWORD PTR [rbp-12], xmm0
        // 除法运算结束
        mov     eax, 0
        pop     rbp
        ret
```
如果把 main() 中第2行的 const 关键字去掉，那么生成的汇编代码如下
```c
main:
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], 123
        mov     DWORD PTR [rbp-8], 3
        // 除法开始
        mov     eax, DWORD PTR [rbp-8]
        cdq
        idiv    DWORD PTR [rbp-4]
        pxor    xmm0, xmm0
        cvtsi2ss        xmm0, eax
        movss   DWORD PTR [rbp-12], xmm0
        // 除法结束
        mov     eax, 0
        pop     rbp
        ret
 ```
看不懂汇编没有关系，粘贴这两段代码的用意就是告诉大家，编译器会对常量运算进行疯狂的优化，尽可能地提高程序运行的效率，因此我们在编写程序的时候，也要尽可能地将常量声明为 const，从而提高程序运行的效率。



下面来看 constexpr，这是C++11 中引入的新关键字，顾名思义：常量(const)+表达式(expr)，也就是说可以用声明一个表达式为常量。看下面这个例子，这样的代码是非法的，因为函数 get_value() 只有在运行期才会被调用，而数组的长度必须是编译期已知的量
```cpp
int get_value(){return 5;}
int arr[get_value() + 7]; // 声明长度为 12 的数组
```
而使用 constexpr 声明函数的话，就可以顺利编译，因为可以在编译期调用常量函数了
```cpp
constexpr int get_value(){return 5;}
int arr[get_value() + 7;] // 声明长度为 12 的数组
```
讲到这里 constexpr 的含义也就呼之欲出了，constexpr 的含义就是告诉编译器，该表达式的值可以在编译期计算得到，让编译器能够尽情优化。

除了定义函数表达式，constexpr 也可以用于声明常量
```cpp
constexpr int a = 1234;
constexpr float b = 334.0f;
constexpr char c[] = "Hello";
```
终于到 const 出场了，在C++ 中 const 包含了两种含义：常量 和 只读，通过上面的讨论我们明白了 constexpr 也可以表示常量，两者在常量定义上的功能存在交集，在声明常量变量的情况下，两者作用相同。但是为了程序的规范易读，我们不妨做一些限制规范：

- constexpr 用于声明所有的常量以及常量表达式

- const 用于声明变量为只读，不可更改

大家可以关注我的公众号「于振洋」，学习更多C++技巧
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722012125256.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l1X3hpYW94aWFuXzIwMTg=,size_16,color_FFFFFF,t_70#pic_center)
