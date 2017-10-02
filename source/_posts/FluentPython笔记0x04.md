---
title: FluentPython笔记0x04
date: 2017-08-01 09:09:19
categories: Python
tags:
  - Python
  - 笔记
---
## p32

+ 和列表相比，元组除了无法修改其中的值以外，只有一处和列表不同：元组无法使用`__reversed__`，但是可以使用以下方法代替：

<!-- more -->

```python

t = (1, 2, 3, 4)
print(tuple(reversed(t)))
```

+ 元组和列表可以与`int`型整数相乘，含义为返回对应的元组或列表，里边包括重复整数次的原来的元组或列表的内容。

## p33

+ 像元组、列表等序列切片时为什么排除结束标号的项：1.可以简单的看出切片得到的长度，例`range(3)`以及`my_list[:3]`；2.可以简单地计算切片长度，只需结束序号减去开始序号；3.可以只使用一个序号就能把序列切成两部分，例如`my_list[:x]`和`my_list[x:]`。

+ 切片的实现是一个切片类：`slice(start, stop, step)`。例：

```python

source = '''
0.......8....12
0 AXL         5
1 August      6
2 key        10
'''
INDEX = slice(0, 2)
NAME = slice(2, 8)
POINT = slice(8, None)
line_items = source.split('\n')[2:]
for item in line_items:
  print(item[POINT], item[NAME])
```

+ 切片赋值操作：

```python

l = list(range(10))
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
l[2:5] = [20, 30]
# [0, 1, 20, 30, 5, 6, 7, 8, 9]
del l[5:7]
# [0, 1, 20, 30, 5, 8, 9]
l[3::2] = [11, 22]
# [0, 1, 20, 11, 5, 22, 9]
l[2:5] = 100
# 错误，左面是列表右面对应应该也是列表等可迭代对象
l[2:5] = [100]
# [0, 1, 100, 22, 9]
```

## p40

+ 有关修改元组中列表的问题：代码如下。

```python

t = (1, 2, [30, 40])
t[2] += [50, 60]
```

终端报错如下

```
TypeError: 'tuple' object does not support item assignment
```

但是此时打印`t`会显示
```python
(1, 2, [30, 40, 50, 60])
```

根据Python生成的字节码，元组`+=`这个操作的关键步骤为：1.将`t[2]`的值压栈（位于栈顶）；2.执行`栈顶值 += [50, 60]`，如果栈顶值不为可变变量的话会报错。但此处栈顶为`list`，故成功执行；3.执行`t[2] = 栈顶值`，如果`t`为不可变变量则会报错，此处`t`为元组故会出现报错。综上最终结果就是`list`的值改变了但是出现了报错。

总结：1.不建议在元组中填入可变项；2.自增不是原子操作，错误抛出可以在自增中的任何一个环节独立抛出，而不影响前面已执行的部分；3.如果遇到类似问题可以尝试阅读Python字节码，它并不难懂。

## p42

+ `list.sort`和`sorted`：`list.sort`只能用于可变序列，会改变列表，返回值为`None`。`sorted`可以用于任何序列，它总是会返回一个新的序列而不改变原序列。

## p44

+ 使用`bisect`进行有序序列的查找和插入：`bisect`即二分查找。

```python

import bisect
import sys


HAYSTACK = [1, 4, 5, 6, 8, 12, 15, 20, 21, 23, 23, 26, 29, 30]
NEEDLES = [0, 1, 2, 5, 8, 10, 22, 23, 29, 30, 31]

ROW_FMT = '{0:2d} @ {1:2d}     {2}{0:2d}'


def demo(bisect_fn):
    for needle in reversed(NEEDLES):
        position = bisect_fn(HAYSTACK, needle)
        offset = position * '  |'
        print(ROW_FMT.format(needle, position, offset))


if __name__ == '__main__':
    if sys.argv[-1] == 'left':
        bisect_fn = bisect.bisect_left
    else:
        bisect_fn = bisect.bisect
    print('DEMO:', bisect_fn.__name__)
    print('haystack ->', ' '.join('%2d' % n for n in HAYSTACK))
    demo(bisect_fn)
```

其中`bisect.bisect`和`bisect.bisect_right`是等价的，它们和`bisect.bisect_left`的区别在于遇到相等的值是插入的位置不同。一般情况下看不出区别，但是像`1`与`1.0`这样的还是有所区别的。

`bisect.bisect`有个有趣的应用：给连续的数值分级。

```python

def grade(score, breakpoints=[60, 70, 80, 90], grades='FDCBA'):
  i = bisect.bisect(breakpoints, score)
  return grades[i]


print([grade(score) for score in [33, 99, 77, 70, 89, 90]])
# FACCBAA
```

此外，`bisect.insort`是向有序序列中插入值而不破坏有序，和`bisect.bisect`类似。

## p48

+ 如果对序列有特殊需求，`list`不一定是最好的选择。例如，如果你想保存10M个浮点数，`array`比`list`更加有效，因为`array`没有保存每一个`float`对象的全部。另外如果需要实现类似队列或者栈则可以使用`deque`。

+ `array`需要先指定类型，即`array`只能存储一种类型的数据，但是这换来的是极大的性能提升。`array.fromfile`和`array.tofile`分别可以读取文件创建序列和将序列写入文件，这两种操作要比读写文本文件加上解析转换要快很多。以下为`array`支持的初始化类型：

<style>
table th, table td {
  text-align: center;
  padding: 5px;
}
</style>

|  类型码  |  C类型  |  Python类型  |  最小占用字节  |
|  ------  |  -----  |  ----------  |  ------------  |
|  c  |  char  |  character  |  1  |
|  b  |  signed char  |  int  |  1  |
|  B  |  unsigned char  |  int  |  1  |
|  u  |  Py_UNICODE  |  Unicode character  |  2  |
|  h  |  signed short  |  int  |  2  |
|  H  |  unsigned short  |  int  |  2  |
|  i  |  signed int  |  int  |  2  |
|  I  |  unsigned int  |  long  |  2  |
|  l  |  signed long  |  int  |  4  |
|  L  |  unsigned long  |  long  |  4  |
|  f  |  float  |  float  |  4  |
|  d  |  double  |  float  |  8  |

## p51

+ `memorview`：内存查看对象，能够使用支持缓冲区协议的数据类型进行包装，在不需要复制的情况下用Python代码访问以及等大小修改。
