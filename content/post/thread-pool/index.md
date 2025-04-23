---
title: 'Java 线程池'
slug: 'thread-pool'
description: Java 线程池详细分析
date: 2025-02-12T14:15:39+08:00
image:
categories:
  - Dev
tags:
  - Java
  - Thread
---

详细分析 Java 线程池参数、如何构建线程池，早消线程。

<!--more-->

## 线程池基础

### 线程池主要参数

- `corePoolSize(int)[> 0]`: 核心线程池数，池中保留的线程数，即使它们处于空闲状态，除非设置了 allowCoreThreadTimeOut 为 true
- `maximumPoolSize(int)[> 0]`: 最大线程数，线程池中最大可创建的线程数
- `keepAliveTime(long)[> 0]`: 线程存活时间，当线程数大于核心线程数时，多余的空闲线程在终止前等待新任务的最长时间
- `unit(TimeUnit)`: keepAliveTime 参数的时间单位
- `workQueue(BlockingQueue<Runnable>)[not null]`: 用于在执行任务之前保存任务的队列
- `threadFactory(ThreadFactory)[not null]`: 执行器创建新线程时使用的工厂
- `handler(RejectedExecutionHandler)[not null]`: 当线程边界和队列容量达到上限时，用于处理阻塞执行的处理器

#### (corePoolSize)核心线程数

如果提交任务后线程还在运行，当线程数小于 `corePoolSize` 值时，无论线程池中的线程是否忙碌，都会创建线程，并把任务交给此新创建的线程进行处理，如果线程数少于等于 `corePoolSize`，那么这些线程不会回收，除非将 `allowCoreThreadTimeOut` 设置为 `true`，但一般不这么干，因为频繁地创建销毁线程会极大地增加系统调用的开销。

计算方式：

通过 Runtime 方法来获取当前服务器 CPU 内核数量，根据 CPU 内核数量来创建核心线程数和最大线程数。

一般根据任务类型会有以下两种划分方式：

- CPU 密集型：核心线程数 = CPU 核心数 + 1
- I/O 密集型：核心线程数 = CPU 核心数 \* 2

```java
private Integer calculateCoreNum() {
    int cpuCoreNum = Runtime.getRuntime().availableProcessors();
    return new BigDecimal(cpuCoreNum).divide(new BigDecimal("0.5")).intValue();
}
```

#### (maximumPoolSize)最大线程数

线程池中最大可创建的线程数，如果提交任务时队列满了且线程数未到达这个设定值，则会创建线程并执行此次提交的任务，如果提交任务时队列满了但线池数已经到达了这个值，此时说明已经超出了线池程的负载能力，就会执行拒绝策略，不能让源源不断地任务进来把线程池给压垮了吧，首先要保证线程池能正常工作。

#### (handler)拒绝策略

默认有以下四种拒绝策略：

1. `AbortPolicy`：丢弃任务并抛出异常，这也是默认策略
2. `CallerRunsPolicy`：用调用者所在的线程来执行任务，如果用的是 CallerRunsPolicy 策略，提交任务的线程（比如主线程）提交任务后并不能保证马上就返回，当触发了这个 reject 策略不得不亲自来处理这个任务
3. `DiscardOldestPolicy`：丢弃阻塞队列中靠最前的任务，并执行当前任务
4. `DiscardPolicy`：直接丢弃任务，不抛出任何异常，这种策略只适用于不重要的任务

自定义拒绝策略：

```java
implements RejectedExecutionHandler
```

> 注意：自定义逻辑之后，需要抛出异常。

#### 线程执行过程

当一个任务通过 execute(Runnable) 方法欲添加到线程池时：

1. 如果当前线程池中的线程数量小于 corePoolSize, 即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。
2. 如果当前线程池中的线程数量等于 corePoolSize, 但是缓冲队列 workQueue 未满，则任务被放入缓冲队列。
3. 如果此时线程池中的线程数量大于 corePoolSize, 缓冲队列 workQueue 已满，并且线程池中的线程数量小于 maxPoolSize, 则新建线程用来处理被添加的任务。
4. 如果此时线程池中的线程数量大于 corePoolSize, 缓冲队列 workQueue 已满，并且线程池中的线程数量等于 maxPoolSize, 那么通过 rejectedExecutionHandler 所指定的策略来处理此任务。

