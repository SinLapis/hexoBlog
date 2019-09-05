---
title: 剑指Offer面试题-正则表达式匹配
date: 2019-09-05 16:06:06
categories: 算法
tags:
  - Java
  - 剑指Offer
  - 字符串
---

# 正则表达式匹配

请实现一个函数用来匹配包括`.`和`*`的正则表达式。模式中的字符`.`表示任意一个字符，而`*`表示它前面的字符可以出现任意次（包含0次）。 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串`aaa`与模式`a.a`和`ab*ac*a`匹配，但是与`aa.a`和`ab*a`均不匹配

```java
public class Q19 {
    public boolean match(char[] str, char[] pattern) {
        if (pattern == null || str == null) return false;
        return subMatch(str, 0, pattern, 0);
    }
    private boolean subMatch(char[] str, int strIndex, char[] pattern, int patternIndex) {
        // [1]
        if (strIndex >= str.length) {
            return patternIndex >= pattern.length ||
                    (patternIndex == pattern.length - 2 && pattern[patternIndex + 1] == '*');
        }
        if (patternIndex >= pattern.length) return false;
        // 匹配单个字符
        if (patternIndex == pattern.length - 1 || pattern[patternIndex + 1] != '*') {
            if (pattern[patternIndex] == '.' || str[strIndex] == pattern[patternIndex]) {
                return subMatch(str, strIndex + 1, pattern, patternIndex + 1);
            }
            return false;
        }
        // 匹配带*
        if (pattern[patternIndex] != '.' && str[strIndex] != pattern[patternIndex]) {
            return subMatch(str, strIndex, pattern, patternIndex + 2);
        }
        //[2]
        if (subMatch(str, strIndex, pattern, patternIndex + 2)) return true;
        //[3]
        while (strIndex < str.length) {
            if (str[strIndex++] != pattern[patternIndex] && pattern[patternIndex] != '.') {
                break;
            }
            if (subMatch(str, strIndex, pattern, patternIndex + 2)) return true;
        }
        return false;
    }

    public static void main(String[] args) {
        Q19 s = new Q19();
        String str = "bcbbabab";
        String pattern = ".*a*a";
        s.match(str.toCharArray(), pattern.toCharArray());
    }
}
```

- 思路：使用递归。首先是退出条件，如果`str`和`pattern`都用完退出返回`true`，如果仅是`pattern`用完则返回`false`。注意[1]处有另一种退出返回`true`的情况，就是`str`用完，`pattern`剩余一个字符和`*`，此时按匹配0个可以返回`true`。然后是匹配单个字符，比较简单。最后是匹配带`*`，注意[2]先进行一次匹配0个，[3]进入循环的条件为`str`没有用完。

