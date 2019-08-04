---
title: Java编程思想笔记0x17
date: 2019-08-02 09:39:46
categories: Java
tags:
  - Java
  - 笔记
---

# 并发（四）

## 死锁

- 任务之间相互等待的连续循环，没有哪个线程能够继续执行，即死锁。
- 死锁发生的条件：
  1. **互斥条件**。任务使用的资源中至少有一个是不能共享的。
  2. **允许持有资源等待**。至少有一个任务必须持有一个资源并且正在等待获取一个当前被别的任务持有的资源。
  3. **资源不能被抢占**，任务必须把资源释放当作普通事件。
  4. **循环等待**。一个任务等待其他任务所持有的资源，后者又在等待另一个任务所持有的资源，这样一直下去，直到有一个任务在等待第一个任务所持有的资源，使得所有任务都被锁住。

## 部分类库的构件

### CountDownLatch

- `CountDownLatch`用来同步一个或多个任务，强制它们等待由其他任务执行的一组操作完成。
- 可以向`CountDownLatch`对象设置一个初始计数值，任何在这个对象上调用`wait()`的方法都将阻塞，直至这个计数值到达0。其他任务结束其工作时，可以在该对象上调用`countDown()`来减小这个计数值。`CountDownLatch`设计为只触发一次，计数值不能被重置。
- 调用`countDown()`的任务在产生这个调用时并没有被阻塞，只有对`await()`的调用会被阻塞，直至计数值到达0。

```java
class TaskPortion implements Runnable {
    private static int counter = 0;
    private final int id = counter++;
    private static Random random = new Random(47);
    private final CountDownLatch latch;

    public TaskPortion(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public void run() {
        try {
            doWork();
            latch.countDown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void doWork() throws InterruptedException {
        TimeUnit.MILLISECONDS.sleep(random.nextInt(2000));
        System.out.println(this + "completed");
    }

    @Override
    public String toString() {
        return String.format("%1$-3d ", id);
    }
}

class WaitingTask implements Runnable {
    private static int counter = 0;
    private final int id = counter++;
    private final CountDownLatch latch;

    public WaitingTask(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public void run() {
        try {
            latch.await();
            System.out.println("Latch barrier passed for " + this);
        } catch (InterruptedException e) {
            System.out.println(this + " interrupted");
        }
    }

    @Override
    public String toString() {
        return String.format("WaitingTask %1$-3d", id);
    }
}
public class Main {

    static final int SIZE = 100;
    public static void main(String[] args) throws Exception {
        ExecutorService exec = Executors.newCachedThreadPool();
        CountDownLatch latch = new CountDownLatch(SIZE);
        for (int i = 0; i < 10; i++)
            exec.execute(new WaitingTask(latch));
        for (int i = 0; i < SIZE; i++)
            exec.execute(new TaskPortion(latch));
        System.out.println("Launched all tasks");
        exec.shutdown();
    }
}
```

### CyclicBarrier

- 一组任务并行执行工作，然后在进行下一个步骤之前等待，直至所有任务都完成。它使得所有的并行任务都将在栅栏处列队，因此可以一致地向前移动，非常像`CountDownLatch`，只是`CountDownLatch`是只触发一次的事件，而`CyclicBarrier`可以多次重用。

```java
class Horse implements Runnable {
    private static int counter = 0;
    private final int id = counter++;
    private int strides = 0;
    private static Random random = new Random(47);
    private static CyclicBarrier cyclicBarrier;
    public Horse(CyclicBarrier cyclicBarrier) {
        Horse.cyclicBarrier = cyclicBarrier;
    }
    public synchronized int getStrides() {
        return strides;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                synchronized (this) {
                    strides += random.nextInt(3);
                }
                cyclicBarrier.await();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public String toString() {
        return "Horse " + id + " ";
    }
    public String tracks() {
        return "*".repeat(Math.max(0, getStrides())) + id;
    }
}
public class Main {
    private static final int FINISH_LINE = 75;
    private List<Horse> horseList = new ArrayList<>();
    private ExecutorService exec = Executors.newCachedThreadPool();

    private Main(int nHorses, final int pause) {
        CyclicBarrier cyclicBarrie = new CyclicBarrier(nHorses, () -> {
            System.out.println("=".repeat(FINISH_LINE));
            for (Horse horse : horseList)
                System.out.println(horse.tracks());
            for (Horse horse : horseList)
                if (horse.getStrides() >= FINISH_LINE) {
                    System.out.println(horse + "won!");
                    exec.shutdownNow();
                    return;
                }
            try {
                TimeUnit.MILLISECONDS.sleep(pause);
            } catch (InterruptedException e) {
                System.out.println("barrier action sleep");
            }
        });
        for (int i = 0; i < nHorses; i++ ) {
            Horse horse = new Horse(cyclicBarrie);
            horseList.add(horse);
            exec.execute(horse);
        }
    }
    public static void main(String[] args) throws Exception {
        int nHorses = 7;
        int pause = 200;
        new Main(nHorses, pause);
    }
}
```

