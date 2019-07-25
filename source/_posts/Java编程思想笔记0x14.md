---
title: Java编程思想笔记0x14
date: 2019-07-24 15:59:24
categories: Java
tags:
  - Java
  - 笔记
---

# 并发

## 基本的线程机制

### 定义任务

```java
class LiftOff implements Runnable {
    protected int countDown = 10;
    private static int taskCount = 0;
    private final int id = taskCount++;
    public LiftOff() {}
    public LiftOff(int countDown) {
        this.countDown = countDown;
    }
    public String status() {
        return String.format("#%d(%s)", id, countDown > 0? countDown:"LiftOff");
    }
    @Override
    public void run() {
        while (countDown-- > 0) {
            System.out.println(status());
            Thread.yield();
        }
    }
}
```

- 实现`Runnable`接口并实现`run()`方法，即可执行自定义任务。
- 静态方法`Thread.yield()`是对线程调度器的一种建议，告知其可以让出CPU转移给其它线程。

### Thread类

```java
public class Main {
    public static void main(String[] args) throws Exception {
        for (int i = 0; i < 5; i++) {
            new Thread(new LiftOff()).start();
        }
    }
}
```



- 可以直接继承`Thread`类，或者向`Thread`构造器中传入`Runnable`对象，来构建由不同线程执行任务的对象。
- 上面代码一次执行的结果可能和另外一次不同，因为线程调度机制是非确定的。

### Executor



