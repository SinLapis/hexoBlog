---
title: 剑指Offer面试题-打印从1到最大的n位数
date: 2019-09-04 15:45:18
categories: 算法
tags:
  - Java
  - 剑指Offer
  - 大数
---

# 打印从1到最大的n位数

输入数字n，按顺序打印从1到最大的n位数十进制数。

```java
class Solution {
    StringBuilder num = new StringBuilder();
    int lastCount = 1;

    public Solution() {
        num.append('1');
    }

    public void check() {
        boolean c = true;
        for (int i = num.length() - 1; i >= 0; i--) {
            if (c) {
                if (num.charAt(i) == '9') num.setCharAt(i, '0');
                else {
                    num.setCharAt(i, (char) (num.charAt(i) + 1));
                    c = false;
                }
            } else break;
        }
        if (c) num.insert(0, '1');
    }

    public void increase() {
        lastCount++;
        if (lastCount >= 10) {
            lastCount = 0;
            check();
        } else {
            num.setCharAt(num.length() - 1, (char) (num.charAt(num.length() - 1) + 1));
        }
    }

    public int getSize() {
        return num.length();
    }

    public void print() {
        System.out.println(num.toString());
    }

    public static void solute(int n) {
        if (n <= 0) return;
        Solution solution = new Solution();
        while (solution.getSize() <= n) {
            solution.print();
            solution.increase();
        }
    }
}
```

- 思路：使用书中第一种解法。使用`StringBuilder`更加方便。

