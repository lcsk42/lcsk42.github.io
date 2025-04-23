---
title: 'Spring Boot'
slug: 'spring-boot'
description: 介绍 Spring Boot 相关知识
date: 2025-02-14T13:50:27+08:00
categories:
  - Dev
tags:
  - Java
  - Spring Boot
---

分模块介绍 Spring Boot 相关知识。

<!--more-->

## Spring Boot 初始化类的先后顺序

1. `SpringApplication`
2. `InitializingBean`
3. `@PostConstruct`
4. `ApplicationRunner`
5. `CommandLineRunner`

### SpringApplication

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BeanDemoApplication {
    public static void main(String[] args) {
        System.out.println("Before:: SpringApplication#run()");
        SpringApplication.run(BeanDemoApplication.class, args);
        System.out.println("After:: SpringApplication#run()");
    }
}
```

### InitializingBean

`InitializingBean` 接口为 bean 提供了属性初始化后的处理方法，只拥有一个 `afterPropertiesSet` 方法，凡是继承该接口的类，在 bean 的属性初始化后都会执行该方法。

```java
package org.springframework.beans.factory;

public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}
```

```java
import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;

@Component
public class CustomInitializingBean implements InitializingBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean#afterPropertiesSet()");
    }
}
```

### @PostConstruct

```java
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

@Component
public class CustomPostConstruct {
    @PostConstruct
    public void method() {
        System.out.println("@PostConstruct -> method()");
    }
}
```

### ApplicationRunner

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Component
public class CustomApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("ApplicationRunner#run(ApplicationArguments args)");
    }
}
```

### CommandLineRunner

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class CustomCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner#run(String... args)");
    }
}

```

### 输出

```log
Before:: SpringApplication#run()

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.7.6)

2025-02-14 16:02:17.808  INFO 76746 --- [           main] c.example.beandemo.BeanDemoApplication   : Starting BeanDemoApplication using Java 1.8.0_442 on MacBook-Pro.local with PID 76746
2025-02-14 16:02:17.810  INFO 76746 --- [           main] c.example.beandemo.BeanDemoApplication   : No active profile set, falling back to 1 default profile: "default"
InitializingBean#afterPropertiesSet()
@PostConstruct -> method()
2025-02-14 16:02:18.008  INFO 76746 --- [           main] c.example.beandemo.BeanDemoApplication   : Started BeanDemoApplication in 0.334 seconds (JVM running for 0.538)
ApplicationRunner#run(ApplicationArguments args)
CommandLineRunner#run(String... args)
After:: SpringApplication#run()
```

## Spring Boot 自动装配

### 什么是 Spring Boot 的自动装配

**SPI**（Service Provider Interface） 是一种 Java 标准机制，用于实现服务的动态发现和加载。Spring Boot 利用 SPI 机制来扩展框架功能，允许开发者通过简单的配置实现自定义逻辑。

SpringBoot 在启动过程中会扫描外部引入的 jar 包中的 `META-INF/spring.factories` （自 Spring Boot 3.0 开始，位置修改为 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`）文件，根据文件配置的类型信息加载到 Spring 容器中，并执行类中定义的各种操作。这样外部的 jar 来说，只要按照 Spring Boot 定义的标准，就能自动将自己的功能加载进 Spring Boot。

### 如何实现 Spring Boot 的自动装配

1. 核心注解 `@SpringBootApplication` 包含了三个注解:
   1. `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
   2. `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类
   3. `@ComponentScan`：扫描被 `@Component` (`@Service`,`@Controller` 等)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。