也就是整体优先级为： 核心线程 corePoolSize -> 任务队列 workQueue -> 最大线程 maxPoolSize -> rejectedExecutionHandler 拒绝策略

#### 为什么不允许使用 Executors 快速创建线程池

使用 Executors 快速生成的线程池没有完整的参数配置，`newCachedThreadPool` 方法的最大线程数设置成了 `Integer.MAX_VALUE`，而 `newSingleThreadExecutor` 方法创建 `workQueue` 时 `LinkedBlockingQueue` 未声明大小，相当于创建了无界队列，一不小心就会导致 OOM。

## 快消线程池

根据线程池的执行过程，核心线程满了之后，新增的任务会被放入队列中。

而快消线程池就是，发现池内线程大于核心线程数，不放入阻塞队列，而是创建非核心线程进行消费任务。

### TaskQueue

```java
import lombok.Setter;

import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.RejectedExecutionException;
import java.util.concurrent.TimeUnit;

/**
 * 快速消费消息队列
 */
public class TaskQueue<R extends Runnable> extends LinkedBlockingQueue<Runnable> {
    @Setter
    private EagerThreadPoolExecutor executor;

    public TaskQueue(int capacity) {
        super(capacity);
    }

    @Override
    public boolean offer(Runnable runnable) {
        // 获取线程池中线程数
        int currentPoolThreadSize = executor.getPoolSize();
        // 如果有核心线程正在空闲，将任务加入阻塞队列，由核心线程进行处理任务
        if (executor.getSubmittedTaskCount() < currentPoolThreadSize) {
            return super.offer(runnable);
        }
        // 当前线程池线程数量小于最大线程数，返回 False，根据线程池源码，会创建非核心线程
        if (currentPoolThreadSize < executor.getMaximumPoolSize()) {
            return false;
        }
        // 如果当前线程池数量大于最大线程数，任务加入阻塞队列
        return super.offer(runnable);
    }

    public boolean retryOffer(Runnable o, long timeout, TimeUnit unit) throws InterruptedException {
        if (executor.isShutdown()) {
            throw new RejectedExecutionException("Executor is shutdown!");
        }
        // 如果当前线程池数量大于最大线程数，任务加入阻塞队列
        return super.offer(o, timeout, unit);
    }
}
```

通过获取线程池中的任务数量进行响应的操作。

### EagerThreadPoolExecutor

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.RejectedExecutionException;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 快速消费线程池
 */
public class EagerThreadPoolExecutor extends ThreadPoolExecutor {

    public EagerThreadPoolExecutor(int corePoolSize,
                                   int maximumPoolSize,
                                   long keepAliveTime,
                                   TimeUnit unit,
                                   BlockingQueue<Runnable> workQueue,
                                   ThreadFactory threadFactory,
                                   RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
    }

    private final AtomicInteger submittedTaskCount = new AtomicInteger(0);

    public int getSubmittedTaskCount() {
        return submittedTaskCount.get();
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        submittedTaskCount.decrementAndGet();
    }

    @Override
    public void execute(Runnable command) {
        submittedTaskCount.incrementAndGet();
        try {
            super.execute(command);
        } catch (RejectedExecutionException ex) {
            TaskQueue taskQueue = (TaskQueue) super.getQueue();
            try {
                if (!taskQueue.retryOffer(command, 0, TimeUnit.MILLISECONDS)) {
                    submittedTaskCount.decrementAndGet();
                    throw new RejectedExecutionException("Queue capacity is full.", ex);
                }
            } catch (InterruptedException iex) {
                submittedTaskCount.decrementAndGet();
                throw new RejectedExecutionException(iex);
            }
        } catch (Exception ex) {
            submittedTaskCount.decrementAndGet();
            throw ex;
        }
    }
}
```

`EagerThreadPoolExecutor` 继承了 `ThreadPoolExecutor`，在 `execute()` 上做了自己的逻辑处理。

维护了一个任务数量的字段，是一个原子类，添加任务时自增，任务异常及结束时递减。

这样就能保证 `TaskQueue#offer(Runnable runnable)` 做出逻辑处理

