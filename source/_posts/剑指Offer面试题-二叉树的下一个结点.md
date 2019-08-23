---
title: 剑指Offer面试题-二叉树的下一个结点
date: 2019-08-23 15:37:35
categories: 数据结构
tags:
  - Java
  - 剑指Offer
  - 树
---

# 二叉树的下一个结点

给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。

```java
class TreeLinkNode {
    int val;
    TreeLinkNode left = null;
    TreeLinkNode right = null;
    TreeLinkNode next = null;

    TreeLinkNode(int val) {
        this.val = val;
    }
}

public class Q8 {
    public TreeLinkNode GetNext(TreeLinkNode pNode) {
        if (pNode == null) return null;
        if (pNode.right != null) {
            TreeLinkNode l = pNode.right;
            while (l.left != null) {
                l = l.left;
            }
            return l;
        }
        TreeLinkNode t = pNode;
        while (t.next != null) {
            if (t.next.left == t) return t.next;
            t = t.next;
        }
        return null;
    }
}
```

- 思路：分类讨论。如果给出节点有右孩子，从右孩子开始一直访问其左孩子，直到当前节点没有左孩子，返回当前节点。如果给出节点没有右孩子，那么如果它是其父节点的左孩子，则返回其父节点。如果不是其父节点的左孩子，则继续找父节点的父节点，直到当前节点是父节点的左孩子，返回当前节点的父节点。如果找到根节点，那么给出节点没有下一个节点。