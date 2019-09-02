---
title: 剑指Offer面试题-剪绳子
date: 2019-09-02 16:14:20
categories: 算法
tags:
  - Java
  - 剑指Offer
  - 动态规划
---

# 剪绳子

一根长度为`n`的绳子，请把绳子剪成`m`段（`n > 1`且`m > 1`），每段绳子的长度记为`k[0], k[1], ..., k[m]`。请问`k[0] * k[1] * , ..., * k[m]`的可能最大乘积是多少？

```java
class Solution {
    public int maxMulti(int n) {
        if (n < 2) return 0;
        if (n < 4) return n - 1;
        int[] back = new int[n + 1]{0, 1, 2, 3};
        int max = 0;
        for (int i = 4; i <= n; i++) {
            max = 0;
            for (int j = 1; j <= i / 2; j++) {
                int temp = back[j] * back[i - j];
                if (temp > max) max= temp;
            }
            back[i] = max;
        }
        return back[n];
    }
}
```

- 思路：使用动态规划，记录长度为`i`时乘积最大值。注意在取`[0, 3]`时分情况，因为题目要求至少剪一刀，所以在`n`为`[0, 3]`时的返回值和记录初始化时的`[0, 3]`不同。