---
title: 剑指Offer面试题-斐波那契数列
date: 2019-08-23 17:19:04
categories: 算法
tags:
  - Java
  - 剑指Offer
  - 动态规划
---

# 斐波那契数列

要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0）。

```java
public class Q10 {
    public int Fibonacci(int n) {
        if (n < 0) return -1;
        if (n <= 1) return n;
        int current = 1, b1 = 1, b2 = 0;
        int i = 2;
        while (i <= n) {
            current = b1 + b2;
            b2 = b1;
            b1 = current;
            i++;
        }
        return current;
    }
}
```

