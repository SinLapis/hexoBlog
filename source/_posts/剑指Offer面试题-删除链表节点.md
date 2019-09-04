---
title: 剑指Offer面试题-删除链表节点
date: 2019-09-04 16:21:23
categories: 数据结构
tags:
  - Java
  - 剑指Offer
  - 链表
---

# 删除链表节点

## 在O(1)时间内删除链表节点

给定单向链表的头指针和一个节点指针，定义一个函数在O(1)时间内删除该节点。

```java
class Node {
    int value;
    Node next;
    public Node(int value) {
        this.value = value;
    }
}
class Solution {
    public Node solute(Node head, Node p) {
        if (p.next != null) {
            p.value = p.next.value;
            p.next = p.next.next;
            return head;
        } else if (head == p) {
            return null;
        } else {
            Node next = head;
            while (next.next != p) {
                next = next.next;
            }
            next.next = null;
            return head;
        }
    }
}
```

- 思路：把要删除的节点的下一个节点的值覆盖到当前节点上，再删除下一个节点。注意需要判断要删除节点是否为尾节点，以及链表中是否只有一个节点。

## 删除排序链表中重复的节点

在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表1->2->3->3->4->4->5 处理后为 1->2->5

```java
class Node {
    int value;
    Node next;

    public Node(int value) {
        this.value = value;
    }
}

class Solution {
    public Node solute(Node head) {
        if (head == null) return null;
        boolean deleteHead = false;
        if (head.next != null && head.value == head.next.value) deleteHead = true;
        Node last = head;
        Node next = head.next;
        int lastValue = head.value;
        while (next != null) {
            if (next.value == lastValue) {
                last.next = next.next;
            } else {
                lastValue = next.value;
                if (next.next == null || next.next.value != lastValue) {
                    last = last.next;
                } else {
                    last.next = next.next;
                }
            }
            next = next.next;
        }
        if (deleteHead) {
            while (true) {
                if (head.next == null || head.value != head.next.value) return head.next;
                head = head.next;
            }
        }
        return head;
    }
}
```

- 思路：如要删除头节点最后处理（头指针向下到第一个值不同的节点），中间节点保存上一个值，如果重复则`next`直接向下，如果`next`的值发生变动则进行预判，相同则直接向下并更新上一个值，否则`last`向下。