## 构建线程池

### Builder

```java
import java.io.Serializable;

/**
 * Builder 模式抽象接口
 *
 * @param <T>
 */
public interface Builder<T> extends Serializable {

    /**
     * 构建方法
     *
     * @return 构建后的对象
     */
    T build();
}

```

### ThreadFactoryBuilder

```java
import lombok.AccessLevel;
import lombok.NoArgsConstructor;

import javax.validation.constraints.NotNull;
import java.io.Serial;
import java.util.Objects;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicLong;

/**
 * 线程工厂构建器，提供灵活的线程配置选项。
 * <p>
 * 支持设置线程名前缀、守护线程状态、优先级和未捕获异常处理器等配置。
 * 采用建造者模式，支持链式调用。
 * </p>
 */
// 私有构造方法，只允许使用 builder() 方法创建实例。
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public final class ThreadFactoryBuilder implements Builder<ThreadFactory> {

    @Serial
    private static final long serialVersionUID = 1L;

    private ThreadFactory backingThreadFactory;
    private String namePrefix;
    private Boolean daemon;
    private Integer priority;
    private Thread.UncaughtExceptionHandler uncaughtExceptionHandler;


    /**
     * 创建新的ThreadFactoryBuilder实例。
     *
     * @return 新的ThreadFactoryBuilder实例
     */
    public static ThreadFactoryBuilder builder() {
        return new ThreadFactoryBuilder();
    }

    /**
     * 设置底层线程工厂。如果未设置，将使用 Executors.defaultThreadFactory()。
     *
     * @param backingThreadFactory 底层线程工厂
     * @return 当前构建器实例
     * @throws NullPointerException 如果 backingThreadFactory 为 null
     */
    public ThreadFactoryBuilder threadFactory(@NotNull ThreadFactory backingThreadFactory) {
        this.backingThreadFactory = Objects.requireNonNull(backingThreadFactory,
                "backingThreadFactory cannot be null");
        return this;
    }

    /**
     * 设置线程名前缀。如果设置，线程名将为 "前缀_序号" 格式。
     *
     * @param namePrefix 线程名前缀
     * @return 当前构建器实例
     */
    public ThreadFactoryBuilder prefix(String namePrefix) {
        this.namePrefix = namePrefix;
        return this;
    }

    /**
     * 设置是否为守护线程。
     *
     * @param daemon 是否为守护线程
     * @return 当前构建器实例
     */
    public ThreadFactoryBuilder daemon(boolean daemon) {
        this.daemon = daemon;
        return this;
    }

    /**
     * 设置线程优先级。
     *
     * @param priority 线程优先级(1-10)
     * @return 当前构建器实例
     * @throws IllegalArgumentException 如果优先级不在 Thread.MIN_PRIORITY(1) 和 Thread.MAX_PRIORITY(10) 之间
     */
    public ThreadFactoryBuilder priority(int priority) {
        if (priority < Thread.MIN_PRIORITY) {
            throw new IllegalArgumentException(String.format("Thread priority (%s) must be >= %s",
                    priority, Thread.MIN_PRIORITY));
        }
        if (priority > Thread.MAX_PRIORITY) {
            throw new IllegalArgumentException(String.format("Thread priority (%s) must be <= %s",
                    priority, Thread.MAX_PRIORITY));
        }
        this.priority = priority;
        return this;
    }

    /**
     * 设置未捕获异常处理器。
     *
     * @param uncaughtExceptionHandler 未捕获异常处理器
     * @return 当前构建器实例
     */
    public ThreadFactoryBuilder uncaughtExceptionHandler(Thread.UncaughtExceptionHandler uncaughtExceptionHandler) {
        this.uncaughtExceptionHandler = uncaughtExceptionHandler;
        return this;
    }

    /**
     * 构建配置好的 ThreadFactory 实例。
     *
     * @return 配置好的 ThreadFactory 实例
     */
    @Override
    public ThreadFactory build() {
        return build(this);
    }

    /**
     * 内部构建方法，根据配置创建 ThreadFactory。
     *
     * @param builder 构建器实例
     * @return 配置好的 ThreadFactory
     */
    private static ThreadFactory build(ThreadFactoryBuilder builder) {
        final ThreadFactory backingThreadFactory = (null != builder.backingThreadFactory)
                ? builder.backingThreadFactory
                : Executors.defaultThreadFactory();

        final String namePrefix = builder.namePrefix;
        final Boolean daemon = builder.daemon;
        final Integer priority = builder.priority;
        final Thread.UncaughtExceptionHandler handler = builder.uncaughtExceptionHandler;
        final AtomicLong count = (null == namePrefix) ? null : new AtomicLong();

        return r -> {
            final Thread thread = backingThreadFactory.newThread(r);
            // 设置线程名称(如果配置了前缀)
            if (null != namePrefix) {
                thread.setName(namePrefix + "_" + count.getAndIncrement());
            }
            // 设置守护线程状态(如果配置)
            if (null != daemon) {
                thread.setDaemon(daemon);
            }
            // 设置线程优先级(如果配置)
            if (null != priority) {
                thread.setPriority(priority);
            }
            // 设置未捕获异常处理器(如果配置)
            if (null != handler) {
                thread.setUncaughtExceptionHandler(handler);
            }
            return thread;
        };
    }
}

```

