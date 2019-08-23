---
title: 剑指Offer面试题-重建二叉树
date: 2019-08-22 16:37:49
categories: 数据结构
tags:
  - Java
  - 剑指Offer
  - 树
---

# 重建二叉树

输入某二叉树的前序遍历和中序遍历的结果，重建该二叉树，返回二叉树的根节点。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。二叉树节点定义：

```java
class TreeNode {
    public int value;
    public TreeNode left;
    public TreeNode right;
}
```

```java
public class Solution {
    private static void buildTree(
            int[] preOrder, int preStart,
            int[] inOrder, int inStart, int inEnd,
            TreeNode current
    ) {
        int index;
        for (index = inStart; index <= inEnd; index++) {
            if (inOrder[index] == current.value) break;
        }
        int leftLength = index - inStart;
        int rightLength = inEnd - index;
        if (leftLength > 0) {
            TreeNode left = new TreeNode(preOrder[preStart + 1]);
            current.left = left;
            buildTree(
                    preOrder, preStart + 1,
                    inOrder, index - leftLength, index - 1,
                    left
            );
        }
        if (rightLength > 0) {
            TreeNode right = new TreeNode(preOrder[preStart + leftLength + 1]);
            current.right = right;
            buildTree(
                    preOrder, preStart + leftLength + 1,
                    inOrder, index + 1, index + rightLength,
                    right
            );
        }
    }
    public static TreeNode reConstructBinaryTree(int[] preOrder, int[] inOrder) {
        if (preOrder == null || preOrder.length == 0) return null;
        TreeNode root = new TreeNode(preOrder[0]);
        buildTree(
                preOrder, 0,
                inOrder, 0, inOrder.length - 1,
                root
        );
        return root;
    }
}
```

- 思路：前序遍历的第一个元素就是当前树的根节点，在中序遍历中找到当前节点的位置就能算出左右序列长度，从而进行递归。