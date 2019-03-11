---
title: GMapping 项目编译及源码阅读
tags:
  - SLAM
categories:
  - SLAM
date: 2019-03-11 19:14:16
---

# 简介
GMapping 是比较古老的SLAM算法，使用粒子滤波融合雷达和运动学模型数据，从而完成建图与定位。GMapping 的源码可以在 OpenSLAM 的 [Github](https://github.com/OpenSLAM-org/openslam_gmapping) 页面查到，GMapping 的算法在以下博客中有很好的解释，本文主要分析一下GMapping 的项目结构。

http://blog.csdn.net/heyijia0327/article/details/40899819 pf原理讲解
http://blog.csdn.net/u010545732/article/details/17462941 pf代码实现
http://www.cnblogs.com/yhlx125/p/5634128.html gmapping分析
http://wenku.baidu.com/view/3a67461550e2524de4187e4d.html?from=search gmapping 分析 
https://blog.csdn.net/roadseek_zw/article/details/53316177 gmapping 分析

话不多说，直接开始分析。

# 编译
首先需要运行以下命令进行初始化配置，生成 global.mk 来配置全局变量。
```
./configure
```
然后运行'make'进行编译，然后编译器会逐句执行 makefile 中的命令，GMapping 的 makefile 内容如下。
```
//调用global.mk 配置全局变量
-include ./global.mk

//根据系统环境设置编译模块
ifeq ($(CARMENSUPPORT),1)
SUBDIRS=utils sensor log configfile scanmatcher carmenwrapper gridfastslam gui gfs-carmen 
else
ifeq ($(MACOSX),1)
SUBDIRS=utils sensor log configfile scanmatcher gridfastslam 
else
SUBDIRS=utils sensor log configfile scanmatcher gridfastslam gui 
endif
endif

LDFLAGS+=
CPPFLAGS+= -I../sensor

//调用 Makefile.subdirs 对子模块进行编译
-include ./build_tools/Makefile.subdirs
```
该文件首先调用 global.mk 配置全局参数，然后根据系统环境（操作系统，第三方库Carmen）选择编译的模块，并保存到变量 SUBDIR 中，然后调用 build_tools 文件夹下 Makefile.subdirs，依次编译子文件夹对应的模块。而每个子文件夹中都有对应的 makefile 规定了编译对象，依赖库和测试对象，并使用 Makefile.generic-shared-object 进行编译。以 gridfastslam 为例，该目录下的 makefile 内容如下，具体指令功能用注释标出。
```
//编译过程中生成目标文件 OBJS APPS
OBJS= gridslamprocessor_tree.o motionmodel.o gridslamprocessor.o gfsreader.o
APPS= gfs2log gfs2rec gfs2neff #gfs2stat

//编译过程中需要链接的库文件
#LDFLAGS+= -lutils -lsensor_range -llog -lscanmatcher -lsensor_base -lsensor_odometry $(GSL_LIB)
LDFLAGS+=  -lscanmatcher -llog -lsensor_range -lsensor_odometry -lsensor_base -lutils
#CPPFLAGS+=-I../sensor $(GSL_INCLUDE)
CPPFLAGS+=-I../sensor

-include ../global.mk
-include ../build_tools/Makefile.generic-shared-object
```

# 项目分析
GMapping 的主要算法在子目录 gridfastslam 下，封装在类 GridSlamProcessor 当中。

# 参考链接
1. [CSDN博客: gmapping 分析](https://blog.csdn.net/roadseek_zw/article/details/53316177)
2. [OpenSLAM](https://openslam-org.github.io/)
3. [GMapping Github 页面](https://github.com/OpenSLAM-org/openslam_gmapping)
4. Giorgio Grisetti, Cyrill Stachniss, and Wolfram Burgard: Improved Techniques for Grid Mapping with Rao-Blackwellized Particle Filters, IEEE Transactions on Robotics, Volume 23, pages 34-46, 2007[link](http://www.informatik.uni-freiburg.de/~stachnis/pdf/grisetti07tro.pdf)
5. Giorgio Grisetti, Cyrill Stachniss, and Wolfram Burgard: Improving Grid-based SLAM with Rao-Blackwellized Particle Filters by Adaptive Proposals and Selective Resampling, In Proc. of the IEEE International Conference on Robotics and Automation (ICRA), 2005[link](http://www.informatik.uni-freiburg.de/~stachnis/pdf/grisetti05icra.pdf)
