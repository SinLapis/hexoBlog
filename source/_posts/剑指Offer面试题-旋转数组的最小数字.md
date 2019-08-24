---
title: 剑指Offer面试题-旋转数组的最小数字
date: 2019-08-24 17:14:46
categories: 算法
tags:
  - Java
  - 剑指Offer
  - 排序
  - 查找
---

# 旋转数组的最小数字

## 快速排序

```java
public class Q11_0 {
    public void quickSort(int[] arr, int start, int end) {
        if (start > end) return;
        int i = start, j = end;
        int t = arr[i];
        while (i < j) {
            while (i < j && t <= arr[j]) j--;
            if (i < j) {
                arr[i] = arr[j];
                i++;
            }
            while (i < j && t > arr[i]) i++;
            if (i < j) {
                arr[j] = arr[i];
                j--;
            }
        }
        arr[i] = t;
        quickSort(arr, start, i - 1);
        quickSort(arr, j + 1, end);
    }
}
```

## 旋转数组的最小数字

```java
public class Q11 {
    public int minNumberInRotateArray(int[] array) {
        if (array == null || array.length == 0) return 0;
        int left = 0, right = array.length - 1;
        int mid;
        while (left < right) {
            mid = left + right >> 1;
            if (array[left] < array[right]) return array[left];
            if (array[mid] > array[left]) left = mid + 1;
            else if (array[mid] < array[left]) right = mid;
            else left++;
        }
        return array[right];
    }
}
```

- 思路：和书上略有不同，主要是处理相同数字上。如果`arr[left]`和`arr[mid]`相等，那么只移动`left`即可。此外，要比较`arr[left]`和`arr[right]`的大小，否则会越过最小值。