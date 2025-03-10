---
layout: post
title: Java 日志工具库
categories: [Java Base]
tags: log,log4j,logback,log4j2
---

# Java 日志工具库

日志是程序员最好的朋友，它既可以跟踪代码，验证代码逻辑，又可以在出现问题的时候及时的提供线索，java 领域中，有很多优秀的日志工具库可供选择，那么实现了解一下他们是有必要的。

## 日志框架

### java.util.logging(JUL)

从jdk1.4开始，通过java.util.logging 提供日志功能，它可以满足最基本的日志功能。但是相对比较简单，基本不会使用在项目中。

### Log4j

apache的一个开源项目[官网](http://logging.apache.org/log4j/2.x/)，在java 领域里面资格较老，应用非常广泛；log4j高度可配置，通过外部配置文件配置，可以配置日志级别，日志输出格式，输出目的地等。

log4j 核心三要素：

- loggers ：负责捕获记录信息
- appenders ：负责发布日志信息到不同的目的地
- layouts ：负责格式化日志信息

### Logback

logback是log4j创始人设计的有一个开源的日志组件，目标是代替log4j，[官网](http://logback.qos.ch/)

logback核心三要素：

- logback-core ：基础模块
- logback-classic：log4j 的一个改良版本，完整实现了SLF4J API ，可以很方斌啊的更换成其他日志系统如log4j；
- logback-access : 访问模块和 servlet 容器继承提供http的访问日志；

### Log4j2

代替log4j和logback的日志组件。[官网](http://logging.apache.org/log4j/2.x/) 

log4j架构图：

![](https://ws1.sinaimg.cn/large/9cd40bd3gy1g72gcfppx7j20l60czaf5.jpg)

Log4j2优势：

- Log4j2 旨在用作审计日志记录框架。Log4j 1.x 和 Logback 都会在重新配置时丢失事件。Log4j 2 不会。在 Logback 中，Appender 中的异常永远不会对应用程序可见。在 Log4j 中，可以将 Appender 配置为允许异常渗透到应用程序。
- Log4j2 在多线程场景中，异步 Loggers 的吞吐量比 Log4j 1.x 和 Logback 高 10 倍，延迟低几个数量级。
- Log4j2 对于独立应用程序是无垃圾的，对于稳定状态日志记录期间的 Web 应用程序来说是低垃圾。这减少了垃圾收集器的压力，并且可以提供更好的响应时间性能。
- Log4j2 使用插件系统，通过添加新的 Appender、Filter、Layout、Lookup 和 Pattern Converter，可以非常轻松地扩展框架，而无需对 Log4j 进行任何更改。
- 由于插件系统配置更简单。配置中的条目不需要指定类名。
- 支持自定义日志等级。
- 支持 lambda 表达式。
- 支持消息对象。
- Log4j 和 Logback 的 Layout 返回的是字符串，而 Log4j2 返回的是二进制数组，这使得它能被各种 Appender 使用。
- Syslog Appender 支持 TCP 和 UDP 并且支持 BSD 系统日志。
- Log4j2 利用 Java5 并发特性，尽量小粒度的使用锁，减少锁的开销。



