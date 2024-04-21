---
layout: post
title: CompletableFuture：Java 8 中的异步编程利器
tagline: by 付义帆
categories: 异步编程
tags:
- 异步编程
---

在现代软件开发中，异步编程已成为提升系统性能、响应能力和可扩展性的关键手段。Java 8 引入了 `CompletableFuture` 类，为 Java 平台带来了强大的异步编程能力。

本篇文章将带你认识这个异步编程神器：`CompletableFuture`。

## 什么是 CompletableFuture

`CompletableFuture` 是 Java 8 引入的 `java.util.concurrent` 包下的一个类，它代表一个异步计算的结果，可以是已完成、正在进行或尚未开始。
`CompletableFuture` 提供了一种灵活、类型安全的方式来表达异步操作的生命周期，包括创建、组合、处理结果以及处理异常。其设计灵感来源于函数式编程中的 Promises/Futures 模式，旨在简化异步编程模型，提高代码的可读性和可维护性。

## CompletableFuture 的核心功能

### 1. 创建 CompletableFuture

#### a. `completedFuture(T value)`

`completedFuture(T value)` 是一个静态工厂方法，用于创建一个已经处于完成状态且包含给定结果值的 `CompletableFuture`。这适用于预先计算好的结果或常量值，使得其他组件可以以异步形式消费。

#### b. `supplyAsync(Supplier<U> supplier, Executor executor)`

`supplyAsync()` 方法接受一个 `Supplier` 函数和一个可选的 `Executor`，异步执行 `supplier.get()`，并将结果封装到一个新的 `CompletableFuture` 中。计算在 `Executor` 管理的线程中进行，不阻塞当前线程。

#### c. `runAsync(Runnable runnable, Executor executor)`

类似于 `supplyAsync()`，`runAsync()` 接受一个 `Runnable` 任务和一个 `Executor`，异步执行任务。由于 `Runnable` 没有返回值，`runAsync()` 返回的 `CompletableFuture` 完成时没有结果。

### 2. 组合 CompletableFuture

#### a. `thenApply(Function<? super T,? extends U> fn)`

在当前 `CompletableFuture` 完成后，应用给定的 `Function` 处理结果，并返回一个新的 `CompletableFuture`，其结果为 `Function` 应用后的值。

#### b. `thenAccept(Consumer<? super T> action)`

当当前 `CompletableFuture` 完成后，执行给定的 `Consumer` 消费结果。由于 `Consumer` 没有返回值，返回的 `CompletableFuture` 完成时没有结果。

#### c. `thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)`

当当前 `CompletableFuture` 与另一个 `CompletionStage`（如另一个 `CompletableFuture`）都完成时，应用给定的 `BiFunction` 合并两个结果，并返回一个新的 `CompletableFuture`。

### 3. 异常处理

#### a. `exceptionally(Function<Throwable,? extends T> fn)`

当当前 `CompletableFuture` 因异常而未能正常完成时，应用给定的 `Function` 处理异常，并返回一个新的 `CompletableFuture`，其结果为 `Function` 应用后的值。

#### b. `handle(BiFunction<? super T, Throwable, ? extends U> fn)`

无论当前 `CompletableFuture` 正常完成还是因异常未能完成，都会应用给定的 `BiFunction` 处理结果或异常，并返回一个新的 `CompletableFuture`。

### 4. 其他重要方法

#### a. `allOf(CompletableFuture<?>... cfs)`

创建一个新的 `CompletableFuture`，当所有给定的 `CompletableFuture` 都完成（不论成功与否）时，新 `CompletableFuture` 完成。

#### b. `anyOf(CompletableFuture<?>... cfs)`

创建一个新的 `CompletableFuture`，当任意一个给定的 `CompletableFuture` 完成时，新 `CompletableFuture` 完成。

## 实战应用

 `CompletableFuture` 的使用场景很广泛，例如
1. 异步数据库查询与结果合并
2. 微服务间异步通信
3. 并行任务执行与结果汇总
4. 异步事件处理与通知

