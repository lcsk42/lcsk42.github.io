---
title: 'Java Microbenchmark Harness'
description: 基准测试
date: 2023-12-10T10:20:18+08:00
categories:
  - Dev
tags:
  - JMH
  - Java
---

JMH is a Java harness for building, running, and analysing nano/micro/milli/macro benchmarks written in Java and other languages targeting the JVM.

JMH 是一个 Java 工具，用于构建、运行和分析用 Java 和其他针对 JVM 的语言编写的纳/微米/毫/宏观基准测试。

<!--more-->

## JMH 基础概念

- Iteration: Iteration 是 JMH 进行测试的最小单位，包含一组 Invocations。
- Invocation: 一次 Benchmark 方法调用。
- Operation: Benchmark 方法中，被测量操作的执行。如果被测试的操作在 Benchmark 方法中循环执行，可以使用 `@OperationsPerInvocation` 表明循环次数，使测试结果为单次 Operation 的性能。
- Warmup: 在实际进行 Benchmark 前先进行预热。因为某个函数被调用多次之后，JIT 会对其进行编译，通过预热可以使测量结果更加接近真实情况。

## 如何开始

JMH 在 JDK 12 中已经被包含，低版本则需要自己在 Maven 中进行引入。

```xml

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <jmh.version>1.27</jmh.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.openjdk.jmh</groupId>
        <artifactId>jmh-core</artifactId>
        <version>${jmh.version}</version>
    </dependency>
    <dependency>
        <groupId>org.openjdk.jmh</groupId>
        <artifactId>jmh-generator-annprocess</artifactId>
        <version>${jmh.version}</version>
        <scope>provided</scope>
    </dependency>
</dependencies>

```

## 相关注解

### Warmup

```java
@Warmup(iterations = -1, time = -1, timeUnit = TimeUnit.SECONDS, batchSize = -1)
public class BenchmarkTest {}
```

- iterations: 预热多少轮。
- time: 预热时间。
- timeUnit: 时间单位，默认 秒。
- batchSize: 指定每次操作调用多少次方法。

对应输出中的以下部分：

```txt
# Warmup Iteration   1: 3359571890.253 ops/s
# Warmup Iteration   2: 3410327185.841 ops/s
# Warmup Iteration   3: 3397921599.396 ops/s
# Warmup Iteration   4: 3435385874.666 ops/s
# Warmup Iteration   5: 3418061628.223 ops/s
```

### Measurement

```java
@Measurement(iterations = -1, time = -1, timeUnit = TimeUnit.SECONDS, batchSize = -1)
public class BenchmarkTest {}
```

- iterations: 执行多少轮。
- time: 执行时间。
- timeUnit: 时间单位，默认 秒。
- batchSize: 指定每次操作调用多少次方法。

`Measurement` 和 `Warmup` 的参数是一样的。不同于预热，它指的是真正的迭代次数。

对应输出中的以下部分：

```txt
Iteration   1: 3439847797.303 ops/s
Iteration   2: 3450228067.947 ops/s
Iteration   3: 3440088656.138 ops/s
Iteration   4: 3427225995.315 ops/s
Iteration   5: 3455433144.375 ops/s
```

### BenchmarkMode

```java
@BenchmarkMode(Mode.All)
public class BenchmarkTest {}
```

基准测试类型:

- Throughput: Throughput, ops/time (吞吐量，单位时间内执行的次数)
- AverageTime: Average time, time/op（平均时间，一次执行需要的单位时间，其实是吞吐量的倒数）
- SampleTime: Sampling time（是基于采样的执行时间，采样频率由 JMH 自动控制，同时结果中也会统计出 p90、p95 的时间）
- SingleShotTime: Single shot invocation time （只运行一次，把 Warmup 次数设为 0，可以用于测试冷启动时的性能）
- All: All benchmark modes （所有模式）

### Fork

```java
@Fork(1)
public class BenchmarkTest {}
```

值一般设置为 1，表示使用一个进程进行测试。如果大于 1，则会启用新的进程进行测试。

值的注意的是，可以使用 `jvmArgs`,`jvmArgsPrepend`,`jvmArgsAppend` 来传递 JVM 相关的参数。

### Threads

```java
@Threads(Threads.MAX)
public class BenchmarkTest {}
```

`@Fork` 是面向进程的，而 `@Threads` 是面向线程的。指定了这个注解以后，将会开启并行测试。

如果设置了 `Threads.MAX` ，将会使用和处理机器核数相同的线程数。

### OutputTimeUnit

```java
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class BenchmarkTest {}
```

指定基准测试结果的时间类型。可以选择秒、毫秒、微秒等。

### State

