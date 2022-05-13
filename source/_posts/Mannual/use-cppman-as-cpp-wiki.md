---
title: 在命令行查询 C++ 用法
date: 2022-05-13 21:29:47
tags:
  - cpp
---

不知道大家有没有同样的经历，在搬砖酣畅淋漓的时候，每一次切屏都是对思路的严重干扰，特别是切了好几屏都找不到浏览器的时候，简直要爆炸。
巧的是，GitHub 网友 aitjcize 也有同样的困扰，于是他开发了一个项目 cppman，可以直接在命令行查看 C++ 相关的头文件和用法，如同 man 一样的操作，简直不要太方便。

Ubuntu 下安装
```shell
sudo apt install cppman
```
查询想要的功能，会默认从 cplusplus.com 拉取最新的内容
```shell
cppman std::vector
```
当然你也可以设置默认源为 cppreference.com
```shell
cppman -s cppreference.com
```
也可以提前缓存到本地，以备不时之需
```shell
cppman -c
```

学了这个技巧，是不是觉得自己编程效率又提高了一点点呢？快来关注我学习更多技巧吧！