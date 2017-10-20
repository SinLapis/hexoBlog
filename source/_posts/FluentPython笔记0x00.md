---
title: Fluent Python笔记0x00
date: 2017-07-05 15:28:36
categories: Python
tags:
  - Python
  - 笔记
---
鬼使神差地买了一本英文的，想想还是写个笔记。版本为东南大学的影印版，原版\*\*\*贵。
# Chapter 1
## p4
+ Python中类似于`__getitem__`这种方法称为special method，作者称其为“dunder-getitem”。此外，类似`__x`是有其他含义的。

<!--more-->

## p5
+ collections: 包含一些数据类型，例如namedtuple等。

参考: http://www.zlovezl.cn/articles/collections-in-python/
+ collections.namedtuple(): 构建一个可以用名称取值的tuple，例如：

```python

Point = collections.namedtuple('Point', ['x', 'y'])
point = Point(1, 2)
print(point.x)

```
参考: http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001411031239400f7181f65f33a4623bc42276a605debf6000
+ 例子1-1：

```python

import collections


Card = collections.namedtuple('Card', ['rank', 'suit'])

class FrenchDeck:
    ranks = [str(n) for n in range(2, 11)] + list('JQKA')
    suits = 'spades diamonds clubs hearts'.split()

    def __init__(self):
        self._cards = [Card(rank, suit) for suit in self.suits for rank in self.ranks]

    def __len__(self):
        return len(self._cards)

    def __getitem__(self, position):
        return self._cards[position]

```

其中，使用collections.namedtuple建立了一张扑克牌的模型，FrenchDeck则建立了所有的扑克牌。
此时在Python shell中输入：

```
>>> deck[3]
```

会得到如下结果：

```
Card(rank='K', suit='hearts')
```

这是由`__getitem__`方法提供的。
+ 对于Python数据模型来说，special method 的优点：
	1. 统一标准操作（例如，用户不需要记住每种数据模型对应的获取长度的方法是`.size()`还是`.length()`）。
	2. 通过提供统一的special method，可以丰富Python标准库（例如上述的random.rich），也可以支持Python中包含的运算符操作（例如FrenchDeck支持迭代，也支持[]操作符）。
+ 使用`in`操作时，如果数据模型没有实现`__contains__`方法，Python会进行顺序扫描。
+ 目前的`_cards`中扑克牌序列是不能被打乱的，在Chapter 11中会介绍`__setitem__`方法来实现。
