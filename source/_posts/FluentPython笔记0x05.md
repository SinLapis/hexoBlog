---
title: FluentPython笔记0x05
date: 2017-08-02 09:22:03
categories: Python
tags:
  - Python
  - 笔记
---
## NumPy and SciPy（p52）

+ `numpy.ndarray`的基础操作：

<!-- more -->

```python

import numpy

a = numpy.arange(12)
# array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11])
# type(a): numpy.ndarray
# a.shape: (12,)
a.shape = 3, 4
'''
array([[0, 1, 2, 3],
       [4, 5, 6, 7],
       [8, 9 ,10, 11]])
'''
print(a[2])
# 8, 9, 10, 11
print(a[2, 1])
# 9
print([:, 1])
# 1, 5, 9
a.transpose()
'''
array([[0, 4, 8],
       [1, 5, 9],
       [2, 6, 10],
       [3, 7, 11]])
'''
```

## Deques and Other Queues（p55）

+ `deque`：如果把`list`当做栈使用性能上十分低下，此时应该使用`deque`代替。

```python

from collections import deque


dq = deque(range(10), maxlen=10)
# deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]， maxlen=10)
dq.rotate(3)
# deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)
# rotate(n)当n>0时将n项从最右移到最左，n<0反之。
dp.appendleft(-1)
# deque([-1, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
# deque会保持长度，最右侧的0被移除
dq.extend([11, 22, 33])
# deque([3, 4, 5, 6, 7, 8, 9, 11, 22, 33], maxlen=10)
dq.extendleft([10, 20, 30, 40])
# deque([40, 30, 20, 10, 3, 4, 5, 6, 7, 8], maxlen=10)
# 从左加入列表中的数据时会反转
```

在`deque`的方法中，注意`remove`的实现是严格按照数据从序列两端进出的，故性能并不佳；`append`和`popleft`是原子操作，在LIFO和多线程操作的情况下不需要锁。