- 可以向`CyclicBarrier`提供一个“栅栏动作”，一个`Runnable`，当计数值到达0时自动执行。这是与`CountDownLatch`的另一个区别。

### DelayQueue

- 一个无界的`BlockingQueue`，用于放置实现了`Delayed`接口的对象，其中的对象只能在其到期时才能从队列中取走。这种队列是有序的，即队头对象的<del>延迟</del>到期的时间最长。如果没有任何延迟到期，那么就不会有任何头元素，并且`poll()`将返回`null`。
- *延迟到期时间略带误导，指设定的到期时间减去当前时间，因此`DelayQueue`中元素排列是按设定时间从小到大排列。*

```java
class DelayedTask implements Runnable, Delayed {
    private static int counter = 0;
    private final int id = counter++;
    private final int delta;
    private final long trigger;
    protected static List<DelayedTask> sequence = new ArrayList<>();

    public DelayedTask(int delta) {
        this.delta = delta;
        trigger = System.nanoTime() + TimeUnit.NANOSECONDS.convert(this.delta, TimeUnit.MILLISECONDS);
        sequence.add(this);
    }

    @Override
    public long getDelay(TimeUnit unit) {
        return unit.convert(trigger - System.nanoTime(), TimeUnit.NANOSECONDS);
    }

    @Override
    public int compareTo(Delayed arg) {
        DelayedTask that = (DelayedTask) arg;
        return Long.compare(trigger, that.trigger);
    }

    @Override
    public void run() {
        System.out.println(this + " ");
    }

    @Override
    public String toString() {
        return String.format("[%1$-4d]", delta) + " Task " + id;
    }

    public String summary() {
        return "(" + id + ": " + delta + ")";
    }

    public static class EndSentinel extends DelayedTask {
        private ExecutorService exec;
        public EndSentinel(int delay, ExecutorService e) {
            super(delay);
            exec = e;
        }

        @Override

        public void run() {
            for (DelayedTask pt: sequence) {
                System.out.println(pt.summary() + " ");
            }
            System.out.println(this + " Calling shutdownNow()");
            exec.shutdownNow();
        }
    }
}

class DelayedTaskConsumer implements Runnable {
    private DelayQueue<DelayedTask> q;

    public DelayedTaskConsumer(DelayQueue<DelayedTask> q) {
        this.q = q;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted())
                q.take().run();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Finished DelayedTaskConsumer");
    }
}
public class Main {

    public static void main(String[] args) throws Exception {
        Random random = new Random(47);
        ExecutorService exec = Executors.newCachedThreadPool();
        DelayQueue<DelayedTask> queue = new DelayQueue<>();
        for (int i = 0; i < 20; i++)
            queue.put(new DelayedTask(random.nextInt(5000)));
        queue.add(new DelayedTask.EndSentinel(5000, exec));
        exec.execute(new DelayedTaskConsumer(queue));
    }
}
```

- `Delayed`接口有一个方法名为`getDelay()`，它可以用来告知延迟到期有多长时间，或者到场时间之前已经到期。*比起`BlockingQueue`，`DelayQueue#take()`会在没有元素或者没有元素到期时阻塞。*

### PriorityBlockingQueue

- *与上面`DelayQueue`类似，只不过优先级是人为指定。*

