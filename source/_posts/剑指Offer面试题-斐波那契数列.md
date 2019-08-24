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

- 思路：动态规划，需要前两个结果。

## 跳台阶

一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。

```JAVA
public class Q10_2 {
    public int JumpFloor(int target) {
        if (target < 2) return 1;
        if (target == 2) return 2;
        int b1 = 2, b2 = 1;
        int current = 1, i = 3;
        while (i <= target) {
            current = b1 + b2;
            b2 = b1;
            b1 = current;
            i++;
        }
        return current;
    }
}
```

- 思路：同上。

## 变态跳台阶

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

- 思路：这个是数学题吧？可以把问题转化为青蛙可以以`1-n`次跳上台阶，求各种次数跳法之和。可以是往`n`个台阶之间插入挡板，有`n-1`个空，可插入`0~n-1`个挡板，则共有`C(n-1, 0) + C(n-1, 1) + C(n-1, 2) + ... +C(n-1, n-1) = 2^(n-1)`种方法。