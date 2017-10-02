---
title: FluentPython笔记0x07
date: 2017-09-15 19:28:22
categories: Python
tags:
  - Python
  - 笔记
---

## Mappings with Flexible Key Lookup（p70）

+ `defaultdict`：类似与`dict.default()`，可创建一个带有默认值类型的`dict`，例如`defaultdict(int)`，则遇到不存在的Key时，其取值为0。这个过程中会调用`default_factory`，进而会调用special method `__missing__`。

+ `__missing__`：该special method没有在`dict`中定义，但如果在`dict`的子类中定义`__missing__`，则`__getitem__`在遇到缺失的key时不会抛出`KeyError`，而是会转而调用`__missing__`。`__missing__`在其他时候不会被调用，例如`__contains__`。

+ 例3-7：

```python

class StrKeyDict0(dict):

    def __missing__(self, key):
        if isinstance(key, str):
            raise KeyError(key)
        return self[str(key)]

    def get(self, key, default=None):
        try:
            return self[key]
        except KeyError:
            return default

    def __contains__(self, key):
        return key in self.keys() or str(key) in self.keys()

```

有两处需要注意：
1.在`__missing__`中，`isinstance`的检查不可少。`__missing__`被调用时，情况有三种：（1）是`int`型，但其`str`型在`dict`中；（2）是`int`型，且`str`型不在`dict`中；（3）是`str`型但不在`dict`中。如果没有`isinstance`的过滤，`str`型也不在`dict`中时`self[str(key)]`还会调用`__missing__`，陷入死循环。
2.在`__contains__`中，没有使用`key in dict`而是用`key in dict.keys()`是因为前者依然会调用`__missing__`，而导致`int`型的`2`也会在`dict`的`keys`中。
