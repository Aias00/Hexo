---
title: volatile关键字面试知识点（持续更新）
tags:
  - volatile
  - interview
abbrlink: d1722f66
date: 2021-05-18 13:28:38
---

volatile 变量具有以下特性：

- 可见性：任意对一个 volatile 变量的读，总能看到任意线程对这个 volatile 变量的最后的写入
- 2.原子性：对任意单个 volatile 变量的读/写操作具有原子性，但类似于 volatile++这类复合操作无法保证原子性

volatile 写的内存语义如下：

- 当写一个 volatile 变量时，JMM 会把该线程对应的工作内存的共享变量的值刷新到主内存

volatile 读的内存语义如下：

- 当读一个 volatile 变量时，JMM 会把该线程对应的工作内存的共享变量缓存置为无效，强制从主内存读取

### volatile 实现原理

volatile 在执行变量写之后执行 Lock 指令，将变量实时写入内存而不是处理器缓存，其他处理器通过缓存一致性协议嗅探到此变量的变更，将本地缓存置为无效，由此实现可见性

在 JMM 中是通过内存屏障实现的

- 在 volatile 写之前插入 storestore 屏障，防止上面的写操作与下面的 volatile 写重排序
- 在 volatile 写之后插入 storeload 屏障，防止上面的 volatile 写与后面的读操作重排序
- 在 volatile 读之后插入 loadload、loadstore 屏障，防止上面的 volatile 读与下面的普通读、volatile 写和普通写重排序

还有 happens-before，对于一个 volatile 变量的写 happens-before 于后续任意对这个 volatile 的读

由此实现有序性

![](https://i.loli.net/2021/04/06/ck86Z1XgFNCtmqy.png)
![](https://i.loli.net/2021/04/06/9tsXI6Kz1bhR7vB.png)
![](https://i.loli.net/2021/04/06/2JmDeF8xRsOrXib.png)
![](https://i.loli.net/2021/04/06/gh86vLoAFsPYZmW.png)

### volatile 指令重排序

volatile 变量的内存可见性是基于内存屏障(Memory Barrier)实现。JMM 内部会有指令重排，并且会有 as-if-serial 和 happens-before 的理念来保证指令重排的正确性。内存屏障就是基于 4 个汇编级别的关键字来禁止指令重排的，volatile 的重排规则如下：

1.第一个为读操作时，第二个任何操作不可重排到第一个操作前面

2.第二个为写操作时，第一个任何操作不可重排到第二个操作后面

3.第一个为写操作时，第二个的读写操作也不运行重排序

![](https://i.loli.net/2021/04/06/G5lCHmaKWMo6SNg.png)

### happens-before 规则

- 程序顺序规则：一个线程中的每个操作，happens-before 于该线程中的任意后续操作
- 监视器锁规则：对一个锁的解锁，happens-before 于后续对这个锁的加锁
- volatile 变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 变量的读
- 传递性：如果 A happens-before 于 B，B happens-before 于 C，则 A happens-before 于 C
