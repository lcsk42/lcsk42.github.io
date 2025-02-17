---
title: 'Design Patterns'
slug: 'design-patterns'
description: 23 种设计模式
date: 2024-02-17T15:01:09+08:00
image:
categories:
  - SE
tags:
  - Design Patterns
---

23 种设计模式。

整理自 [Java Design Patterns](https://java-design-patterns.com/zh/), 去除了代码实现部分，方便快速回顾。

<!--more-->

## 总览

设计模式分为三种类型，共 23 种：

[创建型模式(Creational Pattern)](#创建型模式)：

1. [工厂模式(Factory Method) 5](#工厂模式)
2. [抽象工厂模式(Abstract Factory) 5](#抽象工厂模式)
3. [单例模式(Singleton) 4](#单例模式)
4. [建造者模式(Builder) 2](#建造者模式)
5. [原型模式(Prototype) 3](#原型模式)

[结构型模式(Structural Pattern)](#结构型模式)：

1. [外观模式(Facade) 5](#外观模式)
2. [适配器模式(Adapter) 4](#适配器模式)
3. [组合模式(Composite) 4](#组合模式)
4. [代理模式(Proxy) 4](#代理模式)
5. [桥接模式(Bridge) 3](#桥接模式)
6. [装饰模式(Decorator) 3](#装饰模式)
7. [享元模式(Flyweight) 1](#享元模式)

[行为型模式(Behavioral Pattern)](#行为型模式)：

1. [迭代器模式(Iterator) 5](#迭代器模式)
2. [观察者模式(Observer) 5](#观察者模式)
3. [策略模式(Strategy) 4](#策略模式)
4. [命令模式(Command) 4](#命令模式)
5. [模版方法模式(Template Method) 3](#模版方法模式)
6. [状态模式(State) 3](#状态模式)
7. [责任链模式(Chain of Responsibility) 3](#责任链模式)
8. [中介者模式(Mediator) 2](#中介者模式)
9. [备忘录模式(Memento) 2](#备忘录模式)
10. [解释器模式(Interpreter) 1](#解释器模式)
11. [访问者模式(Visitor) 1](#访问者模式)

## 创建型模式

对类的实例化过程进行了抽象，能够将软件模块中对象的创建和对象的使用分离。为了使软件的结构更加清晰，外界对于这些对象只需要知道它们共同的接口，而不清楚其具体的实现细节，使整个系统的设计更加符合单一职责原则。

创建型模式在创建什么(What)，由谁创建(Who)，何时创建(When)
等方面都为软件设计者提供了尽可能大的灵活性。创建型模式隐藏了类的实例的创建细节，通过隐藏对象如何被创建和组合在一起达到使整个系统独立的目的。

### 工厂模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/factory-method/)

#### 工厂模式 - 目的

为创建一个对象定义一个接口，但是让子类决定实例化哪个类。工厂方法允许类将实例化延迟到子类。

#### 工厂模式 - 解释

真实世界例子

> 铁匠生产武器。精灵需要精灵武器，而兽人需要兽人武器。根据客户来召唤正确类型的铁匠。

通俗的说

> 它为类提供了一种把实例化的逻辑委托给子类的方式。

维基百科上说

> 在基于类的编程中，工厂方法模式是一种创建型设计模式用来解决创建对象的问题，而不需要指定将要创建对象的确切类。这是通过调用工厂方法创建对象来完成的，而不是通过调用构造器。该工厂方法在接口中指定并由子类实现，或者在基类实现并可以选择由子类重写。

#### 工厂模式 - 适用性

使用工厂方法模式当

- 一个类无法预料它所要必须创建的对象的类
- 一个类想要它的子类来指定它要创建的对象
- 类将责任委派给几个帮助子类中的一个，而你想定位了解是具体之中的哪一个

#### 工厂模式 - 使用

- [java.util.Calendar](http://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html#getInstance--)
- [java.util.ResourceBundle](http://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-)
- [java.text.NumberFormat](http://docs.oracle.com/javase/8/docs/api/java/text/NumberFormat.html#getInstance--)
- [java.nio.charset.Charset](http://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#forName-java.lang.String-)
- [java.net.URLStreamHandlerFactory](http://docs.oracle.com/javase/8/docs/api/java/net/URLStreamHandlerFactory.html#createURLStreamHandler-java.lang.String-)
- [java.util.EnumSet](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html#of-E-)
- [javax.xml.bind.JAXBContext](https://docs.oracle.com/javase/8/docs/api/javax/xml/bind/JAXBContext.html#createMarshaller--)

### 抽象工厂模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/abstract-factory/)

#### 抽象工厂模式 - 目的

提供一个用于创建相关对象家族的接口，而无需指定其具体类。

#### 抽象工厂模式 - 解释

真实世界例子

> 要创建一个王国，我们需要具有共同主题的对象。精灵王国需要精灵国王、精灵城堡和精灵军队，而兽人王国需要兽人国王、兽人城堡和兽人军队。王国中的对象之间存在依赖关系。

通俗的说

> 工厂的工厂； 一个将单个但相关/从属的工厂分组在一起而没有指定其具体类别的工厂。

维基百科上说

> 抽象工厂模式提供了一种封装一组具有共同主题的单个工厂而无需指定其具体类的方法

#### 抽象工厂模式 - 适用性

在以下情况下使用抽象工厂模式

- 该系统应独立于其产品的创建，组成和表示方式
- 系统应配置有多个产品系列之一
- 相关产品对象系列旨在一起使用，你需要强制执行此约束
- 你想提供产品的类库，并且只想暴露它们的接口，而不是它们的实现。
- 从概念上讲，依赖项的生存期比使用者的生存期短。
- 你需要一个运行时值来构建特定的依赖关系
- 你想决定在运行时从系列中调用哪种产品。
- 你需要提供一个或更多仅在运行时才知道的参数，然后才能解决依赖关系。
- 当你需要产品之间的一致性时
- 在向程序添加新产品或产品系列时，您不想更改现有代码。

示例场景

- 在运行时在 FileSystemAcmeService ，DatabaseAcmeService 或 NetworkAcmeService 中选择并调用一个
- 单元测试用例的编写变得更加容易
- 适用于不同操作系统的 UI 工具

#### 抽象工厂模式 - 后果

- Java 中的依赖注入会隐藏服务类的依赖关系，这些依赖关系可能导- 致运行时错误，而这些错误在编译时会被捕获。
- 虽然在创建预定义对象时模式很好，但是添加新对象可能会很困难。
- 由于引入了许多新的接口和类，因此代码变得比应有的复杂。

#### 抽象工厂模式 - 使用

- [javax.xml.parsers.DocumentBuilderFactory](http://docs.oracle.com/javase/8/docs/api/javax/xml/parsers/DocumentBuilderFactory.html)
- [javax.xml.transform.TransformerFactory](http://docs.oracle.com/javase/8/docs/api/javax/xml/transform/TransformerFactory.html#newInstance--)
- [javax.xml.xpath.XPathFactory](http://docs.oracle.com/javase/8/docs/api/javax/xml/xpath/XPathFactory.html#newInstance--)

### 单例模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/singleton/)

#### 单例模式 - 目的

确保一个类只有一个实例，并为其提供一个全局访问点。

#### 单例模式 - 解释

情境示例

> 巫师们之在一个象牙塔中学习他们的魔法，并且始终使用同一座附魔的象牙塔。
> 这里的象牙塔是一个单例对象。

通俗来说

> 对于一个特定的类，确保只会创建一个对象。

维基百科说

> 在软件工程中，单例模式是一种软件设计模式，它将类的实例化限制为一个对象。当系统中只需要一个对象来协调各种操作时，这种模式非常有用。

#### 单例模式 - 适用性

当满足以下情况时，使用单例模式：

- 确保一个类只有一个实例，并且客户端能够通过一个众所周知的访问点访问该实例。
- 唯一的实例能够被子类扩展, 同时客户端不需要修改代码就能使用扩展后的实例。

一些典型的单例模式用例包括：

- logging 类
- 管理与数据库的链接
- 文件管理器（File manager）

#### 单例模式 - 使用

- [java.lang.Runtime#getRuntime()](http://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html#getRuntime%28%29)
- [java.awt.Desktop#getDesktop()](http://docs.oracle.com/javase/8/docs/api/java/awt/Desktop.html#getDesktop--)
- [java.lang.System#getSecurityManager()](http://docs.oracle.com/javase/8/docs/api/java/lang/System.html#getSecurityManager--)

### 建造者模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/builder/)

#### 建造者模式 - 目的

将复杂对象的构造与其表示分开，以便同一构造过程可以创建不同的表示。

#### 建造者模式 - 解释

现实世界例子

> 想象一个角色扮演游戏的角色生成器。最简单的选择是让计算机为你创建角色。但是如果你想选择一些像专业，性别，发色等角色细节时，这个角色生成就变成了一个渐进的过程。当所有选择完成时，该过程也将完成。

用通俗的话说

> 允许你创建不同口味的对象同时避免构造器污染。当一个对象可能有几种口味，或者一个对象的创建涉及到很多步骤时会很有用。

维基百科说

> 建造者模式是一种对象创建的软件设计模式，旨在为伸缩构造器反模式寻找一个解决方案。

#### 建造者模式 - 适用性

使用建造者模式当

- 创建复杂对象的算法应独立于组成对象的零件及其组装方式
- 构造过程必须允许所构造的对象具有不同的表示形式

#### 建造者模式 - 使用

- [java.lang.StringBuilder](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html)
- [java.nio.ByteBuffer](http://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html#put-byte-) as well as similar buffers such as FloatBuffer, IntBuffer and so on.
- [java.lang.StringBuffer](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuffer.html#append-boolean-)
- All implementations of [java.lang.Appendable](http://docs.oracle.com/javase/8/docs/api/java/lang/Appendable.html)
- [Apache Camel builders](https://github.com/apache/camel/tree/0e195428ee04531be27a0b659005e3aa8d159d23/camel-core/src/main/java/org/apache/camel/builder)
- [Apache Commons Option.Builder](https://commons.apache.org/proper/commons-cli/apidocs/org/apache/commons/cli/Option.Builder.html)

### 原型模式

## 结构型模式

结构型模式(Structural Pattern)描述如何将类或者对 象结合在一起形成更大的结构，就像搭积木，可以通过
简单积木的组合形成复杂的、功能更为强大的结构。

结构型模式可以分为类结构型模式和对象结构型模式：

- 类结构型模式关心类的组合，由多个类可以组合成一个更大的

系统，在类结构型模式中一般只存在继承关系和实现关系。 - 对象结构型模式关心类与对象的组合，通过关联关系使得在一
个类中定义另一个类的实例对象，然后通过该对象调用其方法。 根据“合成复用原则”，在系统中尽量使用关联关系来替代继
承关系，因此大部分结构型模式都是对象结构型模式。

### 外观模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/facade/)

#### 外观模式 - 目的

为一个子系统中的一系列接口提供一个统一的接口。外观定义了一个更高级别的接口以便子系统更容易使用。

#### 外观模式 - 解释

真实世界的例子

> 一个金矿是怎么工作的？“嗯，矿工下去然后挖金子！”你说。这是你所相信的因为你在使用一个金矿对外提供的一个简单接口，在内部它要却要做很多事情。这个简单的接口对复杂的子系统来说就是一个外观。

用通俗的话说

> 外观模式为一个复杂的子系统提供一个简单的接口。

维基百科说

> 外观是为很大体量的代码（比如类库）提供简单接口的一种对象。

#### 外观模式 - 适用性

使用外观模式当

- 你想为一个复杂的子系统提供一个简单的接口。随着子系统的发展，它们通常会变得更加复杂。多数模式在应用时会导致更多和更少的类。这使子系统更可重用，更易于自定义，但是对于不需要自定义它的客户来说，使用它也变得更加困难。 外观可以提供子系统的简单默认视图，足以满足大多数客户端的需求。只有需要更多可定制性的客户才需要查看外观外的东西（原子系统提供的接口）。
- 客户端与抽象的实现类之间存在许多依赖关系。 引入外观以使子系统与客户端和其他子系统分离，从而提高子系统的独立性和可移植性。
- 您想对子系统进行分层。 使用外观来定义每个子系统级别的入口点。 如果子系统是相关的，则可以通过使子系统仅通过其外观相互通信来简化它们之间的依赖性。

### 适配器模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/adapter/)

#### 适配器模式 - 目的

将一个接口转换成另一个客户所期望的接口。适配器让那些本来因为接口不兼容的类可以合作无间。

#### 适配器模式 - 解释

现实世界例子

> 考虑有这么一种情况，在你的存储卡中有一些照片，你想将其传到你的电脑中。为了传送数据，你需要某种能够兼容你电脑接口的适配器以便你的储存卡能连上你的电脑。在这种情况下，读卡器就是一个适配器。
> 另一个例子就是注明的电源适配器；三脚插头不能插在两脚插座上，需要一个电源适配器来使其能够插在两脚插座上。
> 还有一个例子就是翻译官，他翻译一个人对另一个人说的话。

用直白的话来说

> 适配器模式让你可以把不兼容的对象包在适配器中，以让其兼容其他类。

维基百科中说

> 在软件工程中，适配器模式是一种可以让现有类的接口把其作为其他接口来使用的设计模式。它经常用来使现有的类和其他类能够工作并且不用修改其他类的源代码。

#### 适配器模式 - 应用

使用适配器模式当

- 你想使用一个已有类，但是它的接口不能和你需要的所匹配
- 你需要创建一个可重用类，该类与不相关或不可预见的类进行协作，即不一定具有兼容接口的类
- 你需要使用一些现有的子类，但是子类化他们每一个的子类来进行接口的适配是不现实的。一个对象适配器可以适配他们父类的接口。
- 大多数使用第三方类库的应用使用适配器作为一个在应用和第三方类库间的中间层来使应用和类库解耦。如果必须使用另一个库，则只需使用一个新库的适配器而无需改变应用程序的代码。

#### 适配器模式 - 后果

类和对象适配器有不同的权衡取舍。一个类适配器

- 适配被适配者到目标接口，需要保证只有一个具体的被适配者类。作为结果，当我们想适配一个类和它所有的子类时，类适配器将不会起作用。
- 可以让适配器重写一些被适配者的行为，因为适配器是被适配者的子类。
- 只引入了一个对象，并且不需要其他指针间接访问被适配者。

对象适配器

- 一个适配器可以和许多被适配者工作，也就是被适配者自己和所有它的子类。适配器同时可以为所有被适配者添加功能。
- 覆盖被适配者的行为变得更难。需要子类化被适配者然后让适配器引用这个子类不是被适配者。

#### 适配器模式 - 使用

- [java.util.Arrays#asList()](http://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#asList%28T...%29)
- [java.util.Collections#list()](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#list-java.util.Enumeration-)
- [java.util.Collections#enumeration()](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#enumeration-java.util.Collection-)
- [javax.xml.bind.annotation.adapters.XMLAdapter](http://docs.oracle.com/javase/8/docs/api/javax/xml/bind/annotation/adapters/XmlAdapter.html#marshal-BoundType-)

### 组合模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/composite/)

#### 组合模式 -目的

将对象组合成树结构以表示部分整体层次结构。 组合可以使客户统一对待单个对象和组合对象。

#### 组合模式 - 解释

真实世界例子

> 每个句子由单词组成，单词又由字符组成。这些对象中的每一个都是可打印的，它们可以在它们之前或之后打印一些内容，例如句子始终以句号结尾，单词始终在其前面有空格。

通俗的说

> 组合模式使客户能够以统一的方式对待各个对象。

维基百科说

> 在软件工程中，组合模式是一种分区设计模式。组合模式中，一组对象将像一个对象的单独实例一样被对待。组合的目的是将对象“组成”树状结构，以表示部分整体层次结构。实现组合模式可使客户统一对待单个对象和组合对象。

#### 组合模式 - 适用性

使用组合模式当

- 你想要表示对象的整体层次结构
- 你希望客户能够忽略组合对象和单个对象之间的差异。 客户将统一对待组合结构中的所有对象。

#### 组合模式 - 使用

- [java.awt.Container](http://docs.oracle.com/javase/8/docs/api/java/awt/Container.html) and [java.awt.Component](http://docs.oracle.com/javase/8/docs/api/java/awt/Component.html)
- [Apache Wicket](https://github.com/apache/wicket) component tree, see [Component](https://github.com/apache/wicket/blob/91e154702ab1ff3481ef6cbb04c6044814b7e130/wicket-core/src/main/java/org/apache/wicket/Component.java) and [MarkupContainer](https://github.com/apache/wicket/blob/b60ec64d0b50a611a9549809c9ab216f0ffa3ae3/wicket-core/src/main/java/org/apache/wicket/MarkupContainer.java)

### 代理模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/proxy/)

#### 代理模式 - 目的

为另一个对象提供代理或占位符以控制对其的访问。

#### 代理模式 - 解释

真实世界例子

> 想象有一个塔，当地的巫师去那里学习他们的法术。象牙塔只能够通过代理来进入以此来保证只有首先 3 个巫师才能进入。这里的代理就代表的塔的功能并添加访问控制。

通俗的说

> 使用代理模式，一个类代表另一个类的功能。

维基百科说

> 在最一般的形式上，代理是一个类，它充当与其他对象的接口。代理是客户端调用的包装器或代理对象，以访问后台的实际服务对象。代理本身可以简单地转发到真实对象，也可以提供其他逻辑。在代理中，可以提供额外的功能，例如在对实对象的操作占用大量资源时进行缓存，或者在对实对象的操作被调用之前检查前提条件。

#### 代理模式 - 适用性

代理适用于需要比简单指针更广泛或更复杂的对象引用的情况。这是代理模式适用的几种常见情况。

- 远程代理为不同地址空间中的对象提供了本地代表。
- 虚拟代理根据需要创建昂贵的对象。
- 保护代理控制对原始对象的访问。当对象有不同的接入权限时保护代理很有用。

#### 代理模式 - 典型用例

- 对象的访问控制
- 懒加载
- 实现日志记录
- 简化网络连接
- 对象的访问计数

#### 代理模式 - 使用

- [java.lang.reflect.Proxy](http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Proxy.html)
- [Apache Commons Proxy](https://commons.apache.org/proper/commons-proxy/)
- Mocking frameworks [Mockito](https://site.mockito.org/),
  [Powermock](https://powermock.github.io/), [EasyMock](https://easymock.org/)

### 桥接模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/bridge/)

#### 桥接模式 - 目的

将抽象与其实现分离，以便二者可以独立变化。

#### 桥接模式 - 解释

真实世界例子

> 考虑一下你拥有一种具有不同附魔的武器，并且应该允许将具有不同附魔的不同武器混合使用。 你会怎么做？ 为每个附魔创建每种武器的多个副本，还是只是创建单独的附魔并根据需要为武器设置它？ 桥接模式使您可以进行第二次操作。

通俗的说

> 桥接模式是一个更推荐组合而不是继承的模式。将实现细节从一个层次结构推送到具有单独层次结构的另一个对象。

维基百科说

> 桥接模式是软件工程中使用的一种设计模式，旨在“将抽象与其实现分离，从而使两者可以独立变化”

#### 桥接模式 - 适用性

使用桥接模式当

- 你想永久性的避免抽象和他的实现之间的绑定。有可能是这种情况，当实现需要被选择或者在运行时切换。
- 抽象和他们的实现应该能通过写子类来扩展。这种情况下，桥接模式让你可以组合不同的抽象和实现并独立的扩展他们。
- 对抽象的实现的改动应当不会对客户产生影响；也就是说，他们的代码不必重新编译。
- 你有种类繁多的类。这样的类层次结构表明需要将一个对象分为两部分。Rumbaugh 使用术语“嵌套归纳”来指代这种类层次结构。
- 你想在多个对象间分享一种实现（可能使用引用计数），这个事实应该对客户隐藏。一个简单的示例是 Coplien 的 String 类，其中多个对象可以共享同一字符串表示形式

### 装饰模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/decorator/)

#### 装饰模式 - 目的

动态的为对象附加额外的职责。装饰器为子类提供了灵活的替代方案，以扩展功能。

#### 装饰模式 - 解释

真实世界例子

> 附近的山丘上住着一个愤怒的巨魔。通常它是徒手的，但有时它有武器。为了武装巨魔不必创建新的巨魔，而是用合适的武器动态的装饰它。

通俗的说

> 装饰者模式让你可以在运行时通过把对象包装进一个装饰类对象中来动态的改变一个对象的行为。

维基百科说

> 在面向对象的编程中，装饰器模式是一种设计模式，它允许将行为静态或动态地添加到单个对象中，而不会影响同一类中其他对象的行为。装饰器模式通常对于遵守单一责任原则很有用，因为它允许将功能划分到具有唯一关注领域的类之间。

#### 装饰模式 - 适用性

使用装饰者

- 动态透明地向单个对象添加职责，即不影响其他对象
- 对于可以撤销的责任
- 当通过子类化进行扩展是不切实际的。有时可能会有大量的独立扩展，并且会产生大量的子类来支持每种组合。 否则类定义可能被隐藏或无法用于子类化。

#### 装饰模式 - 使用

- [java.io.InputStream](http://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html), [java.io.OutputStream](http://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html),
  [java.io.Reader](http://docs.oracle.com/javase/8/docs/api/java/io/Reader.html) and [java.io.Writer](http://docs.oracle.com/javase/8/docs/api/java/io/Writer.html)
- [java.util.Collections#synchronizedXXX()](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedCollection-java.util.Collection-)
- [java.util.Collections#unmodifiableXXX()](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#unmodifiableCollection-java.util.Collection-)
- [java.util.Collections#checkedXXX()](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#checkedCollection-java.util.Collection-java.lang.Class-)

### 享元模式

## 行为型模式

行为型模式(Behavioral Pattern)是对在不同的对象之间划分责任和算法的抽象化。

行为型模式不仅仅关注类和对象的结构，而且重点关注它们之间的相互作用。

通过行为型模式，可以更加清晰地划分类与对象的职责，并研究系统在运行时实例对象
之间的交互。在系统运行时，对象并不是孤立的，它们可以通过相互通信与协作完成某些复杂功能，一个对象在运行时也将影响到其他对象的运行。

行为型模式分为类行为型模式和对象行为型模式两种：

- 类行为型模式：类的行为型模式使用继承关系在几个类之间分配行为，类行为型模式主要通过多态等方式来分配父类与子类的职责。
-

对象行为型模式：对象的行为型模式则使用对象的聚合关联关系来分配行为，对象行为型模式主要是通过对象关联等方式来分配两个或多个类的职责。根据“合成复用原则”，系统中要尽量使用关联关系来取代继承关系，因此大部分行为型设计模式都属于对象行为型设计模式。

### 迭代器模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/iterator/)

#### 迭代器模式 - 目的

提供一种在不暴露其基础表示的情况下顺序访问聚合对象的元素的方法。

#### 迭代器模式 - 解释

真实世界例子

> 百宝箱包含一组魔法物品。有多种物品，例如戒指，药水和武器。可以使用藏宝箱提供的迭代器按类型浏览商品。

通俗地说

> 容器可以提供与表示形式无关的迭代器接口，以提供对元素的访问。

维基百科说

> 在面向对象的编程中，迭代器模式是一种设计模式，其中迭代器用于遍历容器并访问容器的元素。

#### 迭代器模式 - 适用性

以下情况使用迭代器模式

- 在不暴露其内部表示的情况下访问聚合对象的内容
- 为了支持聚合对象的多种遍历方式
- 提供一个遍历不同聚合结构的统一接口

#### 迭代器模式 - 使用

- [java.util.Iterator](http://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html)
- [java.util.Enumeration](http://docs.oracle.com/javase/8/docs/api/java/util/Enumeration.html)

### 观察者模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/observer/)

#### 观察者模式 - 目的

定义一种一对多的对象依赖关系这样当一个对象改变状态时，所有依赖它的对象都将自动通知或更新。

#### 观察者模式 - 解释

真实世界例子

> 在遥远的土地上生活着霍比特人和兽人的种族。他们都是户外生活的人所以他们密切关注天气的变化。可以说他们不断地关注着天气。

通俗的说

> 注册成为一个观察者以接收对象状态的改变。

维基百科说

> 观察者模式是这样的一种软件设计模式：它有一个被称为主题的对象，维护着一个所有依赖于它的依赖者清单，也就是观察者清单，当主题的状态发生改变时，主题通常会调用观察者的方法来自动通知观察者们。

#### 观察者模式 - 应用

在下面任何一种情况下都可以使用观察者模式

- 当抽象具有两个方面时，一个方面依赖于另一个方面。将这些方面封装在单独的对象中，可以使你分别进行更改和重用
- 当一个对象的改变的同时需要改变其他对象，同时你又不知道有多少对象需要改变时
- 当一个对象可以通知其他对象而无需假设这些对象是谁时。换句话说，你不想让这些对象紧耦合。

典型用例

- 一个对象的改变导致其他对象的改变

#### 观察者模式 - 使用

- [java.util.Observer](http://docs.oracle.com/javase/8/docs/api/java/util/Observer.html)
- [java.util.EventListener](http://docs.oracle.com/javase/8/docs/api/java/util/EventListener.html)
- [javax.servlet.http.HttpSessionBindingListener](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSessionBindingListener.html)
- [RxJava](https://github.com/ReactiveX/RxJava)

### 策略模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/strategy/)

#### 策略模式 - 目的

定义一个家族算法，并封装好其中每一个，使它们可以互相替换。策略模式使算法的变化独立于使用它的客户。

#### 策略模式 - 解释

现实世界例子

> 屠龙是一项危险的职业。有经验将会使它变得简单。经验丰富的屠龙者对不同类型的龙有不同的战斗策略。

直白点说

> 策略模式允许在运行时选择最匹配的算法。

维基百科上说

> 在程序编程领域，策略模式（又叫政策模式）是一种启用在运行时选择算法的行为型软件设计模式。

#### 策略模式 - 应用

使用策略模式当

- 许多相关的类只是行为不同。策略模式提供了一种为一种类配置多种行为的能力。
- 你需要一种算法的不同变体。比如，你可能定义反应不用时间空间权衡的算法。当这些算法的变体使用类的层次结构来实现时就可以使用策略模式。
- 一个算法使用的数据客户不应该对其知晓。使用策略模式来避免暴露复杂的，特定于算法的数据结构。
- 一个类定义了许多行为，这些行为在其操作中展现为多个条件语句。移动相关的条件分支到它们分别的策略类中来代替这些条件语句。

### 命令模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/command/)

#### 命令模式 - 目的

将请求封装为对象，从而使你可以将具有不同请求的客户端参数化，队列或记录请求，并且支持可撤销操作。

#### 命令模式 - 解释

真实世界例子

> 有一个巫师在地精上施放咒语。咒语在地精上一一执行。第一个咒语使地精缩小，第二个使他不可见。然后巫师将咒语一个个的反转。这里的每一个咒语都是一个可撤销的命令对象。

用通俗的话说

> 用命令对象的方式存储请求以在将来时可以执行它或撤销它。

维基百科说

> 在面向对象编程中，命令模式是一种行为型设计模式，它把在稍后执行的一个动作或触发的一个事件所需要的所有信息封装到一个对象中。

#### 命令模式 - 适用性

使用命令模式当你想

- 通过操作将对象参数化。您可以使用回调函数（即，已在某处注册以便稍后调用的函数）以过程语言表示这种参数化。命令是回调的一种面向对象替代方案。
- 在不同的时间指定，排队和执行请求。一个命令对象的生存期可以独立于原始请求。如果请求的接收方可以以地址空间无关的方式来表示，那么你可以将请求的命令对象传输到其他进程并在那里执行请求。
- 支持撤销。命令的执行操作可以在命令本身中存储状态以反转其效果。命令接口必须有添加的反执行操作，该操作可以逆转上一次执行调用的效果。执行的命令存储在历史列表中。无限撤消和重做通过分别向后和向前遍历此列表来实现，分别调用 unexecute 和 execute。
- 支持日志记录更改，以便在系统崩溃时可以重新应用它们。通过使用加载和存储操作扩展命令接口，你可以保留更改的永久日志。从崩溃中恢复涉及从磁盘重新加载记录的命令，并通过执行操作重新执行它们。
- 通过原始的操作来构建一个以高级操作围绕的系统。这种结构在支持事务的信息系统中很常见。事务封装了一组数据更改。命令模式提供了- 一种对事务进行建模的方法。命令具有公共接口，让你以相同的方式调用所有事务。该模式还可以通过新的事务来轻松扩展系统。

#### 命令模式 - 典型用例

- 保留请求历史
- 实现回调功能
- 实现撤销功能

#### 命令模式 - 使用

- [java.lang.Runnable](http://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html)
- [org.junit.runners.model.Statement](https://github.com/junit-team/junit4/blob/master/src/main/java/org/junit/runners/model/Statement.java)
- [Netflix Hystrix](https://github.com/Netflix/Hystrix/wiki)
- [javax.swing.Action](http://docs.oracle.com/javase/8/docs/api/javax/swing/Action.html)

### 模版方法模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/template-method/)

#### 模版方法模式 - 目的

在一个操作中定义算法的骨架，将某些步骤推迟到子类。模板方法允许子类重新定义算法的某些步骤，而无需更改算法的结构。

#### 模版方法模式 - 解释

真实世界例子

> 偷东西的一般步骤是相同的。 首先，选择目标，然后以某种方式使其迷惑，最后，你偷走了该物品。然而这些步骤有很多实现方式。

通俗的说

> 模板方法模式在父类中列出一般的步骤然后让具体的子类定义实现细节。

维基百科说

> 在面向对象的编程中，模板方法是 Gamma 等人确定的行为设计模式之一。在《设计模式》一书中。模板方法是父类中一个方法，通常是一个抽象父类，根据许多高级步骤定义了操作的骨架。这些步骤本身由与模板方法在同一类中的其他帮助程序方法实现。

#### 模版方法模式 - 适用性

使用模板方法模式可以

- 一次性实现一个算法中不变的部分并将其留给子类来实现可能变化的行为。
- 子类之间的共同行为应分解并集中在一个共同类中，以避免代码重复。如 Opdyke 和 Johnson 所描述的，这是“重构概括”的一个很好的例子。你首先要确定现有代码中的差异，然后将差异拆分为新的操作。最后，将不同的代码替换为调用这些新操作之一的模板方法。
- 控制子类扩展。您可以定义一个模板方法，该方法在特定点调用“ 钩子”操作，从而仅允许在这些点进行扩展

#### 模版方法模式 - 使用

- [javax.servlet.GenericServlet.init](https://jakarta.ee/specifications/servlet/4.0/apidocs/javax/servlet/GenericServlet.html#init--):

  Method `GenericServlet.init(ServletConfig config)` calls the parameterless method `GenericServlet.init()` which is intended to be overridden in subclasses.

  Method `GenericServlet.init(ServletConfig config)` is the template method in this example.

### 状态模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/state/)

#### 状态模式 - 目的

允许对象在内部状态改变时改变它的行为。对象看起来好像修改了它的类。

#### 状态模式 - 解释

真实世界例子

> 当在长毛象的自然栖息地观察长毛象时，似乎它会根据情况来改变自己的行为。它开始可能很平静但是随着时间推移当它检测到威胁时它会对周围的环境感到愤怒和危险。

通俗的说

> 状态模式允许对象改变它的行为。

维基百科说

> 状态模式是一种允许对象在内部状态改变时改变它的行为的行为型设计模式。这种模式接近于有限状态机的概念。状态模式可以被理解为策略模式，它能够通过调用在模式接口中定义的方法来切换策略。

#### 状态模式 - 适用性

在以下两种情况下，请使用 State 模式

- 对象的行为取决于它的状态，并且它必须在运行时根据状态更改其行为。
- 根据对象状态的不同，操作有大量的条件语句。此状态通常由一个或多个枚举常量表示。通常，几个操作将包含此相同的条件结构。状态模式把条件语句的分支分别放入单独的类中。这样一来，你就可以将对象的状态视为独立的对象，该对象可以独立于其他对象而变化。

#### 状态模式 - 使用

- [javax.faces.lifecycle.Lifecycle#execute()](http://docs.oracle.com/javaee/7/api/javax/faces/lifecycle/Lifecycle.html#execute-javax.faces.context.FacesContext-) controlled by [FacesServlet](http://docs.oracle.com/javaee/7/api/javax/faces/webapp/FacesServlet.html), the behavior is dependent on current phase of lifecycle.
- [JDiameter - Diameter State Machine](https://github.com/npathai/jdiameter/blob/master/core/jdiameter/api/src/main/java/org/jdiameter/api/app/State.java)

### 责任链模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/chain/)

#### 责任链模式 - 目的

通过给多个对象一个处理请求的机会，避免请求的发送者和它的接收者耦合。串联接收对象并在链条中传递请求直到一个对象处理它。

#### 责任链模式 - 解释

真实世界例子

> 兽王大声命令他的军队。最近响应的是指挥官，然后是军官，然后是士兵。指挥官，军官，士兵这里就形成了一个责任链。

通俗的说

> 它帮助构建一串对象。请求从一个对象中进入并结束然后进入到一个个对象中直到找到合适的处理器。

维基百科说

> 在面向对象设计中，责任链模式是一种由源命令对象和一系列处理对象组成的设计模式。每个处理对象包含了其定义的可处理的命令对象类型的逻辑。剩下的会传递给链条中的下一个处理对象。

#### 责任链模式 - 适用性

使用责任链模式当

- 多于一个对象可能要处理请求，并且处理器并不知道一个优先级。处理器应自动确定。
- 你想对多个对象之一发出请求而无需明确指定接收者
- 处理请求的对象集合应该被动态指定时

#### 责任链模式 - 使用

- [java.util.logging.Logger#log()](http://docs.oracle.com/javase/8/docs/api/java/util/logging/Logger.html#log%28java.util.logging.Level,%20java.lang.String%29)
- [Apache Commons Chain](https://commons.apache.org/proper/commons-chain/index.html)
- [javax.servlet.Filter#doFilter()](http://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html#doFilter-javax.servlet.ServletRequest-javax.servlet.ServletResponse-javax.servlet.FilterChain-)

### 中介者模式

### 备忘录模式

### 解释器模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/interpreter/)

#### 解释器模式 - 目的

给定一种语言，请定义其语法的表示形式，以及使用该表示形式来解释该语言中的句子的解释器。

#### 解释器模式 - 适用性

有一种要解释的语言时，请使用解释器模式，并且可以将语言中的语句表示为抽象语法树。解释器模式在以下情况下效果最佳

- 语法很简单。 对于复杂的语法，语法的类层次结构变得庞大且难以管理。 在这种情况下，解析器生成器之类的工具是更好的选择。 他们可以在不构建抽象语法树的情况下解释表达式，这可以节省空间并可能节省时间
- 效率不是关键问题。 通常，最有效的解释器不是通过直接解释解析树来实现的，而是先将其转换为另一种形式。 例如，正则表达式通常会转换为状态机。 但是即使这样，翻译器也可以通过解释器模式实现，因此该模式仍然适用。

#### 解释器模式 - 使用

- [java.util.Pattern](http://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html)
- [java.text.Normalizer](http://docs.oracle.com/javase/8/docs/api/java/text/Normalizer.html)
- All subclasses of [java.text.Format](http://docs.oracle.com/javase/8/docs/api/java/text/Format.html)
- [javax.el.ELResolver](http://docs.oracle.com/javaee/7/api/javax/el/ELResolver.html)

### 访问者模式

详细地址: [Java Design Patterns](https://java-design-patterns.com/zh/patterns/visitor/)

#### 访问者模式 - 目的

表示要在对象结构的元素上执行的操作。访问者可让你定义新操作，而无需更改其所操作元素的类。

#### 访问者模式 - 解释

真实世界例子

> 考虑有一个带有军队单位的树形结构。指挥官下有两名中士，每名中士下有三名士兵。基于这个层级结构实现访问者模式，我们可以轻松创建与指挥官，中士，士兵或所有人员互动的新对象

通俗的说

> 访问者模式定义可以在数据结构的节点上执行的操作。

维基百科说

> 在面向对象的程序设计和软件工程中，访问者设计模式是一种将算法与操作对象的结构分离的方法。这种分离的实际结果是能够在不修改结构的情况下向现有对象结构添加新操作。

#### 访问者模式 - 适用性

使用访问者模式当

- 对象结构包含许多具有不同接口的对象类，并且你希望根据这些对象的具体类对这些对象执行操作。
- 需要对对象结构中的对象执行许多不同且不相关的操作，并且你想避免使用这些操作“污染”它们的类。 访问者可以通过在一个类中定义相关操作来将它们保持在一起。当许多应用程序共享对象结构时，请使用访问者模式将操作仅放在需要它们的那些应用程序中
- 定义对象结构的类很少变化，但是你经常想在结构上定义新的操作。更改对象结构类需要重新定义所有访问者的接口，这可能会导致成本高昂。如果对象结构类经常更改，则最好在这些类中定义操作。

#### 访问者模式 - 使用

- [Apache Wicket](https://github.com/apache/wicket) component tree, see [MarkupContainer](https://github.com/apache/wicket/blob/b60ec64d0b50a611a9549809c9ab216f0ffa3ae3/wicket-core/src/main/java/org/apache/wicket/MarkupContainer.java)
- [javax.lang.model.element.AnnotationValue](http://docs.oracle.com/javase/8/docs/api/javax/lang/model/element/AnnotationValue.html) and [AnnotationValueVisitor](http://docs.oracle.com/javase/8/docs/api/javax/lang/model/element/AnnotationValueVisitor.html)
- [javax.lang.model.element.Element](http://docs.oracle.com/javase/8/docs/api/javax/lang/model/element/Element.html) and [Element Visitor](http://docs.oracle.com/javase/8/docs/api/javax/lang/model/element/ElementVisitor.html)
- [java.nio.file.FileVisitor](http://docs.oracle.com/javase/8/docs/api/java/nio/file/FileVisitor.html)