```java
@State(Scope.Thread)
public class BenchmarkTest {}
```

指定了在类中变量的作用范围。可以用 Scope 参数用来表示该状态的共享范围。

Scope 有三个参数：

- Benchmark：表示变量的作用范围是某个基准测试类。
- Thread：每个线程一份副本，如果配置了 Threads 注解，则每个 Thread 都拥有一份变量，它们互不影响。
- Group：联系 `@Group` 注解，在同一个 Group 里，将会共享同一个变量实例。

### Param

```java
public class BenchmarkTest {
    @Param({"1", "31", "65", "101", "103"})
    public int arg;

    @Param({"0", "1", "2", "4", "8", "16", "32"})
    public int certainty;

    @Benchmark
    public boolean bench() {
        return BigInteger.valueOf(arg).isProbablePrime(certainty);
    }
}
```

属性注解，简单来说就是测试的时候将设置的各种值分别带入。

需要注意的是，如果你设置了非常多的参数，这些参数将执行多次，通常会运行很长时间。

比如参数 x 共 m 个，参数 y 共 n 个，那么总共要执行 m\*n 次。

### Group GroupThreads

```java
public class BenchmarkTest {

    @Group("group1")
    @GroupThreads(1)
    public void test() {}

}
```

`@Group` 注解只能加在方法上，用来把测试方法进行归类。

如果单个测试文件中方法很多，需要将其归类，则可以使用这个注解。

与之关联的 `@GroupThreads` 注解，会在这个归类的基础上，再进行一些线程方面的设置。

### Setup TearDown

```java
public class BenchmarkTest {

    @Setup(Level.Trial)
    public void init() {}

    @TearDown(Level.Trial)
    public void destory() {}

}
```

Setup 用于基准测试前的初始化动作,TearDown 用于基准测试后的动作,可以用来做一些全局的配置。

这两个注解，同样有一个 Level 值，标明了方法运行的时机，它有三个取值。

- Trial：默认的级别。也就是 Benchmark 级别。
- Iteration：每次迭代都会运行。
- Invocation：每次方法调用都会运行，这个是粒度最细的。

### Benchmark

```java
public class BenchmarkTest {
    @Benchmark
    public void test() {}

}
```

方法级注解，表示该方法是需要进行 benchmark 的对象，用法和 JUnit 的 @Test 类似。

## 开始测试

创建相关类标注完注解之后，可以直接在 main 方法中开始测试：

```java
public class BenchmarkTest {
    public static void main(String[] args) throws RunnerException {
        final Options options = new OptionsBuilder()
                .include(BenchmarkTest.class.getSimpleName())
                .build();
        new Runner(options).run();
    }
}
```

## Dead-Code Elimination (死码消除)

```java
public class DeadCode {

    /*
     * The downfall of many benchmarks is Dead-Code Elimination (DCE): compilers
     * are smart enough to deduce some computations are redundant and eliminate
     * them completely. If the eliminated part was our benchmarked code, we are
     * in trouble.
     */

    private double x = Math.PI;

    private double compute(double d) {
        for (int c = 0; c < 10; c++) {
            d = d * d / Math.PI;
        }
        return d;
    }

    @Benchmark
    public void baseline() {
        // do nothing, this is a baseline
    }

    @Benchmark
    public void measureWrong() {
        // This is wrong: result is not used and the entire computation is optimized away.
        compute(x);
    }

    @Benchmark
    public double measureRight() {
        // This is correct: the result is being used.
        return compute(x);
    }

}
```

```txt
Benchmark                           Mode  Cnt  Score   Error  Units
DeadCode.baseline      avgt    5  0.292 ± 0.008  ns/op
DeadCode.measureRight  avgt    5  7.374 ± 0.213  ns/op
DeadCode.measureWrong  avgt    5  0.293 ± 0.003  ns/op
```

Dead-Code Elimination (DCE) 死码消除，编译器非常聪明，上面的代码中，baseline() 和 measureWrong() 有着相同的效率,因为编译器发现有的代码没有作用，如上述 measureWrong() 并没有返回值，计算结果并没有被使用到，这时候为了效率，编译器会消除掉这段代码，但是对基准测试来说就很不友好。我们可以通过增加 return 的方法来避免编译期间代码被擦除掉。

另外一种解决 DCE 的方法是，JMH 提供了一个 Blackholes （黑洞），我们使用 Blackholes 吃掉(consume)返回值就好了。

```java
    @Benchmark
    public void measure(final Blackhole blackhole) {
        blackhole.consume(compute(x));
    }
```

## Constant Fold (常量折叠)

