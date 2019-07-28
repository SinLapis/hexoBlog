---
title: Java编程思想笔记0x14
date: 2019-07-24 15:59:24
categories: Java
tags:
  - Java
  - 笔记
---

# 并发（一）

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

- `Executor`能够管理异步任务的执行，无须显式地管理线程的生命周期。

```java
public class Main {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new LiftOff());
        }
        exec.shutdown();
    }
}
```

- 在上面代码中，`ExecutorService`对象使用静态方法`Executors.newCachedThreadPool()`创建，`ExecutorService#execute()`为每一个任务都创建了一个线程。
- `ExecutorService#shutdown()`方法可以防止新任务提交给该`Executor`，而当前线程将继续运行在`shutdown()`被调用之前提交的所有任务，待所有任务均执行完毕后，当前线程将会尽快退出。
- `ChchedThreadPool`在程序执行过程中通常会创建与所需数量相同的线程，然后在它回收旧线程时停止创建新线程。

```java
public class Main {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            exec.execute(new LiftOff());
        }
        exec.shutdown();
    }
}
```

- `FixedThreadPool`可以一次性预先执行代价高昂的线程分配，因而也可以限制线程的数量。由于不需要为每个任务都固定付出创建线程的开销，因此可以节省时间。在事件驱动的系统中，需要线程的事件处理器，通过直接从池中获取线程以尽快得到服务。在这个过程中，由于线程总数是有限的，因此不会出现资源滥用。
- 在任何线程池中，现有线程在可能的情况下都会被自动复用。
- `SingleThreadExecutor`类似于线程数量为1的`FixedThreadPool`，可以用于在另一个线程中连续运行任何事物（长期服务），例如监听进入的套接字连接的任务。如果向`SingleThreadExecutor`提交了多个任务，那么这些任务将会排队执行，所有的任务将使用相同的线程。这能够保证在任意时刻在任意线程中都只有唯一的任务在运行，不需要在共享资源上处理同步并有限制的使用这些资源。

### 从任务中产生返回值

- 如果需要任务在完成时能够返回一个值，可以实现`Callable`接口。`Callable`是一种具有类型参数的泛型，它的类型参数表示的是从方法`call()`中返回值的类型。向`Executor`提交任务时使用`ExecutorService#submit()`。

```java
class Test implements Callable<String> {
    private int id;
    public Test(int id) {
        this.id = id;
    }

    @Override
    public String call() throws Exception {
        return "result: " + id;
    }
}
public class Main {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newFixedThreadPool(5);
        List<Future<String>> result = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            result.add(exec.submit(new Test(i)));
        }
        for (Future<String> r: result) {
            try {
                System.out.println(r.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            } finally {
                exec.shutdown();
            }
        }
        exec.shutdown();
    }
}
```

- `ExecutorService#submit()`方法会产生`Future`对象，它用`Callable`返回的结果的特定类型进行参数化。可以调用`Future#isDone()`来查询`Future`是否已经完成，如果完成可以调用`Future#get()`获取对应结果。也可以直接调用`Future#get()`，此时会发生阻塞，直到结果准备就绪。

### 休眠

- 可以调用`TimeUnit.MILLSECONDS.sleep()`来中止任务执行给定的时间，同时会抛出`InterruptedException`异常。由于异常不能跨线程传播回主线程，因此必须在本地处理所有在任务内部产生的异常。

### 优先级

- 线程的优先级将该线程的重要性传递给了调度器，调度器将倾向度让优先级最高的线程先执行。然而，这并不意味着优先级较低的线程得不到执行，仅仅是执行的频率较低。

```java
class Test implements Runnable {
    private int countDown = 5;
    private volatile double d;
    private int priority;
    public Test(int priority) {
        this.priority = priority;
    }
    public String toString() {
        return Thread.currentThread() + ": " + countDown;
    }

    @Override
    public void run() {
        Thread.currentThread().setPriority(priority);
        while(true) {
            for (int i = 1; i < 100000; i++) {
                d += (Math.PI + Math.E) / (double)i;
                if (i % 1000 == 0) Thread.yield();
            }
            System.out.println(this);
            if (--countDown == 0) return;
        }
    }
}
public class Main {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new Test(Thread.MAX_PRIORITY));
            exec.execute(new Test(Thread.MIN_PRIORITY));
        }
    }
}
```

