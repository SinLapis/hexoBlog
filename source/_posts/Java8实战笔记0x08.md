---
title: Java8实战笔记0x08
date: 2019-08-21 09:23:06
categories: Java
tags:
  - Java
  - 笔记
  - 并发
---

# CompletableFuture：组合异步编程（二）

## 对多个异步任务进行流水线操作

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
    public String getPrice(String product) {
       double price = calculatePrice(product);
       Discount.Code code = Discount.Code.values()[random.nextInt(Discount.Code.values().length)];
       return String.format("%s:%.2f:%s", name, price, code);
    }
}

class Discount {
    public enum Code {
        NONE(0), SILVER(5), GOLD(10), PLATINUM(15), DIAMOND(20);

        private final int percentage;

        Code(int percentage) {
            this.percentage = percentage;
        }
    }

    public static String applyDiscount(Quote quote) {
        return quote.getShopName() + " price is " + Discount.apply(quote.getPrice(), quote.getDiscountCode());
    }

    public static double apply(double price, Code code) {
        Shop.delay();
        return price * (100 - code.percentage) / 100;
    }
}

class Quote {
    private final String shopName;
    private final double price;
    private final Discount.Code discountCode;

    public Quote(String shopName, double price, Discount.Code discountCode) {
        this.shopName = shopName;
        this.price = price;
        this.discountCode = discountCode;
    }

    public static Quote parse(String s) {
        String[] split = s.split(":");
        String shopName = split[0];
        double price = Double.parseDouble(split[1]);
        Discount.Code discountCode = Discount.Code.valueOf(split[2]);
        return new Quote(shopName, price, discountCode);
    }

    public String getShopName() {
        return shopName;
    }

    public double getPrice() {
        return price;
    }

    public Discount.Code getDiscountCode() {
        return discountCode;
    }
}
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

   public static List<String> findPrices(String product) {
       return shops.stream()
               .map(shop -> shop.getPrice(product))
               .map(Quote::parse)
               .map(Discount::applyDiscount)
               .collect(Collectors.toList());
   }

   public static List<String> findPricesByFuture(String product) {
       List<CompletableFuture<String>> priceFuture = shops.stream()
               .map(shop -> CompletableFuture.supplyAsync(() -> shop.getPrice(product), executor))
               .map(future -> future.thenApply(Quote::parse))
               .map(future -> future.thenCompose(quote -> CompletableFuture.supplyAsync(
                       () -> Discount.applyDiscount(quote), executor
               )))
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
        test(Main::findPrices);
        test(Main::findPricesByFuture);
    }
}
/* Output
[BestPrice price is 130.6535, LetsSaveBig price is 77.87200000000001, MyFavoriteShop price is 113.27, BuyItAll price is 152.532, Steam price is 113.49600000000001, Epic price is 115.32, GOG price is 106.46, Taptap price is 142.44299999999998, W price is 104.76, V price is 132.297, Z price is 136.6575, Y price is 99.2085, X price is 95.60700000000001]
Done in 26058 msecs
[BestPrice price is 126.36, LetsSaveBig price is 94.728, MyFavoriteShop price is 149.43, BuyItAll price is 107.01, Steam price is 83.936, Epic price is 153.5485, GOG price is 142.53300000000002, Taptap price is 99.2275, W price is 97.064, V price is 104.976, Z price is 87.40549999999999, Y price is 138.051, X price is 140.409]
Done in 2007 msecs
*/
```

