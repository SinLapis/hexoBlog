---
title: FluentPython笔记0x03
date: 2017-07-27 10:10:41
categories: Python
tags:
  - Python
  - 笔记
---

唔，如果把这本书涉及到但是没提到的点拓展开，我什么时候能读完这本书呢？

## p26

+ 元组不仅仅是不可变的列表，还可以存储记录。


<!-- more  -->

## p27

```python

lax_coordinates = (33.9425, -118.408056)
# 洛杉矶国际机场的经纬度
city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014)
# 有关东京的数据：名称，年份，人口（百万），人口变化，面积（平方公里）
traveler_ids = [('USA', '31195855'), ('BRA', 'CE342567'), ('ESP', 'XDA205856')]
for passport in traveler_ids:
  print('%s/%s' % passport)
# 上面对元组的操作称为元组解包
for country, _ in traveler_ids:
  print(country)
# 使用 _ 作为占位符，可以只获取元组中需要的字段
b, a = a, b
# 一个优雅的交换两个变量值的操作
print(divmod(20, 8))

t = (20, 8)
print(divmod(*t))
# 使用 * 来解包元组传参
```

## p29

+ `*`还可以获取多余的项。

```python

a, b, *rest = range(5)
print(repr((a, b, rest)))
# 结果：(0, 1, [2, 3, 4])
a, b, *rest = range(3)
print(repr((a, b, rest)))
# 结果：(0, 1, [2])
a, b, *rest = range(2)
print(repr((a, b, rest)))
# 结果：(0, 1, [])
a, *body, c, d = range(5)
print(repr((a, body, c, d)))
# 结果：(0, [1, 2], 3, 4)
*head, b, c, d = range(5)
print(repr((head, b, c, d)))
# 结果：([0, 1], 2, 3, 4)
```

可见前缀`*`可以出现在任意位置。
+ 嵌套元组解包：

```python

metro_areas = [
    ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
    ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
    ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
    ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
    ('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
]
print('{:15} | {:^9} | {:^9}'.format('', 'lat.', 'long.'))
fmt = '{:15} | {:9.4f} | {:9.4f}'
for name, cc, pop, (latitude, longitude) in metro_areas:
    if longitude <= 0:
        print(fmt.format(name, latitude, longitude))

'''结果：
                |   lat.    |   long.  
Mexico City     |   19.4333 |  -99.1333
New York-Newark |   40.8086 |  -74.0204
Sao Paulo       |  -23.5478 |  -46.6358
'''
```

+ 格式字符串`str.format`（参考[blog.xiayf.cn](http://blog.xiayf.cn/2013/01/26/python-string-format/)）: 其实`%`这种格式输出已经很老了，`str.format`可以更加方便的格式输出以及提供更多功能。下表为一些基础的数格式输出。

<style>
table th, table td {
  text-align: center;
  padding: 5px;
}
</style>

|  数  |  格式  |  输出  |  描述  |
|  --  |  ----  |  ----  |  ----  |
|  3.1415926  |  {:.2f}  |  3.14  |  保留小数点后两位  |
|  3.1415926  |  {:+.2f}  |  +3.14  |  带符号保留小数后两位  |
|  -1  |  {:+.2f}  |  -1.00  |  带符号保留小数后两位  |
|  2.71828  |  {:.0f}  |  3  |  不带小数  |
|  5  |  {:0>2d}  |  05  |  数左补0，宽度到2  |
|  5  |  {:x<4d}  |  5xxx  |  数右补x，宽度到4  |
|  10  |  {:x<4d}  |  10xx  |  数右补x，宽度到4  |
|  1000000  |  {:,}  |  1,000,000  |  使用逗号分隔  |
|  0.25  |  {:.2%}  |  25.00%  |  百分比格式，两位小数  |
|  1000000000  |  {:.2e}  |  1.00e+09  |  科学计数法  |
|  13  |  {:10d}  |  ······13  |  右对齐，默认，宽度为10  |
|  13  |  {:<10d}  |  13······  |  左对齐，宽度为10  |
|  13  |  {:^10d}  |  ···13···  |  居中，宽度为10  |

有关于`str.format`的用法：

```python

s1 = "so much depends upon {}".format("a red wheel barrow")
s2 = "glazed with {} water beside the {} chickens".format("rain", "white")
```

这里`{}`即充当占位符，和`%s`的功能类似，会将`format`的参数依次填入。当然你也可以指定顺序填入，见下。

```python

s1 = " {0} is better than {1} ".format("emacs", "vim")
s2 = " {1} is better than {0} ".format("emacs", "vim")
```

此外，这样能够标记序号的好处还有防止漏掉变量，见下。

```python

set =  '(%s, %s, %s, %s, %s, %s, %s, %s)' % (a, b, c, d, e, f, g, h, i) # i是多余的
set = '({0}, {1}, {2}, {3}, {4}, {5}, {6}, {7})'.format(a, b, c, d, e, f, g)
```

`format`可以为占位符命名，这样就不需要严格的顺序了。

```python

madlib = " I {verb} the {object} off the {place} ".format(verb="took", object="cheese", place="table")
```

可以重复使用同一变量。

```python

str = "Oh {0}, {0}! wherefore art thou {0}?".format("Romeo")
```

可以转换进制。

```python

print("{0:d} - {0:x} - {0:o} - {0:b} ".format(21))
```

可以把`format`当做方法使用。

```python

# 定义格式
email_f = "Your email address was {email}".format

# 在另一个地方使用
print(email_f(email="bob@example.com"))
```

如果你需要显示大括号，只要写两次就可以了。

```python

print(" The {} set is often represented as { {0} } ".format("empty"))
```

## p30

+ 含名称元组`collections.namedtuple`：在前文中提到过。注意使用`namedtuple`所需的内存是小于使用`class`的，因为`namedtuple`不会把参数存到完整实例`__dict__`中。

```python

from collections import namedtuple
City = namedtuple('City', 'name country population coordinates')
tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
```

在创建一个`namedtuple`时，既可以像之前一样使用列表包含属性，也可以使用空格分隔的字符串。

## p31

+ 含名称元组的参数和方法：

```python

print(repr(City._fields))
# ('name', 'country', 'population', 'coordinates')

LatLong = namedtuple('LatLong', 'lat long')
delhi_data = ('Delhi NCR', 'IN', 21.935, LatLong(28.613889, 77.208889))
delhi = City._make(delhi_data)
print(repr(delhi._asdict))
# OrderedDict([('name', 'Delhi NCR'), ('country', 'IN'), ('population', 21.935), ('coordinates', LatLong(lat=28.613889, long=77.208889))])
for key, value in delhi._asdict().items():
  print(key + ':', value)
'''
name: Delhi NCR
country: IN
population: 21.935
coordinates: LatLong(lat=28.613889, long=77.208889)
'''
```

其中`City._make(delhi_data)`与`City(*delhi_data)`等价。
