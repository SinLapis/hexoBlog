---
title: Fluent Python笔记0x01
date: 2017-07-19 14:34:10
categories: Python
tags:
  - Python
  - 笔记
---
划了两周水......
## p8
+ special methods是由Python解释器调用，不需要你来调用（虽然也可以亲自调用）。
+ 对于内置类型如`list`，`str`，`bytearray`等来说，调用`len()`则是由CPython直接返回`PyVarObject`（C的struct）中关于长度的变量，这比调用`__len__`要快很多。

<!-- more -->

## p10
+ 例子1-2：

```python

from math import hypot


class Vector:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Vector(%r, %r)' % (self.x, self.y)

    def __abs__(self):
        return hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))

    def __add__(self, other):
        x = self.x + other.x
        y = self.y + other.y
        return Vector(x, y)

    def __mul__(self, other):
        return Vector(self.x * other, self.y * other)
```
## p11
+ 关于`__str__`和`__repr__`的区别（参考：[StackOverflow](https://stackoverflow.com/questions/1436703/difference-between-str-and-repr-in-python)）

简单来说`__repr__`是为了保证无歧义，而`__str__`是为了提供可读性。在Python提供的类中，使用`__repr__`返回的结果都是格式化的数据，是符合Python语法的；`__str__`则是面向一般用户，提供可读性，但不保证无歧义，例如int型的3和str的3满足：

```python
str(3) == str('3')
```
## p12
+ 关于运算符：例1-2中声明的`__add__`和`__mul__`是针对`+`和`*`的重载。注意这里乘法参数顺序必须为向量在前数在后。
+ 布尔值：在Python中，任何对象都有布尔值。如果对象未声明`__bool__`或`__len__`，则其布尔值恒为`True`。如果声明了`__bool__`则`bool()`会使用其返回值。如果没有声明`__bool__`而声明了`__len__`则会使用`len()`的返回值，为0则为`False`，否则为`True`。
+ 有关例1-2中`Vector.__bool__`一个更加便捷的声明：

```python

def __bool__(self):
    return bool(self.x or self.y)
```

这样可以避免使用`abs()`。
