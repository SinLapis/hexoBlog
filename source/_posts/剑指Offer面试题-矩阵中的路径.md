---
title: 剑指Offer面试题-矩阵中的路径
date: 2019-08-27 10:29:36
categories: 算法
tags:
  - Java
  - 剑指Offer
  - 回溯法
---

# 矩阵中的路径

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则该路径不能再进入该格子。 例如 a b c e s f c s a d e e 矩阵中包含一条字符串"bccced"的路径，但是矩阵中不包含"abcb"路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该格子。

```java
class Point {
    int x;
    int y;
    public boolean[] visited = {false, false, false, false};

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object obj) {
        try {
            return x == ((Point) obj).x && y == ((Point) obj).y;
        } catch (ClassCastException ce) {
            return super.equals(obj);
        }
    }

}

public class Solution {
    private void resetMB(boolean[][] mb) {
        for (boolean[] bs : mb) {
            for (boolean b : bs) {
                b = false;
            }
        }
    }

    public boolean hasPath(char[] matrix, int rows, int cols, char[] str) {
        int firstRow = 0, firstCol = 0;
        char[][] mc = new char[rows][cols];
        boolean[][] mb = new boolean[rows][cols];
        resetMB(mb);
        for (int i = 0; i < rows; i++) {
            mc[i] = new char[cols];
            for (int j = 0; j < cols; j++) {
                mc[i][j] = matrix[i * cols + j];
            }
        }
        while (true) {
            // 先定位第一个字符
            outer:
            for (; firstRow < rows; firstCol = 0, firstRow++) { //[1]
                for (; firstCol < cols; firstCol++) {
                    if (matrix[firstRow * cols + firstCol] == str[0]) break outer;
                }
            }
            if (firstRow >= rows) return false;
            Stack<Point> route = new Stack<>();
            route.push(new Point(firstRow, firstCol));
            int strIndex = 1;
            if (strIndex >= str.length) return true; //[2]
            resetMB(mb);
            mb[firstRow][firstCol] = true;
            // 在定位周围找下一个字符
            while (!route.isEmpty()) {
                Point current = route.peek();
                int visitOri = -1;
                if (
                        current.x - 1 >= 0 &&
                                !current.visited[0] &&
                                !mb[current.x - 1][current.y] &&
                                mc[current.x - 1][current.y] == str[strIndex]
                ) {
                    //0
                    route.push(new Point(current.x - 1, current.y));
                    route.peek().visited[1] = true;
                    visitOri = 0;
                    mb[current.x - 1][current.y] = true;
                    strIndex++;
                } else if (
                        current.x + 1 < rows &&
                                !current.visited[1] &&
                                !mb[current.x + 1][current.y] &&
                                mc[current.x + 1][current.y] == str[strIndex]
                ) {
                    //1
                    route.push(new Point(current.x + 1, current.y));
                    route.peek().visited[0] = true;
                    visitOri = 1;
                    mb[current.x + 1][current.y] = true;
                    strIndex++;
                } else if (
                        current.y - 1 >= 0 &&
                                !current.visited[2] &&
                                !mb[current.x][current.y - 1] &&
                                mc[current.x][current.y - 1] == str[strIndex]
                ) {
                    //2
                    route.push(new Point(current.x, current.y - 1));
                    route.peek().visited[3] = true;
                    visitOri = 2;
                    mb[current.x][current.y - 1] = true;
                    strIndex++;
                } else if (
                        current.y + 1 < cols &&
                                !current.visited[3] &&
                                !mb[current.x][current.y + 1] &&
                                mc[current.x][current.y + 1] == str[strIndex]
                ) {
                    //3
                    route.push(new Point(current.x, current.y + 1));
                    route.peek().visited[2] = true;
                    visitOri = 3;
                    mb[current.x][current.y + 1] = true;
                    strIndex++;
                } else {
                    Point p = route.pop();
                    mb[p.x][p.y] = false;
                    strIndex--;
                }
                if (strIndex >= str.length) return true;
                if (visitOri >= 0) current.visited[visitOri] = true;
            }
            if (firstCol >= cols - 1) { //[1]
                firstCol = 0;
                firstRow++;
            } else {
                firstCol++;
            }
        }
    }
}
```

- 思路：先找开始位置，然后在周围找下一个。要保存两种状态，一是当前已经选择的位置（由`boolean`二维数组保存），二是回溯失败的状态（由`Point`对象保存）。注意：如果要找下一个初始位置，[1]处要对初始坐标进行处理，包括移动到下一个坐标和一行循环完成置列号为0（因为要保存之前的状态，所以不能在循环列时置0）；[2]处防止不需要进入循环时要返回`true`的情况。