2. `@EnableAutoConfiguration` 实现自动装配的核心逻辑
   1. `AutoConfigurationImportSelector` 类实现了 `ImportSelector` 接口，也就实现了这个接口中的 `selectImports` 方法, 该方法获取到所有符合条件的类的全限定类名，这些类就是需要加载到 IoC 容器中的类。
   2. 方法中的 `getAutoConfigurationEntry()` 主要负责加载自动配置类
      1. 判断自动装配开关是否打开（`spring.boot.enableautoconfiguration` 配置）
      2. 用于获取 `@EnableAutoConfiguration` 注解中的 `exclude` 和 `excludeName`
      3. 获取需要自动装配的所有配置类，读取所有 Spring Boot Starter 下的 `META-INF/spring.factories`
      4. 筛选所有满足 `@ConditionalOnXXX` 注解的类

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
    ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
  @Override

  // ...

  public String[] selectImports(AnnotationMetadata annotationMetadata) {
    // 1. 判断判断自动装配开关是否打开
    if (!isEnabled(annotationMetadata)) {
      return NO_IMPORTS;
    }
    // 2. 获取所有需要装配的 bean
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
  }

  // ...

  protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    // 1. 判断自动装配开关是否打开
    if (!isEnabled(annotationMetadata)) {
      return EMPTY_ENTRY;
    }
    // 2. 用于获取 EnableAutoConfiguration 注解中的 exclude 和 excludeName
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 3. 获取需要自动装配的所有配置类，读取 META-INF/spring.factories
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    // 4. 筛选 是否满足 @ConditionalOnXXX
    configurations = removeDuplicates(configurations);
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = getConfigurationClassFilter().filter(configurations);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
  }

  // ...

}
```

Spring Boot 提供的条件注解：

- `@ConditionalOnBean`：当容器里有指定 Bean 的条件下@ConditionalOnMissingBean：当容器里没有指定 Bean 的情况下
- `@ConditionalOnSingleCandidate`：当指定 Bean 在容器中只有一个，或者虽然有多个但是指定首选 Bean
- `@ConditionalOnClass`：当类路径下有指定类的条件下
- `@ConditionalOnMissingClass`：当类路径下没有指定类的条件下
- `@ConditionalOnProperty`：指定的属性是否有指定的值
- `@ConditionalOnResource`：类路径是否有指定的值
- `@ConditionalOnExpression`：基于 SpEL 表达式作为判断条件
- `@ConditionalOnJava`：基于 Java 版本作为判断条件
- `@ConditionalOnJndi`：在 JNDI 存在的条件下差在指定的位置
- `@ConditionalOnNotWebApplication`：当前项目不是 Web 项目的条件下
- `@ConditionalOnWebApplication`：当前项目是 Web 项 目的条件下

### 如何实现一个 Spring Boot Starter

#### 如何命名

Spring 官方提供 Starter 通常命名为 `spring-boot-starter-{name}` 如：

- spring-boot-starter-web
- spring-boot-starter-activemq
- 等等

Spring 官方建议非官方提供的 Starter 命名应遵守 `{name}-spring-boot-starter` 的格式 如：

- mybatis-spring-boot-starter

#### 创建项目

```txt

└── src
|   ├── main
|   │   ├── java
|   │   │   └── com
|   │   │       └── lcsk42
|   │   │           └── starter
|   │   │               └── demospringbootstarter
|   |   |                   ├── AutoConfigurationTest.java
|   |   |                   ├── ServiceBean.java
|   │   └── resources
|   |       └── META-INF
|   |           └── spring.factories
├── pom.xml
```

#### POM 依赖配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.11.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>com.lcsk42.starter</groupId>
    <artifactId>demo-spring-boot-starter</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>

</project>
```

#### 相关类

```java
public class ServiceBean {
    public String sayHello(String name) {
        return String.format("Hello World, %s", name);
    }
}
```

```java
@Configuration
public class AutoConfigurationTest {

    @Bean
    public ServiceBean getServiceBean() {
        return new ServiceBean();
    }
}
```

#### 配置文件

```txt
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.lcsk42.starter.demospringbootstarter.AutoConfigurationTest
```

## 事务

### 事务分类

Spring 支持两种事务管理方式，分别是**编程式事务管理**和**声明式事务管理**。

#### 编程式事务

通过 `TransactionTemplate` 或者 `TransactionManager(PlatformTransactionManager @2.7)` 手动管理事务

`TransactionTemplate` 事务管理：

```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.TransactionCallbackWithoutResult;
import org.springframework.transaction.support.TransactionTemplate;

@Service
@RequiredArgsConstructor
public class TransactionDemo {
    private final TransactionTemplate transactionTemplate;

    private void testTransactionTemplate() {
        transactionTemplate.execute(
                new TransactionCallbackWithoutResult() {
                    @Override
                    protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
                        try {
                            // ....  业务代码
                        } catch (Exception e) {
                            // 回滚
                            transactionStatus.setRollbackOnly();
                        }
                    }
                }
        );
    }
}
```

`TransactionManager(PlatformTransactionManager @2.7)` 事务管理：

