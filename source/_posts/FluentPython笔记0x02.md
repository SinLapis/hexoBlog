---
title: FluentPython笔记0x02
date: 2017-07-26 08:57:06
categories: Python
tags:
  - Python
  - 笔记
---
嗯，记点单词。
## p14
+ augmented assignment: 增量赋值，例`i += 1`。

<!-- more -->

+ aspect-oriented programming: 面向侧面编程（AOP），又称面向方面编程，是对面向对象编程的补充。侧面是一种新的模块化机制，用来描述分散在对象、类或函数中的横切关注点，即关键的逻辑部分。例如Python的装饰器，某装饰器可以包含执行某一过程前后的日志记录，那么这种编程方法即可称为面向侧面编程。

# Chapter 2
## p21
+ list生成：比较两种方法的可读性。

```python

symbols = 'abcdef'
codes = []
for symbol in symbols:
  codes.append(ord(symbol))
print(codes)
```


```python

symbols = 'abcdef'
codes = [ord(symbol) for symbol in symbols]
print(codes)
```

for循环的功能有很多，诸如扫描队列并取出元素、计算总和等等，但是listcomp(list comprehension, 列表推导式)的功能就只有生成一个新列表。从这个角度来看后者可读性更高。
+ 在Python 2中，如果列表推导式、生成器表达式的作用域中的变量名和外面有重复的话是会产生冲突的。虽然Python 3解决了这一问题，但是在写代码时还是避免这种情况。

## p23

+ Cartesian product: 笛卡尔积。下面的代码为推荐写法。

```python

colors = ['black', 'white']
sizes = ['S', 'M', 'L']
tshirts = [(color, size) for color in colors
                         for size in sizes]
print(tshirts)
```

## p25

+ generator expression: 生成器表达式。

```python

symbols = 'abcdef'
print(tuple(ord(symbol) for symbol in symbols))

import array
print(array.array('I', (ord(symbol) for symbol in symbols)))
```

当生成器表达式单独作为参数时不需要加括号。但像在`array`中接受两个参数时，生成器表达式需要加上圆括号。上述的T恤的代码用生成器表达式改写：

```python

colors = ['black', 'white']
sizes = ['S', 'M', 'L']
for tshirt in ('%s %s' % (c, s) for c in colors for s in sizes):
  print(tshirt)
```

注意生成器的结果不会全部保存在内存中，它一次只会给`for`循环一个结果。
