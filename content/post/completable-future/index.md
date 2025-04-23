---
title: 'CompletableFuture'
slug: 'completable-future'
description: CompletableFuture 介绍
date: 2025-04-23T15:26:47+08:00
categories:
  - Dev
tags:
  - Java
  - Thread
---

`CompletableFuture` 是 Java 8 引入的增强版异步编程工具，它在 `Future` 的基础上做了重大改进，用于更优雅、更灵活地处理异步任务、回调链式操作、组合多个异步任务等。

<!--more-->

## 为什么会冒出个 CompletableFuture

先来回答一个关键问题：为什么会有 `CompletableFuture`？既然 Java 专门搞了这么个新工具，肯定是因为老的方式不够好用，甚至在实际开发中会让人抓狂。

在 Java 8 之前，我们处理异步任务主要靠 `Future` 接口，比如在线程池中提交一个任务，代码大概是这样的：

```java
Future<String> future = executor.submit(() -> "hello");
String result = future.get(); // 阻塞等待结果
```

看着挺朴素的代码，其实问题不少：

1. **get() 是阻塞的**：任务没执行完，调用方就只能干等，完全不是我们想要的“异步”
2. **没有回调机制**：任务完成之后，不能自动触发后续操作，只能主动去问它“你好了没”
3. **不能链式组合多个异步任务**：比如 A 完成后继续执行 B，Future 根本做不到
4. **异常处理很笨重**：要靠开发者自己 try-catch，还没法像流水线一样优雅地传递异常
5. **没法聚合多个 Future 的结果**：比如等 A 和 B 都完成后再执行 C？不好意思，不支持

于是，Java 8 推出了 `CompletableFuture`，一次对异步能力的全面升级。从命名上就能看出它的野心：这是一个可以“被主动完成”的未来，不再是等待，而是掌控。

它不仅支持非阻塞调用，还有完善的回调机制、链式任务组合、异常处理流转，以及多个任务的聚合能力。

简而言之，`CompletableFuture` 把异步这件事，从“能用”提升到了“好用”，不止是更强大，而是更适合写出清晰、优雅、可维护的异步代码。

## CompletableFuture 的方法

`CompletableFuture` 对象的方法非常非常多，如何了解和记忆是个老大难了，这里推荐使用分组、对比的方式进行记忆。

### 创建方法

```java
// 创建一个已完成的 Future
public static <U> CompletableFuture<U> completedFuture(U value) {
}

// 无返回值、默认线程池（ForkJoinPool.commonPool()）
public static CompletableFuture<Void> runAsync(Runnable runnable) {
}

// 无返回值、自定义线程池
public static CompletableFuture<Void> runAsync(Runnable runnable,
                                               Executor executor) {
}

// 有返回值、默认线程池（ForkJoinPool.commonPool()）
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
}

// 有返回值、自定义线程池
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,
                                                   Executor executor) {
}

```

示例：

```java
CompletableFuture<String> completedFuture =
    CompletableFuture
        .completedFuture("CompletableFuture (with the given value)");

CompletableFuture<Void> runAsyncFuture =
    CompletableFuture.runAsync(() -> {
        System.out.println("Running async task (with default executor)");
    });
CompletableFuture<Void> runAsyncFuture =
    CompletableFuture.runAsync(() -> {
        System.out.println("Running async task with custom executor");
    }, executor);

CompletableFuture<String> supplyAsyncFuture =
    CompletableFuture.supplyAsync(() -> {
        return "Result of the computation (with default executor)";
    });
CompletableFuture<String> supplyAsyncFuture =
    CompletableFuture.supplyAsync(() -> {
        return "Result of the computation with custom executor";
    }, executor);
```

### 结果处理对比

#### 转换类方法

```java
// 同步转换结果
public <U> CompletableFuture<U> thenApply(
    Function<? super T,? extends U> fn) {
}

// 异步转换结果(默认线程池)
public <U> CompletableFuture<U> thenApplyAsync(
    Function<? super T,? extends U> fn) {
}

// 异步转换结果(指定线程池)
public <U> CompletableFuture<U> thenApplyAsync(
    Function<? super T,? extends U> fn, Executor executor) {
}

```

示例：

```java
CompletableFuture<Integer> lengthFuture =
    supplyAsyncFuture.thenApply(s -> s.length());
CompletableFuture<Integer> lengthFuture =
    supplyAsyncFuture.thenApplyAsync(s -> s.length());
CompletableFuture<Integer> lengthFuture =
    supplyAsyncFuture.thenApplyAsync(s -> s.length(), executor);

```

