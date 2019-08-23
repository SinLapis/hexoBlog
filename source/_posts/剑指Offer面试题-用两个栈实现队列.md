---
title: 剑指Offer面试题-用两个栈实现队列
date: 2019-08-23 16:26:27
categories: 数据结构
tags:
  - Java
  - 剑指Offer
  - 栈
  - 队列
---

# 用两个栈实现队列

用两个栈来实现一个队列，完成队列的`Push`和`Pop`操作。 队列中的元素为`int`类型。

```java
public class Q9 {
    Stack<Integer> stack1 = new Stack<>();
    Stack<Integer> stack2 = new Stack<>();

    public void push(int node) {
        stack1.push(node);
    }

    private void fillStack2() {
        while (!stack1.empty()) {
            stack2.push(stack1.pop());
        }
    }
    public int pop() {
        if (stack2.empty()) {
            fillStack2();
        }
        return stack2.pop();
    }
}

```

- 思路：一个栈用来入队，另一个栈用来出队。如果出队的栈为空，需要把入队的栈的值导入出队的栈。