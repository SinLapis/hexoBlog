---
title: 剑指Offer面试题-二维数组中的查找
date: 2019-08-20 17:11:20
categories: 算法
tags:
  - 算法
  - Java
  - 剑指Offer
  - 数组
---

# 二维数组中的查找

在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

```java
public class Q4 {
    public static boolean solution(int[][] arr, int target) {
        if (arr == null || arr.length == 0 || arr[0] == null || arr[0]. length == 0)
            return false;
        if (arr[0][0] > target || arr[arr.length - 1][arr[0].length - 1] < target) return false;
        int column = arr[0].length - 1, row = 0;
        while (column >= 0 && row < arr.length) {
            if (arr[row][column] == target) return true;
            else if (arr[row][column] > target) column--;
            else row++;
        }
        return false;
    }
}
// Test
class Q4Test {

    @Test
    void solution() {
        int[][] arr = {
                {1, 2, 8, 9},
                {2, 4, 9, 12},
                {4, 7, 10, 13},
                {6, 8, 11, 15}
        };
        assertTrue(Q4.solution(arr, 7));
        assertTrue(Q4.solution(arr, 11));
        assertTrue(Q4.solution(arr, 4));
        assertTrue(Q4.solution(arr, 1));
        assertTrue(Q4.solution(arr, 15));
        assertFalse(Q4.solution(arr, 0));
        assertFalse(Q4.solution(arr, 20));
        assertFalse(Q4.solution(arr, 3));
        assertFalse(Q4.solution(null, 0));
        assertFalse(Q4.solution(new int[0][0], 5));
        assertFalse(Q4.solution(new int[1][0], 2));
    }
}
```

![在二维数组中查找](/images/2019-08-22_154547.png)

- 思路：从右上角开始，如图，由于`9`大于`7`，那么第四列所有的数都大于`7`，因此可以排除第四列。以此类推。如果当前值小于目标值，例如第二列的`2`小于`7`，那么`2`右面的所有数都小于`7`，因此可以排除第二行。

