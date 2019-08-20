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

### 将同步方法转换为异步方法

```java
class Shop {
    private Random random = new Random(47);

    public static void delay() {
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    private double calculatePrice(String product) {
        delay();
        return random.nextDouble() * product.charAt(0) + product.charAt(1);
    }

    // 同步方法
    public double getPrice(String product) {
        return calculatePrice(product);
    }

    // 异步方法
    public Future<Double> getPriceAsync(String product) {
        CompletableFuture<Double> futurePrice = new CompletableFuture<>();
        new Thread(() -> futurePrice.complete(calculatePrice(product))).start();
        return futurePrice;
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        Shop shop = new Shop();
        long start = System.nanoTime();
        Future<Double> futurePrice = shop.getPriceAsync("my favorite product");
        long invocationTime = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Invocation returned after " + invocationTime + " msecs");
        // 执行其它操作
        Shop.delay();
        try {
           double price = futurePrice.get();
            System.out.printf("Price is %.2f%n", price);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        long retrievalTime = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Price returned after " + retrievalTime + "msecs");
    }
}
```

### 错误处理

- 如果异步方法发生错误，其异常会被限制在当前线程中，并且该线程最终会被杀死，而这会导致`get`方法永远阻塞。可以使用重载后的`get`方法，支持超时参数来防止永久阻塞，程序会得到`TimeoutException`。但是还是无法得知异步方法线程的错误原因，此时应该使用`CompletableFuture`的`completeExceptionally`方法将导致`CompletableFuture`内发生的问题抛出。

```java
public Future<Double> getPriceAsync(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    new Thread(() -> {
        try {
            futurePrice.complete(calculatePrice(product));
        } catch (Exception e) {
            futurePrice.completeExceptionally(e);
        }
    }).start();
    return futurePrice;
}
```

### 使用工厂方法创建CompletableFuture

- `CompletableFuture.supplyAsync()`是一个工厂方法，接受一个`Supplier<T>`参数，返回一个`CompletableFuture`对象。使用该方法创建的`CompletableFuture`与上面带有异常处理的代码是等价的。

```java
public Future<Double> geetPriceAsync(String product) {
    return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```

## 免受阻塞

```java
class Shop {
    private Random random = new Random();
    private String name;

    public Shop(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public static void delay() {
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    private double calculatePrice(String product) {
        delay();
        return random.nextDouble() * product.charAt(0) + product.charAt(1);
    }

    // 同步方法
    public double getPrice(String product) {
        return calculatePrice(product);
    }

    // 异步方法
    public Future<Double> getPriceAsync(String product) {
        CompletableFuture<Double> futurePrice = new CompletableFuture<>();
        new Thread(() -> {
            try {
                futurePrice.complete(calculatePrice(product));
            } catch (Exception e) {
                futurePrice.completeExceptionally(e);
            }
        }).start();
        return futurePrice;
    }
}
public class Main {
    public static List<String> findPrices(String product, List<Shop> shops) {
        return shops.stream()
                .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
                .collect(Collectors.toList());
    }
    public static void main(String[] args) throws Exception {
        List<Shop> shops = Arrays.asList(
                new Shop("BestPrice"),
                new Shop("LetsSaveBig"),
                new Shop("MyFavoriteShop"),
                new Shop("BuyItAll")
        );
        long start = System.nanoTime();
        System.out.println(findPrices("Fallout New Vegas", shops));
        long duration = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Done in " + duration + " msecs");
    }
}
/* Output:
[BestPrice price is 97.75, LetsSaveBig price is 118.44, MyFavoriteShop price is 98.66, BuyItAll price is 149.10]
Done in 4013 msecs
*/
```

### 使用并行流对请求进行并行操作

```java
public class Main {
    public static List<String> findPrices(String product, List<Shop> shops) {
        return shops.parallelStream()
                .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
                .collect(Collectors.toList());
    }
    public static void main(String[] args) throws Exception {
        List<Shop> shops = Arrays.asList(
                new Shop("BestPrice"),
                new Shop("LetsSaveBig"),
                new Shop("MyFavoriteShop"),
                new Shop("BuyItAll")
        );
        long start = System.nanoTime();
        System.out.println(findPrices("Fallout New Vegas", shops));
        long duration = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Done in " + duration + " msecs");
    }
}
/* Output:
[BestPrice price is 163.63, LetsSaveBig price is 125.60, MyFavoriteShop price is 148.02, BuyItAll price is 118.71]
Done in 1022 msecs
*/
```

### 使用CompletableFuture发起异步请求

