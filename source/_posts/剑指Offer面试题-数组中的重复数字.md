---
title: 剑指Offer面试题-数组中的重复数字
date: 2019-08-19 15:20:05
categories: 算法
tags:
  - 算法
  - Java
  - 剑指Offer
  - 数组
---

# 数组中的重复数字

## 找出数组中重复的数字

在一个长度为`n`的数组里的所有数字都在`0`到`n-1`的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为`7`的数组`{2, 3, 1, 0, 2, 5, 3}`，那么对应的输出是第一个重复的数字`2`。

```java
public class Q3 {
    public static int solution(int[] arr) {
        int i = 0;
        if (arr == null) return -1;
        for (int value : arr) {
            if (value < 0 || value > arr.length - 1) return -1;
        }
        while (i < arr.length) {
            if (arr[i] != i) {
                if (arr[i] == arr[arr[i]]) return arr[i];
                int t = arr[i];
                arr[i] = arr[arr[i]];
                arr[t] = t;
            } else {
                i++;
            }
        }
        return -1;
    }
}
// Test
class Q3Test {

    @Test
    void solution() {
        assertEquals(2, Q3.solution(new int[]{2, 3, 1, 0, 2, 5, 3}));
        assertEquals(-1, Q3.solution(new int[0]));
        assertEquals(-1, Q3.solution(null));
        assertEquals(-1, Q3.solution(new int[]{0, 1, 2, 3}));
        assertEquals(-1, Q3.solution(new int[]{1, 2, 3}));
    }
}
```

- 思路：从`0`号位开始，将值和下标对应起来，例如`arr[0] == 2`，那么就看`arr[2]`是不是`2`，是则`2`是其中一个重复数字，否则交换。如果当前位置下标和当前值相等就向后移动。
- 时间复杂度`O(n)`，空间复杂度`O(1)`。

## 不修改数组找出重复的数字

在一个长度为`n + 1`的数组里的所有数字都在`1`到`n`的范围内。 数组中至少有一个数字是重复的，。请找出数组中任意一个重复的数字，但不修改数组。 例如，如果输入长度为`8`的数组`{2, 3, 5, 4, 3, 2, 6, 7}`，那么对应的输出是重复的数字`2`或`3`。

```java
public class Q3_2 {
    public static int solution(int[] arr) {
        if (arr == null) return  0;
        int left = 1;
        int right = arr.length - 1;
        int big = 0, eq = 0, small = 0;
        int mid;
        while (left < right) {
            mid = left + (right - left >> 1);
            for (int value : arr) {
                if (value >= left && value <= right) {
                    if (value == mid) eq++;
                    else if (value < mid) small++;
                    else big++;
                }
            }
            if (eq > 1) return mid;
            if (small > big) right = mid;
            else left = mid + 1;
        }
        return 0;
    }
}
// Test
class Q3_2Test {

    @Test
    void solution() {
        int[] arr = {2, 3, 5, 4, 3, 2, 6, 7};
        assertThat(Q3_2.solution(arr), anyOf(equalTo(2), equalTo(3)));
        assertArrayEquals(new int[]{2, 3, 5, 4, 3, 2, 6, 7}, arr);
        // 无重复
        arr = new int[]{0, 1, 2};
        assertEquals(Q3_2.solution(arr), 0);
        // null
        assertEquals(Q3_2.solution(null), 0);
    }
}
```

- 思路：针对值范围进行二分查找。例如`1-n`，中间值为`mid`，先遍历数组，分别找出小于`mid`的个数`small`、等于`mid`的个数`eq`、大于`mid`的个数`big`。如果`eq`大于`1`，那说明为`mid`的元素有两个，即找到结果。否则，看`small`和`big`哪一个更大，则重复值在哪边。
- 时间复杂度`O(nlogn)`，空间复杂度`O(1)`