```java
class PriorizedTask implements Runnable, Comparable<PriorizedTask> {
    private Random random = new Random(47);
    private static int counter = 0;
    private final int id  = counter++;
    private final int priority;
    protected static List<PriorizedTask> sequence = new ArrayList<>();
    public PriorizedTask(int priority) {
        this.priority = priority;
        sequence.add(this);
    }

    @Override
    public int compareTo(PriorizedTask o) {
        return Integer.compare(priority, o.priority);
    }

    @Override
    public void run() {
        try {
            TimeUnit.MILLISECONDS.sleep(random.nextInt(250));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(this);
    }

    @Override
    public String toString() {
        return String.format("[%1$-3d]", priority) + " Task " + id;
    }

    public String summary() {
        return "(" + id + ": " + priority + ")";
    }

    public static class EndSentinel extends PriorizedTask {
        private ExecutorService exec;
        public EndSentinel(ExecutorService exec) {
            super(-1);
            this.exec = exec;
        }

        @Override
        public void run() {
            int count = 0;
            for (PriorizedTask pt: sequence) {
                System.out.println(pt.summary());
                if (++count % 5 == 0) System.out.println();
            }
            System.out.println(this + " Calling shutdownNow()");
            exec.shutdownNow();
        }
    }
}

class PrioritizedTaskProducer implements Runnable {
    private Random random = new Random(47);
    private Queue<Runnable> queue;
    private ExecutorService exec;

    public PrioritizedTaskProducer(Queue<Runnable> queue, ExecutorService exec) {
        this.queue = queue;
        this.exec = exec;
    }

    @Override
    public void run() {
        for (int i = 0; i <20; i++) {
            queue.add(new PriorizedTask(random.nextInt(10)));
            Thread.yield();
        }
        try {
            for (int i = 0; i < 10; i++) {
                TimeUnit.MILLISECONDS.sleep(250);
                queue.add(new PriorizedTask(10));
            }
            for (int i = 0; i < 10; i++) {
                queue.add(new PriorizedTask(i));
            }
            queue.add(new PriorizedTask.EndSentinel(exec));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Finished PrioritizedTaskProducer");
    }
}

class PrioritizedTaskConsumer implements Runnable {
    private PriorityBlockingQueue<Runnable> q;

    public PrioritizedTaskConsumer(PriorityBlockingQueue<Runnable> q) {
        this.q = q;
    }

    @Override
    public void run() {
        try {
            while (!Thread.interrupted()) {
                q.take().run();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Finished PrioritizedTaskConsumer");
    }
}
public class Main {

    public static void main(String[] args) throws Exception {
        ExecutorService exec = Executors.newCachedThreadPool();
        PriorityBlockingQueue<Runnable> queue = new PriorityBlockingQueue<>();
        exec.execute(new PrioritizedTaskProducer(queue, exec));
        exec.execute(new PrioritizedTaskConsumer(queue));
    }
}
```

### ScheduledExecutorService

- `ScheduledThreadPoolExecutor`提供`schedule()`（运行一次任务）和`scheduleAtFixedRate()`（可以设定初次延迟和再次运行间隔时间）来自动调度`Runnable`对象。

### Semaphore

```java
public class Main {

    private static final int COUNT = 40;
    private static Executor executor = Executors.newFixedThreadPool(COUNT);
    private static Semaphore semaphore = new Semaphore(10);

    public static void main(String[] args) {
        for (int i = 0; i < COUNT; i++) {
            executor.execute(new Main.Task());
        }
    }

    static class Task implements Runnable {
        @Override
        public void run() {
            try {
                semaphore.acquire();
                // 使用资源
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### Exchanger

- 可以让两个线程（只能是两个）交换一个对象。先调用`exchange()`的线程阻塞，直到另一个线程也调用`exchange()`。

```java
class Producer<T> extends Thread {
    private List<T> list = new ArrayList<>();
    private Exchanger<List<T>> exchanger;
    private Supplier<T> supplier;

    public Producer(Exchanger<List<T>> exchanger, Supplier<T> supplier) {
        super();
        this.exchanger = exchanger;
        this.supplier = supplier;
    }

    @Override
    public void run() {
        list.clear();
        for (int i = 0; i < 5; i++) {
            list.add(supplier.get());
        }
        try {
            list = exchanger.exchange(list);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Consumer<T> extends Thread {
    private List<T> list = new ArrayList<>();
    private Exchanger<List<T>> exchanger;

    public Consumer(Exchanger<List<T>> exchanger) {
        super();
        this.exchanger = exchanger;
    }

    @Override
    public void run() {
        try {
            list = exchanger.exchange(list);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.print("From Producer: ");
        System.out.print(list);
    }
}

public class Main {

    public static void main(String[] args) {
        Exchanger<List<Integer>> exchanger = new Exchanger<>();
        Random random = new Random(47);
        new Consumer<>(exchanger).start();
        new Producer<>(exchanger, () -> random.nextInt(100)).start();
    }
}
```

