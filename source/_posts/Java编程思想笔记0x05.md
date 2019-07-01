---
title: Java编程思想笔记0x05
date: 2019-07-01 20:19:43
categories: Java
tags:
  - Java
  - 笔记
---

# 类型信息（RTTI）

## Class对象

- Java使用`Class`对象来执行其RTTI。
- 类是程序的一部分，每个类都有一个`Class`对象
- 类加载过程（见[深入理解Java虚拟机笔记0x02](2019/06/23/深入理解Java虚拟机笔记0x02/)）
- `Class`对象仅在需要的时候才被加载
- `Class.forName(String className)`可以取得`Class`对象的引用，参数为一个包含目标类的文本名（区分大小写）。

