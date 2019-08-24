---
title: Java8实战笔记0x0a
date: 2019-08-24 10:08:36
categories: Java
tags:
  - Java
  - 笔记
  - 函数式
---

# 函数式编程的技巧

## 无处不在的函数

### 高阶函数

- 函数满足以下任一要求即可被称为告诫函数：
  - 接受至少一个函数作为参数
  - 返回结果是一个函数

```java
Comparator<Apple> c = comparing(Apple::getWeight);
```

### 科里化

- 科里化是指一个能将具有`n`个参数的函数转化为使用`n`个参数中部分参数的函数，该函数的返回值是另一个函数，参数为转化的函数未使用的参数，例如`f(x, y) =  (g(x))(y)`

```java
// 摄氏度转华氏度
static DoubleUnaryOperator curriedConverter(double f, double b) {
    return (double x) -> x * f + b;
}

DoubleUnaryOperator convertCtoF = curriedConverter(9.0/5, 32);
double c = convertCotF(25);
```

- 一个函数使用所有参数仅有部分被传递时，通常称这个函数是部分应用的。