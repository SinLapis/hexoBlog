---
title: 剑指Offer面试题-机器人的运动范围
date: 2019-09-02 15:28:27
categories: 算法
tags:
  - Java
  - 剑指Offer
  - BFS
---

# 机器人的运动范围

地上有一个m行和n列的方格。一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？

```java
class Point {
    int x;
    int y;
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
class Solution {
    private int addSingle(int n) {
        int result = 0;
        while (n > 0) {
            result += n % 10;
            n /= 10;
        }
        return result;
    }
    private boolean check(int x, int y, int k, boolean[][] visited) {
        if (
                x < 0 ||
                y < 0 ||
                x >= visited.length ||
                y >= visited[0].length ||
                visited[x][y]
        ) return false; // [1]
        return addSingle(x) + addSingle(y) <= k;
    }
    public int movingCount(int threshold, int rows, int cols) {
        if (threshold < 0 || rows== 0 || cols == 0) return 0;
        if (threshold == 0) return 1;
        // 初始化访问标记矩阵
        boolean[][] visited = new boolean[rows][cols];
        for (boolean[] lb: visited) {
            lb = new boolean[cols];
            for (boolean b: lb) {
                b = false;
            }
        }
        // BFS
        Queue<Point> queue = new LinkedList<>();
        queue.add(new Point(0, 0));
        Point current;
        int result = 1;
        visited[0][0] = true;
        while (!queue.isEmpty()) {
            current = queue.poll();
            if (check(current.x - 1, current.y, threshold, visited)) {
                queue.add(new Point(current.x - 1, current.y));
                visited[current.x - 1][current.y] = true;
                result++;
            }
            if (check(current.x + 1, current.y, threshold, visited)) {
                queue.add(new Point(current.x + 1, current.y));
                visited[current.x + 1][current.y] = true;
                result++;
            }
            if (check(current.x, current.y - 1, threshold, visited)) {
                queue.add(new Point(current.x, current.y - 1));
                visited[current.x][current.y - 1] = true;
                result++;
            }
            if (check(current.x, current.y + 1, threshold, visited)) {
                queue.add(new Point(current.x, current.y + 1));
                visited[current.x][current.y + 1] = true;
                result++;
            }
        }
        return result;
    }
}
```

- 思路：没有使用书上的解法，使用BFS，用队列实现。注意[1]处边界条件。