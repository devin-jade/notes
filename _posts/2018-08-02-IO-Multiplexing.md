---
layout: post
title: IO 初识
categories: [Redis]
tags: I/O
---

## 概述

从上文得知，IO复用可以以尽可能少的资源来监视多个描述符，一单某个描述符就绪（即读就绪或者是写就绪），能够通知应用程序进行相应的读写操作，以便于实现高并发和高吞吐量。IO多路复用有select、poll和epoll三种机制，都是同步IO，在读写事件就绪以后自己完成读写，而这个读写过程是阻塞的。IO多路复用的使用场景：

1. 当客户处理多个描述字时（一般是交互式输入和网络套接口），必须使用I/O复用。
2. 当一个客户同时处理多个套接口时，而这种情况是可能的，但很少出现。
3. 如果一个TCP服务器既要处理监听套接口，又要处理已连接套接口，一般也要用到I/O复用。
4. 如果一个服务器即要处理TCP，又要处理UDP，一般要使用I/O复用。
5. 如果一个服务器要处理多个服务或多个协议，一般要使用I/O复用。

与多进程和多线程技术相比，I/O多路复用技术的最大优势是系统开销小，系统不必创建进程/线程，不必进行线程切换，也不必维护这些进程/线程，从而大大减小了系统的开销。

IO多路复用

## select 模式

select 模式：最早的的IO复用机制，有点是处理简单，缺点是资源利率低。





## epoll 模式





## 参考资料

[多路复用技术之Epoll原理](http://m.lanhusoft.com/Article/707.html)

[IO多路复用原理剖析](https://juejin.im/post/59f9c6d66fb9a0450e75713f)

[epoll原理详解及epoll反应堆模型](https://blog.csdn.net/daaikuaichuan/article/details/83862311)

[select、poll、epoll之间的区别](https://www.cnblogs.com/aspirant/p/9166944.html)