```java
public class Main {
    public static List<String> findPrices(String product, List<Shop> shops) {
        List<CompletableFuture<String>> priceFuture = shops.stream()
                .map(shop -> CompletableFuture.supplyAsync(
                        () -> shop.getName() + " price is " + shop.getPrice(product)
                ))
                .collect(Collectors.toList());
        return priceFuture.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList());
    }
    public static void main(String[] args) throws Exception {
        List<Shop> shops = Arrays.asList(
                new Shop("BestPrice"),
                new Shop("LetsSaveBig"),
                new Shop("MyFavoriteShop"),
                new Shop("BuyItAll")
        );
        long start = System.nanoTime();
        System.out.println(findPrices("Fallout New Vegas", shops));
        long duration = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Done in " + duration + " msecs");
    }
}
/* Output:
[BestPrice price is 116.10675116625683, LetsSaveBig price is 159.1609417232611, MyFavoriteShop price is 166.64834055365026, BuyItAll price is 121.8279536327426]
Done in 1044 msecs
*/
```

- *~~此处执行时间远低于书中所述（2005 ms），比较接近并行流的性能。有待查证。~~和CPU逻辑处理器数量有关，运行环境是12核，因此当把商店数量改为13后，执行时间为2063 ms。*

### 使用定制的执行器

- 可以使用线程池。线程池大小估算：`T = N * U * (1 + W / C)`，其中，`T`为线程池大小，`N`为CPU核数，`U`为期望的CPU利用率，`W / C`为等待时间与计算时间比率。

```java
public class Main {
    private static final List<Shop> shops = Arrays.asList(
            new Shop("BestPrice"),
            new Shop("LetsSaveBig"),
            new Shop("MyFavoriteShop"),
            new Shop("BuyItAll"),
            new Shop("Steam"),
            new Shop("Epic"),
            new Shop("GOG"),
            new Shop("Taptap"),
            new Shop("W"),
            new Shop("V"),
            new Shop("Z"),
            new Shop("Y"),
            new Shop("X")
    );
    private static final Executor executor = Executors.newFixedThreadPool(
            Math.min(shops.size(), 100),
            r -> {
                Thread t = new Thread(r);
                t.setDaemon(true);
                return t;
            });

    public static List<String> findPricesByFuture(String product) {
        List<CompletableFuture<String>> priceFuture = shops.stream()
                .map(shop -> CompletableFuture.supplyAsync(
                        () -> shop.getName() + " price is " + shop.getPrice(product)
                ))
                .collect(Collectors.toList());
        return priceFuture.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList());
    }

    public static List<String> findPricesByFutureWithExec(String product) {
        List<CompletableFuture<String>> priceFuture = shops.stream()
                .map(shop -> CompletableFuture.supplyAsync(
                        () -> shop.getName() + " price is " + shop.getPrice(product),
                        executor
                ))
                .collect(Collectors.toList());
        return priceFuture.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList());
    }

    public static void test(Function<String, List<String>> findPrices) {
        long start = System.nanoTime();
        System.out.println(findPrices.apply("Fallout New Vegas"));
        long duration = (System.nanoTime() - start) / 1_000_000;
        System.out.println("Done in " + duration + " msecs");
    }

    public static void main(String[] args) throws Exception {
        test(Main::findPricesByFuture);
        test(Main::findPricesByFutureWithExec);
    }
}
/* Output:
[BestPrice price is 123.81188673960165, LetsSaveBig price is 134.2037700185142, MyFavoriteShop price is 108.25237001214239, BuyItAll price is 115.00922099155312, Steam price is 111.23106850193753, Epic price is 99.22328167208421, GOG price is 164.50758310616402, Taptap price is 121.17345205244206, W price is 123.06486848925549, V price is 138.86613263159464, Z price is 160.93733921008214, Y price is 150.61296275286742, X price is 151.1427293015105]
Done in 2063 msecs
[BestPrice price is 147.8957557234686, LetsSaveBig price is 115.91180890944013, MyFavoriteShop price is 145.9127196738407, BuyItAll price is 153.58049341481873, Steam price is 127.4807398778012, Epic price is 143.5858168683404, GOG price is 151.72178968386282, Taptap price is 140.27525423457325, W price is 142.2637869480202, V price is 115.3615656625179, Z price is 151.1027261194255, Y price is 121.78157514886323, X price is 150.22069837374056]
Done in 1003 msecs
*/
```

### 并行的方案选择

- 如果进行计算密集的操作，并且没有I/O，那么推荐使用`Stream`接口。反之，如果并行单元需要等待I/O操作，那么使用`CompletableFuture`灵活性更好，还可以根据需要设置线程数。而且，如果此时使用流，那么流的延迟特性会导致难以判断什么时候触发了等待。
- *流的延迟特性：`Stream`的操作由零个或多个中间操作和一个结束操作两部分组成。只有执行了结束操作，`Stream`定义的中间操作才会依次执行。*