---
title: 剑指Offer面试题-从尾到头打印链表
date: 2019-08-22 15:58:56
categories: 数据结构
tags:
  - Java
  - 剑指Offer
  - 链表
---

# 从尾到头打印链表

输入一个链表的头节点，从尾到头反过来打印出每个节点的值。链表定义如下：

```java
class Node {
    public int value;
    public Node next;
}
```

```java
class Node {
    public int value;
    public Node next;

    public Node(int value, Node next) {
        this.value = value;
        this.next = next;
    }
}
public class Q6 {
    public static void solution(Node head) {
        if (head != null) {
            if (head.next != null)
                solution(head.next);
            System.out.print(head.value + " ");
        }
    }
}
// Test
class Q6Test{
    private static final ByteArrayOutputStream outContent = new ByteArrayOutputStream();
    private static final ByteArrayOutputStream errContent = new ByteArrayOutputStream();
    private static final PrintStream originalOut = System.out;
    private static final PrintStream originalErr = System.err;

    @BeforeAll
    public static void setUpStreams() {
        System.setOut(new PrintStream(outContent));
        System.setErr(new PrintStream(errContent));
    }

    @AfterAll
    public static void restoreStreams() {
        System.setOut(originalOut);
        System.setErr(originalErr);
    }

    @Test
    void solution() {
        Node n1 = new Node(1, null);
        Node n2 = new Node(2, n1);
        Node n3 = new Node(3, n2);
        Node n4 = new Node(4, n3);
        Node n5 = new Node(5, n4);
        Q6.solution(n5);
        assertEquals("1 2 3 4 5 ", outContent.toString());
        outContent.reset();
        Q6.solution(n1);
        assertEquals("1 ", outContent.toString());
        outContent.reset();
        Q6.solution(null);
        assertEquals("", outContent.toString());
    }
}
```

- 思路：使用栈达到先入后出的效果，同样可以使用递归。