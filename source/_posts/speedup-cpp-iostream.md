---
title: 加速C++的读写
date: 2019-08-15 16:35:36
tags: cpp

---

---

今天在[LeetCode](www.leetcode.com)刷题，看到高手的答案中有下面这行代码，了解了一下这些代码的含义，发现了提高C++读写速度的方法。

```cpp
static auto speedup = [](){
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    return nullptr;}();
```

C++中的输入输出流为std::cin和std::cout，与之对应的C中的输入输出分别为scanf()和printf()。不少人有误解说C++的输入输出比C的慢，然而这时错误的认知，玄机就体现在以上代码中。C++为兼容C，并保证输入输出的线程安全，将C++的输入输出流与C的绑定在了一起，也就是说C++并不维护缓存区，而是将保存在C的缓存区中，这样每次IO都会调用一次flush()，因此导致开销较大。

而cin.tie()默认将输入绑定到cout，这样每一行输入都能够被打印出来。而将cin绑定到空指针，能够进一步提高输入的速度。

## 参考资料

1. [cpp-reference: sync_with_stdio](https://en.cppreference.com/w/cpp/io/ios_base/sync_with_stdio)
2. [cin.tie与sync_with_stdio加速时输入输出](https://blog.csdn.net/jrrrj/article/details/81319369)