- 可以使用`getPriority()`来读取现有线程的优先级，并且可以在任何时候通过调用`setPriority()`来修改优先级。
- 尽管JDK有10个优先级，但是和多数的操作系统都不能映射得很好。唯一可移植的方法是只使用`MAX_PRIORITY`、`NORM_PRIORITY`和`MIN_PRIORITY`。

### 后台线程

- 后台线程，是指在程序运行的时候在后台提供一种通用服务的线程，并且这种线程并不属于程序中不可或缺的部分。因此，当所有的非后台线程结束时，程序也就终止了，同时会杀死进程中所有的后台线程。
- 可以使用`Thread#setDaemon()`来设置线程为后台线程，但必须在线程启动之前设置。

```java
class DaemonThreadFactory implements ThreadFactory {
    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setDaemon(true);
        return t;
    }
}
class Daemon implements Runnable {
    @Override
    public void run() {
        try {
            while (true) {
                TimeUnit.MILLISECONDS.sleep(100);
                System.out.println(Thread.currentThread() + " " + this);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
public class Main {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool(new DaemonThreadFactory());
        for (int i = 0; i < 10; i++) {
            exec.execute(new Daemon());
        }
        System.out.println("all start.");
        TimeUnit.MILLISECONDS.sleep(500);
    }
}
```

- 每个静态的`ExecutorService`创建方法可以被重载为接收一个`ThreadFactory`对象，而这个工厂对象将用于创建新的线程。在上面代码中，自定义的工厂将新的线程均设置成为后台线程。
-  可以通过调用`Thread#isDaemon()`方法来确定线程是否为一个后台线程。后台线程创建的线程都将自动设置成为后台线程。

```java
class Daemon implements Runnable {
    @Override
    public void run() {
        try {
            System.out.println("starting daemon");
            TimeUnit.SECONDS.sleep(1); // [1]
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("run?");
        }
    }
}
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(new Daemon());
        t.setDaemon(true);
        t.start();
    }
}
```

- 当最后一个非后台线程终止时，JVM就会立即关闭所有后台进程（例如上面代码，尽管执行到[1]处会被强制停止，但不会执行`finally`子句）。这不是关闭后台进程优雅的方式，相应的，非后台的`Executor`控制的所有任务可以同时被关闭，这是一种更好的解决方式。

### 加入一个线程

- 一个线程可以在其他线程之上调用`join()`方法，此时调用`join()`的线程将被挂起，直到目标线程结束才回复（即`t.isAlive`为假）。也可以在调用`join()`时带上一个超时参数，如果发生超时目标线程还没有结束，`join()`方法也会返回。

### 捕获线程异常

```java
class ThrowException implements Runnable {
    @Override
    public void run() {
        throw new RuntimeException();
    }
}
public class Main {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();
        try {
            exec.execute(new ThrowException());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- 上面代码中的`try`语句无法捕获线程中的异常。

```java
class ThrowException implements Runnable {
    @Override
    public void run() {
        throw new RuntimeException();
    }
}

class CatchException implements Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println("caught: " + e);
    }
}

class ExceptionThreadFactory implements ThreadFactory {
    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setUncaughtExceptionHandler(new CatchException());
        return t;
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool(new ExceptionThreadFactory());
        exec.execute(new ThrowException());
    }
}
/* Output:
caught: java.lang.RuntimeException
*/
```

- `Thread.UncaughtException`接口可以为每个`Thread`对象都附着一个异常处理器。其中`uncaughtException()`方法会在线程因未捕获异常而退出前被调用。上面代码中创建了一个`ThreadFactory`子类来为线程添加异常处理器。