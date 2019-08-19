---
title: Java8实战笔记0x07
date: 2019-08-19 09:48:25
categories: Java
tags:
  - Java
  - 笔记
---

# CompletableFuture：组合异步编程

## Future接口

- `Future`接口是对将来某个时刻会发生的结果进行建模。它建模了一种异步计算，返回一个执行运算结果的引用，当运算结束后，这个引用被返回公文袋调用方。在`Future`中触发那些潜在耗时的操作把调用线程解放出来，让它能继续执行其他有价值的工作。

```java
ExecutorService executor = Executors.newCachedThreadPool();
Future<Double> future = executor.submit(new Callable<Double> () {
    public Double call() {
        return doSomeLongComputation();
    }
});
doSomethingElse();

try {
    Double result = future.get(1, TimeUnit.SECONDS);
} catch (ExecutionException ee) {
    ee.printStackTrace();
} catch (InterruptedException ie) {
    ie.printStackTrace();
} catch (TimeoutException te) {
    te.printStackTrace();
}
```

### Future接口的局限性

- `Future`表达能力有限，某些异步计算用`Future`难以表达。

## 使用CompletableFuture构建异步应用





