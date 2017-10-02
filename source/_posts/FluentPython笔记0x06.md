---
title: FluentPython笔记0x06
date: 2017-08-17 08:57:21
categories: Python
tags:
  - Python
  - 笔记
---

# Chapter 3
## Generic Mapping Types（p64)

+ 可哈希：如果一个对象在它的生命周期中有一个不会改变的哈希值（它需要有`__hash__()`函数），并且它能够和其他对象进行比较（需要有`__eq__()`函数）。可哈希的对象相等必须拥有相同的哈希值。`str`、`bytes`以及数类型都是可哈希的；`frozenset`也总是可哈希的，因为它的元素必须可哈希；`tuple`仅当它包含的所有元素可哈希是它才是可哈希的。例：

<!-- more -->

```python

tt = (1, 2, (30, 40))
print(hash(tt))
# 8027212646858338501
tl = (1, 2, [30, 40])
print(hash(tl))
# TypeError: unhashable type: 'list'
```

+ 建立`dict`：

```python

a = dict(one=1, two=2, three=3)
b = {'one': 1, 'two': 2, 'three': 3}
c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
d = dict({'three': 3, 'one': 1, 'two': 2})
print(a == b == c == d)
# True
```

## dict Comprehesions（p66）

+ 类似与listcomp，`dict`也有dictcomp。

```python

DIAL_CODES = [
  (86, 'China'),
  (91, 'India'),
  (1, 'United States'),
  (62, 'Indonesia'),
  (55, 'Brazil'),
  (92, 'Pakistan'),
  (880, 'Bangladesh'),
  (234, 'Nigeria'),
  (7, 'Russia'),
  (81, 'Japan'),
]
country_code = {country: code for code, country in DIAL_CODES}
print(repr(country_code))
# {'China': 86, 'India': 91, 'United States': 1, 'Indonesia': 62, 'Brazil': 55, 'Pakistan': 92, 'Bangladesh': 880, 'Nigeria': 234, 'Russia': 7, 'Japan': 81}
print(repr({code: country.upper() for country, code in country_code.items() if code < 66}))
# {1: 'UNITED STATES', 62: 'INDONESIA', 55: 'BRAZIL', 7: 'RUSSIA'}
```

+ 鸭子类型（duck typing，参考[维基百科](https://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B)）：“当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。”鸭子类型是动态类型的一种风格。在这种风格中，一个对象有效的语义，不是由继承自特定的类或实现特定的接口，而是由"当前方法和属性的集合"决定。

## Handling Missing Keys with setdefault（p68）

+ `d.get`和`d.setdefault`：`d.get(k, default)`是一种比较方便来处理`KeyError`的方法，在没有`k`值时会返回`default`。但是如果想更新字典中键值对使用`setdefault`会更加方便。

```python

import sys
import re

WORD_RE = re.compile('\w+')


def dict_get(fp):
    index = {}
    for line_no, line in enumerate(fp, 1):
        for match in WORD_RE.finditer(line):
            word = match.group()
            column_no = match.start() + 1
            location = (line_no, column_no)
            occurrences = index.get(word, [])
            occurrences.append(location)
            index[word] = occurrences
    for word in sorted(index, key=str.upper):
        print(word, index[word])


def dict_setdefault(fp):
    index = {}
    for line_no, line in enumerate(fp, 1):
        for match in WORD_RE.finditer(line):
            word = match.group()
            column_no = match.start() + 1
            location = (line_no, column_no)
            index.setdefault(word, []).append(location)
    for word in sorted(index, key=str.upper):
        print(word, index[word])


if __name__ == '__main__':
    with open(sys.argv[1], encoding='utf-8') as fp:
        dict_get(fp)
        dict_setdefault(fp)

```

对比`dict_get`和`dict_setdefault`两种方法可发现`setdefault`更加方便。

+ 有关于字典排序：对于字典`d`，如果希望对其进行按`key`排序，可以直接使用`sorted(d)`，如果希望对其进行值的排序，则可以：

```python

for key in sorted(d, key=d.get):
  print(key, d[key])
```

+ 关于正则表达式匹配的几个函数`match`、`search`、`findall`、`finditer`：`match`成功匹配会返回`Match`对象，失败会返回`None`；`search`类似`match`，但只返回第一个匹配字符串；`findall`返回为数组形式的结果；`finditer`返回迭代器形式的结果。