#### 消费类方法

```java
// 同步消费结果
public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
}

// 异步消费结果(默认线程池)
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action) {
}

// 异步消费结果(指定线程池)
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action,
                                                Executor executor) {
}

// 同步执行无参操作
public CompletableFuture<Void> thenRun(Runnable action) {
}

// 异步执行无参操作(默认线程池)
public CompletableFuture<Void> thenRunAsync(Runnable action) {
}

// 异步执行无参操作(指定线程池)
public CompletableFuture<Void> thenRunAsync(Runnable action,
                                            Executor executor) {
}

```

示例：

```java
supplyAsyncFuture.thenAccept(result ->
        System.out.println("Received: " + result)
    );
supplyAsyncFuture.thenAcceptAsync(result ->
        System.out.println("Received: " + result)
    );
supplyAsyncFuture.thenAcceptAsync(result ->
        System.out.println("Received: " + result), executor
    );

supplyAsyncFuture.thenRun(() ->
        System.out.println("Task completed")
    );
supplyAsyncFuture.thenRunAsync(() ->
        System.out.println("Task completed")
    );
supplyAsyncFuture.thenRunAsync(() ->
        System.out.println("Task completed"), executor
    );

```

### 组合方法对比

#### 双 Future 组合

```java
// 合并两个结果，并返回新的 CompletableFuture
public <U> CompletableFuture<U> thenCompose(
    Function<? super T, ? extends CompletionStage<U>> fn) {
}

public <U> CompletableFuture<U> thenComposeAsync(
    Function<? super T, ? extends CompletionStage<U>> fn) {
}

public <U> CompletableFuture<U> thenComposeAsync(
    Function<? super T, ? extends CompletionStage<U>> fn,
    Executor executor) {
}

// 消费两个 Future 的结果
public <U> CompletableFuture<Void> thenAcceptBoth(
    CompletionStage<? extends U> other,
    BiConsumer<? super T, ? super U> action) {
}

public <U> CompletableFuture<Void> thenAcceptBothAsync(
    CompletionStage<? extends U> other,
    BiConsumer<? super T, ? super U> action) {
}

public <U> CompletableFuture<Void> thenAcceptBothAsync(
    CompletionStage<? extends U> other,
    BiConsumer<? super T, ? super U> action, Executor executor) {
}

// 两个都完成后执行操作
public CompletableFuture<Void> runAfterBoth(CompletionStage<?> other,
                                            Runnable action) {
}

public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other,
                                                  Runnable action) {
}

public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other,
                                                  Runnable action,
                                                  Executor executor) {
}
```

示例：

```java
CompletableFuture<String> supplyAsyncFuture1 =
    CompletableFuture.supplyAsync(() -> {
        return "R1";
    });
CompletableFuture<String> supplyAsyncFuture2 =
    CompletableFuture.supplyAsync(() -> {
        return "R2";
    });

CompletableFuture<String> result = supplyAsyncFuture1.thenCombine(supplyAsyncFuture2,
        (r1, r2) -> String.format("%s | %s", r1, r2)
    );
result.thenAccept(System.out::println);

supplyAsyncFuture1.thenAcceptBoth(supplyAsyncFuture2,
        (r1, r2) -> System.out.println("%s | %s", r1, r2)
    );


CompletableFuture<String> runAsyncFuture1 =
    CompletableFuture.runAsync(() -> {
        System.out.println("run task 1")
    });
CompletableFuture<String> runAsyncFuture2 =
    CompletableFuture.runAsync(() -> {
        System.out.println("run task 2")
    });

runAsyncFuture1.runAfterBoth(runAsyncFuture2,
        () -> System.out.println("run task. ")
    );
```

#### 多 Future 组合

```java
// 所有 Future 完成后完成
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs) {
}

// 任意一个 Future 完成后完成
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs) {
}
```

示例：

