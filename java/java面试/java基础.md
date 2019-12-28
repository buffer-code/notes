---
typora-copy-images-to: ..\images
typora-root-url: ..
---

**Java 语言特性**，包括泛型、Lambda 等语言特性；

**基础类库**，包括集合、IO/NIO、网络、并发、安全等基础类库。

**谈谈 JVM 的一些基础概念和机制**，比如 Java 的类加载机制，

**常用版本 JDK（如 JDK 8）内嵌的 Class-Loader**，例如 Bootstrap、 Application 和 Extension Class-loader；

**类加载大致过程**：加载、验证、链接、初始化（这里参考了周志明的《深入理解 Java 虚拟机》，非常棒的 JVM 上手书籍）；自定义 Class-Loader 等。

**还有垃圾收集的基本原理**，最常见的垃圾收集器，如 SerialGC、Parallel GC、 CMS、 G1 等，

**JDK 包含哪些工具或者 Java 领域内其他工具等**，如编译器、运行时环境、安全工具、诊断和监控工具等

![java知识结构](/images/java知识结构.png)

**1、谈谈你对java平台的理解？“java是解释执行”。这句话对吗？**

------

Java是一种面向对象语言，显著特性主要是**“书写一次，到出运行”**，**“垃圾收集”**。

**JRE：**是java运行环境，包含jvm和java类库，以及一些模块。

**JDK：**是JRE的一个超集，提供了更多的工具，如编译器，各种诊断工具等。

“Java是解释执行”，这句话，说法不太准确。

通常我们开发的java源代码，**首先通过Javac编译成为字节码（bytecode），然后在运行时，通过Java虚拟机（JVM）内嵌的解释器将字节码转换成为最终的机器码。**但是常见的JVM，比如我们大对数情况下使用的Oracle JDK提供的Hostpot JVM,都提供了JIT（Just-In-Time）编译器，也就是通常所说动态编译器，JIT能够在运行时将**热点代码编译成机器码**，这种情况下部分热点代码就属于编译执行，而不是解释执行了。

------

在**运行时**，JVM 会通过类加载器（Class-Loader）加载字节码，解释或者编译执行。就像我前面提到的，主流 Java 版本中，如 **JDK 8 实际是解释和编译混合的一种模式，即所谓的混合模式（-Xmixed）。通常运行在 server 模式的 JVM，会进行上万次调用以收集足够的信息进行高效的编译，client 模式这个门限是 1500 次。**Oracle Hotspot JVM 内置了两个不同的 JIT compiler，**C1 对应前面说的 client 模式**，适用于对于启动速度敏感的应用，比如普通 Java 桌面应用；**C2 对应 server 模式**，它的优化是为长时间运行的服务器端应用设计的。默认是采用所谓的分层编译（TieredCompilation）。

Java 虚拟机启动时，可以指定不同的参数对运行模式进行选择。 比如，**指定“-Xint”**，就是告诉 JVM **只进行解释执行，不对代码进行编译**，这种模式抛弃了 JIT 可能带来的性能优势。毕竟解释器（interpreter）是逐条读入，逐条解释运行的。与其相对应的，还有一个**“-Xcomp”参数**，这是告诉 **JVM 关闭解释器，不要进行解释执行**，或者叫作**最大优化级别**。那你可能会问这种模式是不是最高效啊？简单说，还真未必。“-Xcomp”会导致 JVM 启动变慢非常多，同时有些 JIT 编译器优化方式，比如分支预测，如果不进行 profiling（解析），往往并不能进行有效优化。

除了我们日常最常见的 Java 使用模式，其实还有一种新的编译方式，即所谓的 AOT（Ahead-of-Time Compilation），**直接将字节码编译成机器代码**，这样就避免了 **JIT 预热**等各方面的开销，比如 Oracle JDK 9 就引入了实验性的 AOT 特性，并且增加了新的 jaotc 工具。利用下面的命令

```shell

jaotc --output libHelloWorld.so HelloWorld.class
jaotc --output libjava.base.so --module java.base
```

然后，在启动时直接指定就可以了。

```shell
java -XX:AOTLibrary=./libHelloWorld.so,./libjava.base.so HelloWorld
```

而且，Oracle JDK 支持分层编译和 AOT 协作使用，这两者并不是二选一的关系。如果你有兴趣，可以参考相关文档：http://openjdk.java.net/jeps/295。