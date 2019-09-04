---
title: 剑指Offer面试题-数值的整数次方
date: 2019-09-02 16:39:01
categories: 算法
tags:
  - Java
  - 剑指Offer
  - 数学
---

# 数值的整数次方

给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。保证base和exponent不同时为0。

```java
class Solution {
    private double unsignedPower(double base, int exponent) {
        if (exponent == 0) return 1;
        if (exponent == 1) return base;
        double result = unsignedPower(base, exponent >> 1);
        result *= result;
        if (exponent & 1 == 1) result *= base;
        return result;
    }
    public double Power(double base, int exponent) {
        if (base == 0.0) return 0.0;
        if (exponent == 0) return 1.0;
        int exp = Math.abs(exponent);
        double result = unsignedPower(base, exp);
        if (exponent < 0) result = 1 / result;
        return result;
    }
}
```

- 思路：求`a^n`，先求`a^(n/2)`，然后平方，依次递归。注意指数为负数、0以及底数为0.0的情况。