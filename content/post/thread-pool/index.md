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