这里以第一个场景举例：
**场景**：在一个订单处理系统中，需要查询订单的详细信息、关联的商品信息以及用户的个人信息。为减少查询延迟，可以使用 `CompletableFuture` 对每个查询进行异步执行，并在所有查询完成后合并结果。
**示例：**

如果我们不使用Java8提供的这个`CompletableFuture` 来实现
```java

@Service
@RequiredArgsConstructor
public class OrderProcessingServiceLegacy {
    private final OrderRepository orderRepo;
    private final ProductRepository productRepo;
    private final UserRepository userRepo;

    public OrderDetails fetchOrderDetails(int orderId) throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(3);
        CountDownLatch orderLatch = new CountDownLatch(1);
        CountDownLatch productsLatch = new CountDownLatch(1);
        CountDownLatch userLatch = new CountDownLatch(1);

        Order order = null;
        List<Product> products = null;
        User user = null;

        // 异步查询订单
        executor.execute(() -> {
            try {
                order = orderRepo.findOrderById(orderId);
                orderLatch.countDown();
            } finally {
                productsLatch.countDown();
            }
        });

        // 异步查询商品
        executor.execute(() -> {
            try {
                products = productRepo.findProductsByOrderId(orderId);
            } finally {
                productsLatch.countDown();
            }
        });

        // 异步查询用户（等待订单查询完成后再执行）
        executor.execute(() -> {
            try {
                orderLatch.await(); // 确保订单查询已完成
                user = userRepo.findUserById(order.getCustomerId());
            } finally {
                userLatch.countDown();
            }
        });

        // 等待所有查询完成
        userLatch.await();

        return new OrderDetails(order, products, user);
    }

    // ... 其他方法 ...
	@Data
	@AllArgsConstructor 
    public static class OrderDetails {
        private final Order order;
        private final List<Product> products;
        private final User user;
    }
}
```

**使用`CompletableFuture`实现** 
```java
@Service
@RequiredArgsConstructor
public class OrderProcessingService {
    private final OrderRepository orderRepo;
    private final ProductRepository productRepo;
    private final UserRepository userRepo;
	private final ThreadPoolExecutor executor;


    public CompletableFuture<OrderDetails> fetchOrderDetails(int orderId) {

        CompletableFuture<Order> orderFuture = CompletableFuture.supplyAsync(() -> orderRepo.findOrderById(orderId), executor);
        CompletableFuture<List<Product>> productsFuture = CompletableFuture.supplyAsync(() -> productRepo.findProductsByOrderId(orderId), executor);
        CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> userRepo.findUserById(order.getCustomerId()), executor);

        return CompletableFuture.allOf(orderFuture, productsFuture, userFuture)
                .thenApplyAsync(unused -> {
                    Order order = orderFuture.join();
                    List<Product> products = productsFuture.join();
                    User user = userFuture.join();

                    return new OrderDetails(order, products, user);
                }, executor);
    }

    // ... 其他方法 ...
	@Data
    public static class OrderDetails {
        private final Order order;
        private final List<Product> products;
        private final User user;

    }
}
```

在这个示例中：
- 使用了`CompletableFuture`之后，代码量减少了，整洁度和可读性也得到提高。
- `fetchOrderDetails` 方法接受一个订单 ID，使用 `CompletableFuture.supplyAsync()` 异步查询订单、商品和用户信息。
- 使用 `CompletableFuture.allOf()` 监控所有查询的完成状态。
- 当所有查询完成时，使用 `thenApplyAsync()` 合并结果，创建一个包含完整订单详情的 `OrderDetails` 对象。

## 小结
`CompletableFuture` 作为 Java 8 引入的重要异步编程工具，极大地提升了 Java 平台在应对高并发、高性能场景的能力。
结合 Java 8 的并行流（`Stream.parallel()`）与 `CompletableFuture`，可以轻松实现数据集的并行处理和结果聚合。
下次给大家聊聊`Stream.parallel()`。