### ThreadPoolBuilder

```java
import lombok.AccessLevel;
import lombok.NoArgsConstructor;
import org.springframework.util.Assert;

import java.math.BigDecimal;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * 线程池构建器，提供链式API配置并创建ThreadPoolExecutor实例。
 * 这是一个不可变构建器，线程安全。
 */
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public final class ThreadPoolBuilder implements Builder<ThreadPoolExecutor> {

    // 默认核心线程数为CPU核心数的5倍（基于20%利用率计算）
    private int corePoolSize = calculateCoreNum();

    // 默认最大线程数为核心线程数的 1.5 倍
    private int maximumPoolSize = corePoolSize + (corePoolSize >> 1);

    // 默认线程空闲存活时间 30 秒
    private long keepAliveTime = 30_000L;

    // 默认时间单位为毫秒
    private TimeUnit timeUnit = TimeUnit.MILLISECONDS;

    // 默认工作队列为无界队列，容量 4096
    private BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(4096);

    // 默认拒绝策略为 AbortPolicy
    private RejectedExecutionHandler rejectedExecutionHandler = new ThreadPoolExecutor.AbortPolicy();

    // 默认线程为非守护线程
    private boolean isDaemon = false;

    // 线程名前缀
    private String threadNamePrefix;

    // 线程工厂
    private ThreadFactory threadFactory;

    /**
     * 创建 ThreadPoolBuilder 实例的工厂方法
     *
     * @return 新的 ThreadPoolBuilder 实例
     */
    public static ThreadPoolBuilder builder() {
        return new ThreadPoolBuilder();
    }

    /**
     * 计算默认核心线程数，基于 CPU 核心数和 20% 利用率
     *
     * @return 计算后的核心线程数
     */
    private int calculateCoreNum() {
        int cpuCoreNum = Runtime.getRuntime().availableProcessors();
        return new BigDecimal(cpuCoreNum).divide(new BigDecimal("0.2")).intValue();
    }

    /**
     * 设置自定义线程工厂
     *
     * @param threadFactory 线程工厂实例
     * @return 当前构建器实例（链式调用）
     */
    public ThreadPoolBuilder threadFactory(ThreadFactory threadFactory) {
        this.threadFactory = threadFactory;
        return this;
    }

    /**
     * 设置核心线程数
     *
     * @param corePoolSize 核心线程数
     * @return 当前构建器实例（链式调用）
     * @throws IllegalArgumentException 如果 corePoolSize 小于0
     */
    public ThreadPoolBuilder corePoolSize(int corePoolSize) {
        if (corePoolSize < 0) {
            throw new IllegalArgumentException("Core pool size must be non-negative");
        }
        this.corePoolSize = corePoolSize;
        return this;
    }

    /**
     * 设置最大线程数，如果小于当前核心线程数则自动调整核心线程数
     *
     * @param maximumPoolSize 最大线程数
     * @return 当前构建器实例（链式调用）
     * @throws IllegalArgumentException 如果 maximumPoolSize 小于等于 0 或小于核心线程数
     */
    public ThreadPoolBuilder maximumPoolSize(int maximumPoolSize) {
        if (maximumPoolSize <= 0) {
            throw new IllegalArgumentException("Maximum pool size must be positive");
        }
        this.maximumPoolSize = maximumPoolSize;
        if (maximumPoolSize < this.corePoolSize) {
            this.corePoolSize = maximumPoolSize;
        }
        return this;
    }

    /**
     * 设置线程工厂的基本属性
     *
     * @param threadNamePrefix 线程名前缀
     * @param isDaemon         是否为守护线程
     * @return 当前构建器实例（链式调用）
     */
    public ThreadPoolBuilder threadFactory(String threadNamePrefix, Boolean isDaemon) {
        this.threadNamePrefix = threadNamePrefix;
        this.isDaemon = isDaemon;
        return this;
    }

    /**
     * 设置线程空闲存活时间（使用默认时间单位毫秒）
     *
     * @param keepAliveTime 存活时间
     * @return 当前构建器实例（链式调用）
     */
    public ThreadPoolBuilder keepAliveTime(long keepAliveTime) {
        this.keepAliveTime = keepAliveTime;
        return this;
    }

    /**
     * 设置线程空闲存活时间和时间单位
     *
     * @param keepAliveTime 存活时间
     * @param timeUnit      时间单位
     * @return 当前构建器实例（链式调用）
     */
    public ThreadPoolBuilder keepAliveTime(long keepAliveTime, TimeUnit timeUnit) {
        this.keepAliveTime = keepAliveTime;
        this.timeUnit = timeUnit;
        return this;
    }

    /**
     * 设置拒绝策略
     *
     * @param rejectedExecutionHandler 拒绝策略处理器
     * @return 当前构建器实例（链式调用）
     */
    public ThreadPoolBuilder rejected(RejectedExecutionHandler rejectedExecutionHandler) {
        this.rejectedExecutionHandler = rejectedExecutionHandler;
        return this;
    }

    /**
     * 设置工作队列
     *
     * @param workQueue 工作队列实例
     * @return 当前构建器实例（链式调用）
     */
    public ThreadPoolBuilder workQueue(BlockingQueue<Runnable> workQueue) {
        this.workQueue = workQueue;
        return this;
    }

    /**
     * 构建ThreadPoolExecutor实例
     *
     * @return 配置好的ThreadPoolExecutor实例
     * @throws IllegalArgumentException 如果参数不合法或线程名前缀为空
     */
    @Override
    public ThreadPoolExecutor build() {
        if (threadFactory == null) {
            Assert.hasLength(threadNamePrefix, "The thread name prefix cannot be empty or an empty string.");
            threadFactory = ThreadFactoryBuilder.builder().prefix(threadNamePrefix).daemon(isDaemon).build();
        }
        ThreadPoolExecutor executorService;
        try {
            executorService = new ThreadPoolExecutor(corePoolSize,
                    maximumPoolSize,
                    keepAliveTime,
                    timeUnit,
                    workQueue,
                    threadFactory,
                    rejectedExecutionHandler);
        } catch (IllegalArgumentException ex) {
            throw new IllegalArgumentException("Error creating thread pool parameter.", ex);
        }
        return executorService;
    }
}

```

### 使用方式

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.scheduling.annotation.EnableAsync;

import java.util.concurrent.Executor;

// @EnableAsync 注解可以放在配置类和启动类
// 如果简单使用，不用自己定义的线程池，放在启动类也行
// 如果使用了自定义线程池，推荐放在配置类，职责分离
@EnableAsync
@Configuration
public class ThreadPoolConfiguration {

    @Bean
    // 如果有多个线程池配置，需要配置一个默认的
    @Primary
    // 如果想要 @Async 注解使用当前线程池，需要注意，名称必须是 "taskExecutor"
    public Executor taskExecutor() {
        // 最简单的创建方式，指定线程前缀即可
        return ThreadPoolBuilder.builder()
                .threadFactory("d_task_", false)
                .build();
    }
}

```