```java
// 创建三个异步任务
CompletableFuture<String> task1 = CompletableFuture.supplyAsync(() -> "R1");
CompletableFuture<String> task2 = CompletableFuture.supplyAsync(() -> "R2");
CompletableFuture<String> task3 = CompletableFuture.supplyAsync(() -> "R3");

// 等待所有任务完成
CompletableFuture<Void> allFutures = CompletableFuture.allOf(task1, task2, task3);

// 所有完成后获取结果
allFutures.thenRun(() -> {
    try {
        System.out.println("T1 Result: " + task1.get());
        System.out.println("T2 Result: " + task2.get());
        System.out.println("T3 Result: " + task3.get());
    } catch (Exception e) {
        e.printStackTrace();
    }
});


// 模拟不同响应时间的服务
CompletableFuture<String> fastService = CompletableFuture.supplyAsync(() -> {
    sleep(100); // 模拟100ms延迟
    return "Fast service response";
});
CompletableFuture<String> mediumService = CompletableFuture.supplyAsync(() -> {
    sleep(300);
    return "Medium service response";
});
CompletableFuture<String> slowService = CompletableFuture.supplyAsync(() -> {
    sleep(500);
    return "Slow service response";
});
// 获取最先响应的服务
CompletableFuture<Object> firstResponse = CompletableFuture.anyOf(
    fastService, mediumService, slowService
);
firstResponse.thenAccept(result ->
    System.out.println("The first results: " + result)
);
// 可能输出: "The first results: Fast service response"
```

### 异常处理对比

```java
// 捕获异常并返回默认值
public CompletableFuture<T> exceptionally(
    Function<Throwable, ? extends T> fn) {
}

// 无论成功失败都会执行
public <U> CompletableFuture<U> handle(
    BiFunction<? super T, Throwable, ? extends U> fn) {
}

public <U> CompletableFuture<U> handleAsync(
    BiFunction<? super T, Throwable, ? extends U> fn) {
}

public <U> CompletableFuture<U> handleAsync(
    BiFunction<? super T, Throwable, ? extends U> fn, Executor executor) {
}

// 无论成功失败都会执行，不影响结果
public CompletableFuture<T> whenComplete(
    BiConsumer<? super T, ? super Throwable> action) {
}

public CompletableFuture<T> whenCompleteAsync(
    BiConsumer<? super T, ? super Throwable> action) {
}

public CompletableFuture<T> whenCompleteAsync(
    BiConsumer<? super T, ? super Throwable> action, Executor executor) {
}

```

示例：

```java
CompletableFuture.supplyAsync(() -> {
    if (Math.random() > 0.5) {
        throw new RuntimeException("Error");
    }
    return "Success";
})
.exceptionally(ex -> {
    System.out.println("Exception: " + ex.getMessage());
    return "Recovered";
})
.thenAccept(System.out::println);

CompletableFuture.supplyAsync(() -> {
    return "Result";
})
.handle((result, ex) -> {
    if (ex != null) {
        return "Default value";
    }
    return result;
});

CompletableFuture.supplyAsync(() -> {
    if (new Random().nextBoolean()) {
        throw new RuntimeException("Error");
    }
    return "Success";
})
.whenComplete((result, ex) -> {
    if (ex != null) {
        log.error("Error", ex);
        metrics.increment("failures");
    } else {
        log.info("Result: {}", result);
        metrics.increment("success");
    }
})
.thenAccept(System.out::println);

```

## 总结

1. 方法名中的 `Async` 后缀，代表异步执行
2. 参数列表中的 `Executor` 表示可以自定义线程池，如果不自定义，则使用默认线程池(`ForkJoinPool.commonPool()`)
3. 创建阶段的两个方法：`[run|supply](Async)`
   - `run`: 无返回值
   - `supple`: 有返回值
4. 结果处理阶段三个关键字 `then[Apply|Accept|Run](Async)`
   - `apply`: `Function<T,R>` 接收一个参数，处理后返回结果
   - `accept`: `Consumer<T>` 接收一个参数，不返回
   - `run`: `Runnable` 不接受参数，不返回
5. 组合多个分为两类，两个组合和多个组合

   - 两个组合 `then[Compose|AcceptBoth](Async)` 和 `runAfterBoth(Async)`

     - `compose`: `Function<T,R>` 合并两个结果，处理后返回结果
     - `acceptBoth`: `BiConsumer<T, U>` 消费两个结果，不返回
     - `runAfterBoth`: `Runnable` 不接受参数，不返回

   - 多个组合 `[all|any]Of`

     - `all`: 所有 Future 完成后完成
     - `any`: 任意一个 Future 完成后完成

6. 异常处理分为三部分 `exceptionally` `handle(Async)` 和 `whenComplete(Async)`
   - `exceptionally`: `Function<Throwable, T>` 接收异常为参数，处理后返回一个值
   - `handle`: `BiFunction<T, Throwable, U>` 可以得到异常和结果后，重新返回一个结果
   - `whenComplete`: `BiConsumer<T, Throwable>` 可以得到异常和返回值，不对结果进行处理
7. 注意： `handle` 和 `whenComplete` 比较特殊，无论是否发生异常，都会处理（类似 `finally`)