```java
public class ConstantFold {

    /*
     * The flip side of dead-code elimination is constant-folding.
     *
     * If JVM realizes the result of the computation is the same no matter what,
     * it can cleverly optimize it. In our case, that means we can move the
     * computation outside of the internal JMH loop.
     *
     * This can be prevented by always reading the inputs from non-final
     * instance fields of @State objects, computing the result based on those
     * values, and follow the rules to prevent DCE.
     */

    // IDEs will say "Oh, you can convert this field to local variable". Don't. Trust. Them.
    // (While this is normally fine advice, it does not work in the context of measuring correctly.)
    private double x = Math.PI;

    // IDEs will probably also say "Look, it could be final". Don't. Trust. Them. Either.
    // (While this is normally fine advice, it does not work in the context of measuring correctly.)
    private final double wrongX = Math.PI;

    private double compute(double d) {
        for (int c = 0; c < 10; c++) {
            d = d * d / Math.PI;
        }
        return d;
    }

    @Benchmark
    public double baseline() {
        // simply return the value, this is a baseline
        return Math.PI;
    }

    @Benchmark
    public double measureWrong_1() {
        // This is wrong: the source is predictable, and computation is foldable.
        return compute(Math.PI);
    }

    @Benchmark
    public double measureWrong_2() {
        // This is wrong: the source is predictable, and computation is foldable.
        return compute(wrongX);
    }

    @Benchmark
    public double measureRight() {
        // This is correct: the source is not predictable.
        return compute(x);
    }
}
```

```txt
Benchmark                                 Mode  Cnt  Score   Error  Units
ConstantFold.baseline        avgt    5  1.939 ± 0.100  ns/op
ConstantFold.measureRight    avgt    5  7.276 ± 0.042  ns/op
ConstantFold.measureWrong_1  avgt    5  1.896 ± 0.004  ns/op
ConstantFold.measureWrong_2  avgt    5  1.916 ± 0.010  ns/op
```

constant-folding (常量折叠)，上述代码的 measureWrong1 和 measureWrong2 中的运算都是可以预测的值，所以也会在编译期直接替换为计算结果，从而导致基准测试失败，注意 final 修饰的变量也会被折叠。

## Loops（循环）

```java
public class Loops {

    /*
     * It would be tempting for users to do loops within the benchmarked method.
     * (This is the bad thing Caliper taught everyone). These tests explain why
     * this is a bad idea.
     *
     * Looping is done in the hope of minimizing the overhead of calling the
     * test method, by doing the operations inside the loop instead of inside
     * the method call. Don't buy this argument; you will see there is more
     * magic happening when we allow optimizers to merge the loop iterations.
     */

    /*
     * Suppose we want to measure how much it takes to sum two integers:
     */

    int x = 1;
    int y = 2;

    /*
     * This is what you do with JMH.
     */

    @Benchmark
    public int measureRight() {
        return (x + y);
    }

    /*
     * The following tests emulate the naive looping.
     * This is the Caliper-style benchmark.
     */
    private int reps(int reps) {
        int s = 0;
        for (int i = 0; i < reps; i++) {
            s += (x + y);
        }
        return s;
    }

    /*
     * We would like to measure this with different repetitions count.
     * Special annotation is used to get the individual operation cost.
     */

    @Benchmark
    @OperationsPerInvocation(1)
    public int measureWrong_1() {
        return reps(1);
    }

    @Benchmark
    @OperationsPerInvocation(10)
    public int measureWrong_10() {
        return reps(10);
    }

    @Benchmark
    @OperationsPerInvocation(100)
    public int measureWrong_100() {
        return reps(100);
    }

    @Benchmark
    @OperationsPerInvocation(1_000)
    public int measureWrong_1000() {
        return reps(1_000);
    }

    @Benchmark
    @OperationsPerInvocation(10_000)
    public int measureWrong_10000() {
        return reps(10_000);
    }

    @Benchmark
    @OperationsPerInvocation(100_000)
    public int measureWrong_100000() {
        return reps(100_000);
    }

}
```

```txt
Benchmark                               Mode  Cnt  Score    Error  Units
Loops.measureRight         avgt    5  1.880 ±  0.014  ns/op
Loops.measureWrong_1       avgt    5  1.880 ±  0.009  ns/op
Loops.measureWrong_10      avgt    5  0.188 ±  0.001  ns/op
Loops.measureWrong_100     avgt    5  0.019 ±  0.001  ns/op
Loops.measureWrong_1000    avgt    5  0.022 ±  0.001  ns/op
Loops.measureWrong_10000   avgt    5  0.019 ±  0.001  ns/op
Loops.measureWrong_100000  avgt    5  0.020 ±  0.001  ns/op
```

不要在基准测试的时候使用循环，使用循环就会导致测试结果不准确。

## Blackhole

