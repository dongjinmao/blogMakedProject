---
title: C语言学习
toc: true
date: 2023-10-21 14:03:39
tags:
categories: C语言
---

# C语言学习

## 一、头文件

###  需求：

在实际的开发中，我们往往需要在不同的文件中，去调用其他文件的定义的函数，比如 **hello.c** 中，去使用 **myfuns.c** 文件中的函数，如何实现？

### 头文件基本概念

1. 头文件事扩展名为 **.h** 的文件，包含了C函数声明和宏定义，被多个源文件中引用共享。有两种类型的头文件：自己编写的头文件和C标准库自带的头文件。
2. 在程序中要使用头文件，需要使用C预处理指令 **#include** 来引用它。比如 **stdio.h** 头文件。
3. **#include** 叫做文件包含命令，用来引入对应的头文件 **.h文件** 。**#include** 也是 C语言 **预处理命令** 的一种。
4. **#include** 的处理过程很简单，就是将头文件的内容插入到该命令所在的位置，从而把头文件和当前源文件连接成一个源文件，这与复制粘贴的效果相同。但是我们不会直接在源文件中复制头文件的内容，因为这么做容易出错，特别在程序是由多个源文件组成的时候。
5. 建议把所以的**常量**、**宏**、**系统全局变量**和**函数原型**写在头文件中，在需要的时候随时引用这些头文件。

### 工作原理

使用头文件示意图

![使用头文件示意图](learnC/使用头文件示意图.png)