```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

@Service
@RequiredArgsConstructor
public class TransactionDemo {
    private final PlatformTransactionManager transactionManager;

    private void testTransactionManager() {
        final TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            // ....  业务代码
            transactionManager.commit(status);
        } catch (Exception e) {
            transactionManager.rollback(status);
        }
    }
}
```

#### 声明式事务

`@Transactional` 的全注解方式，实际是通过 AOP 实现。

```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class TransactionDemo {
    @Transactional(
            propagation = Propagation.REQUIRED,
            isolation = Isolation.DEFAULT,
            rollbackFor = RuntimeException.class
    )
    public void testTransaction(){
        // 业务代码
    }
}
```

### 事务的传播行为

规定了当一个事物方法被另一个事物方法调用的时候，如何进行事务之间的传播。

```java
@Transactional(propagation = xxx)
public void methodA(){

}

@Transactional(propagation = xxx)
public void methodB(){

}
```

对应上方伪代码中的 xx，Spring 总共提供了七种事务传播行为：

| 事务传播行为类型        | 说明                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------- |
| **REQUIRED** （最常用） | 当前没有事务，就新建一个事务；如果已存在一个事务，加入到这个事务中                 |
| _SUPPORTS_              | 支持当前事务，如果当前没有事务，就以非事务方式执行                                 |
| **MANDATORY**           | 使用当前事务，如果当前没有事务，抛出异常                                           |
| **REQUIRES_NEW**        | 新建事务，如果当前存在事务，把当前事务挂起                                         |
| _NOT_SUPPORTED_         | 以非事务方式执行，如果当前存在事务，把当前事务挂起                                 |
| _NEVER_                 | 以非事务方式执行，如果当前存在事务，抛出异常                                       |
| **NESTED**              | 如果当前存在事务，嵌套到当前事务中运行；如果当前不存在事务，执行类似 REQUIRED 操作 |

针对以上的代码，给出以下几种情况(A 都是 REQUIRED)：

| 情况            | 异常 | 是否会滚            | 说明                                  |
| --------------- | ---- | ------------------- | ------------------------------------- |
| B(REQUIRED)     | A B  | **A(是)** **B(是)** | 本质是一个事务                        |
| B(REQUIRES_NEW) | A    | **A(是)** B(否)     | B 开启了独立事务                      |
| B(REQUIRES_NEW) | B    | **A(是)** **B(是)** | B 的异常也会被 A 的事务管理机制检测到 |
| B(NESTED)       | A    | **A(是)** **B(是)** |                                       |
| B(NESTED)       | B    | A(否) **B(是)**     |                                       |

### 事务只读

对于只有读取数据查询的事务，可以指定事务类型为 readonly，即只读事务。

- 如果只执行一条数据查询，则没有必要使用事务只读
- 如果要执行多条数据查询，需要保证整体的读**一致性**,则可以使用只读事务

### @Transactional 事务注解原理

`@Transactional` 基于 APO 的动态代理实现。

- 如果目标类实现了接口，Spring 使用 JDK 动态代理
- 如果目标类没有实现接口，Spring 使用 CGLIB 代理

`TransactionInterceptor` 拦截被 `@Transactional` 注解标记的方法

#### 开启事务

- 在方法执行前，根据 `@Transactional` 的配置，调用 `PlatformTransactionManager` 的 `getTransaction` 方法开启事务。

#### 执行业务逻辑

- 调用目标方法执行业务逻辑。

#### 提交或回滚事务

- 如果方法执行成功，调用 `commit` 提交事务
- 如果方法抛出异常，调用 `rollback` 回滚事务

### Spring AOP 自调用问题

如果一个方法增加了 `@Transactional` 注解，在别的类调用是可以生效的。

但是如果在自己的类中使用 `this` 调用，则不会生效。

这是由 AOP 的工作原理决定的，Spring AOP 使用了动态代理来实现事务的管理, 运行时为带有注解的方法生成代理对象，并在方法调用的前后实现了事务逻辑。

如果使用 IoC 注入方式调用会走代理对象，如果使用 `this` 调用，则不会走代理对象。

解决方案：

1. 拆分方法到不同类
2. 通过 `AopContext.currentProxy()` 获取当前类的代理对象，然后通过代理对象调用方法
3. 自注入，在自己的类中惰性注入自己