```java
public class ConsumeCPU {

    /*
     * At times you require the test to burn some of the cycles doing nothing.
     * In many cases, you *do* want to burn the cycles instead of waiting.
     *
     * For these occasions, we have the infrastructure support. Blackholes
     * can not only consume the values, but also the time! Run this test
     * to get familiar with this part of JMH.
     *
     * (Note we use static method because most of the use cases are deep
     * within the testing code, and propagating blackholes is tedious).
     */

    @Benchmark
    public void consume_0000() {
        Blackhole.consumeCPU(0);
    }

    @Benchmark
    public void consume_0001() {
        Blackhole.consumeCPU(1);
    }

    @Benchmark
    public void consume_0002() {
        Blackhole.consumeCPU(2);
    }

    @Benchmark
    public void consume_0004() {
        Blackhole.consumeCPU(4);
    }

    @Benchmark
    public void consume_0008() {
        Blackhole.consumeCPU(8);
    }

    @Benchmark
    public void consume_0016() {
        Blackhole.consumeCPU(16);
    }

    @Benchmark
    public void consume_0032() {
        Blackhole.consumeCPU(32);
    }

    @Benchmark
    public void consume_0064() {
        Blackhole.consumeCPU(64);
    }

    @Benchmark
    public void consume_0128() {
        Blackhole.consumeCPU(128);
    }

    @Benchmark
    public void consume_0256() {
        Blackhole.consumeCPU(256);
    }

    @Benchmark
    public void consume_0512() {
        Blackhole.consumeCPU(512);
    }

    @Benchmark
    public void consume_1024() {
        Blackhole.consumeCPU(1024);
    }
}
```

```txt
Benchmark                             Mode  Cnt     Score   Error  Units
ConsumeCPU.consume_0000  avgt    5     1.976 ± 0.007  ns/op
ConsumeCPU.consume_0001  avgt    5     1.933 ± 0.014  ns/op
ConsumeCPU.consume_0002  avgt    5     2.058 ± 0.007  ns/op
ConsumeCPU.consume_0004  avgt    5     3.182 ± 0.012  ns/op
ConsumeCPU.consume_0008  avgt    5     4.462 ± 0.020  ns/op
ConsumeCPU.consume_0016  avgt    5    10.607 ± 0.043  ns/op
ConsumeCPU.consume_0032  avgt    5    27.758 ± 0.138  ns/op
ConsumeCPU.consume_0064  avgt    5    77.873 ± 0.437  ns/op
ConsumeCPU.consume_0128  avgt    5   190.842 ± 1.602  ns/op
ConsumeCPU.consume_0256  avgt    5   412.204 ± 2.976  ns/op
ConsumeCPU.consume_0512  avgt    5   856.362 ± 3.612  ns/op
ConsumeCPU.consume_1024  avgt    5  1742.743 ± 9.288  ns/op
```

Blackhole 除了可以用来“死码消除”，同时 Blackhole 也可以“吞噬”cpu 时间片。

Blackhole.consumeCPU 的参数是时间片的 tokens，和时间片成线性关系。

## Profiler

```java
        public static void main(String[] args) throws RunnerException {
            Options opt = new OptionsBuilder()
                    .include(ProfilersTest.Classy.class.getSimpleName())
                    .addProfiler(GCProfiler.class)
                    .build();

            new Runner(opt).run();
        }
```

JMH 内置的性能剖析工具可以查看基准测试消耗在什么地方，具体的剖析方式内置的有如下几种：

- ClassloaderProfiler：类加载剖析
- CompilerProfiler：JIT 编译剖析
- GCProfiler：GC 剖析
- StackProfiler：栈剖析
- PausesProfiler：停顿剖析
- HotspotThreadProfiler：Hotspot 线程剖析
- HotspotRuntimeProfiler：Hotspot 运行时剖析
- HotspotMemoryProfiler：Hotspot 内存剖析
- HotspotCompilationProfiler：Hotspot 编译剖析
- HotspotClassloadingProfiler：Hotspot 类加载剖析

## 图形化分析

```java
public class BenchmarkTest {

    public static void main(String[] args) throws RunnerException {
        final Options options = new OptionsBuilder()
                .include(BenchmarkTest.class.getSimpleName())
                .result("BenchmarkTest.json")
                .resultFormat(ResultFormatType.JSON)
                .build();
        new Runner(options).run();
    }

}

```

使用 resultFormat 指定导出格式，result 指定导出为止，执行完成后，将测试数据导出为 JSON 文件后，上传到以下网站即可进行分析

[JMH Visualizer](https://jmh.morethan.io/)

[JMH Visual Chart](https://deepoove.com/jmh-visual-chart/)
