---
title: cas
abbrlink: 3ad60b
date: 2021-04-12 15:14:34
tags: [cas]
---

CAS 全称 Compare And Swap（比较与交换），是一种无锁算法。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。java.util.concurrent 包中的原子类就是通过 CAS 来实现的

CAS 算法涉及到 3 个操作数：

- 需要读写的内存值 V
- 进行比较的值 A
- 要写入的新值 B

当且仅当 V 的值等于 A 时，CAS 通过原子方式用新值 B 来更新 V 的值（"比较"+"更新"，整体是一个原子操作），否则不会执行任何操作。一般情况下，"更新"是一个不断重试的操作

AtomicInteger 源码：

![2021-04-12-15-24-1120210412152410](https://i.loli.net/2021/04/12/DfLJnte9r8AS7wV.png)

根据定义我们可以看出各属性的作用：

- unsafe：获取并操作内存的数据
- valueOffset：存储 value 在 AtomicInteger 中的偏移量
- value：存储 AtomicInteger 的 int 值，该属性需要借助 volatile 关键字保证其在线程间是可见的

接下来查看 AtomicInteger 的自增函数 incrementAndGet()的源码，发现自增函数底层调用的是 unsafe.getAndAddInt()。但是由于 JDK 本身只有 Unsafe.class，只通过 class 文件中的参数名，并不能很好的了解方法的作用，所以通过 OpenJDK8 来查看 Unsafe 的源码

```java
// ------------------------- JDK 8 -------------------------
// AtomicInteger 自增方法
public final int incrementAndGet() {
  return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

// Unsafe.class
public final int getAndAddInt(Object var1, long var2, int var4) {
  int var5;
  do {
      var5 = this.getIntVolatile(var1, var2);
  } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
  return var5;
}

// ------------------------- OpenJDK 8 -------------------------
// Unsafe.java
public final int getAndAddInt(Object o, long offset, int delta) {
   int v;
   do {
       v = getIntVolatile(o, offset);
   } while (!compareAndSwapInt(o, offset, v, v + delta));
   return v;
}
```

根据 OpenJDK8 的源码我们可以看出，getAndAddInt 循环获取给定对象 o 中的偏移量处的值 v，然后判断内存值是否等于 v。如果相等则将内存值设为 v+delta，否则返回 false，继续循环进行重试，知道设置成功才能退出循环，并将旧值返回。整个"比较+更新"操作封装在 compareAndSwapInt()中，在 JNI 里是借助一个 CPU 指令完成的，属于原子操作，可以保证多个线程都能看到同一个变量的修改值。

后续 JDK 通过 CPU 的 cmpxchg 指令，去比较寄存器中的 A 和内存中的值 V。如果相等，就把要写入的新值 B 存入内存中。如果不相等，就将内存值 V 赋值给寄存器中的值 A，然后通过 java 代码中的 while 循环再次调用 cmpxchg 指令进行重试，直到设置成功为止

CAS 虽然很高效，但是也存在三大问题：

- ABA 问题：CAS 需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。但是如果内存值原来是 A，后来变成了 B，然后又变成了 A，那么 CAS 进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA 问题的解决思路就是在变量前面加版本号，每次变量更新的时候都把版本号+1，这样变化过程就从"A-B-A"变成了"1A-2B-3A"。
  - JDK 从 1.5 版本开始提供了 AtomicStampedReference 类来解决 ABA 问题，具体操作封装在 compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值
- 循环时间长开销大：CAS 操作如果长时间不成功，会导致其一直自旋，给 CPU 带来很大负担
- 只能保证一个共享变量的原子操作：对一个共享变量执行操作时，CAS 能够保证原子操作，但是对多个共享变量操作时，CAS 是无法保证操作的原子性的
  - java 从 1.5 版本开始提供了 AtomicStampedReference 类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行 CAS 操作

### 问题

1.假设有多个线程同时在对同一块内存进行 CAS 操作的话：两个线程 T1、T2 同时执行，V 存储同一块内存地址，A 当然也是旧的预期值，那么这种情况下 T1 和 T2 是否都可以更新成功

在多个线程同时 CAS 的情况下是不会发生多个线程 CAS 成功的情况的，因为计算机底层实现保证了 V 指向内存的互斥性和立即可见性，可以理解为 CAS 操作是底层保证的线程安全

一个线程 T 在 CAS 操作时，其他线程无法访问 V 指向的内存地址，并且一旦 T 更新了 V 指向内存的值，其他所有线程的 V 指向内存都变得无效

- 处理器实现原子操作有两种做法
  - 一是总线锁：在多 CPU 下，当其中一个处理器要对共享内存进行操作的时候，在总线上发出一个 LOCK#信号，这个信号使得其他处理器无法通过总线来访问到共享内存中的数据
  - 二是缓存锁：如果共享内存已经被缓存，那么锁总线没有意义。缓存锁核心是使用了缓存一致性协议，如 MESI 协议
    - MESI 表示缓存行的四种状态
      - M(Modify)：表示共享数据只缓存在当前 CPU 缓存中，并且是被修改状态，也就是缓存的数据和主内存的数据不一致
      - E(Exclusive)：表示缓存的独占状态，数据只缓存在当前 CPU 缓存中，并且没有被修改
      - S(Shared)：表示数据可能被多个 CPU 缓存，并且各个缓存中的数据和主内存数据一致
      - I(Invalid)：表示缓存已经失效
    - 在 MESI 协议中，每个缓存的缓存控制器不仅知道自己的读写操作，而且也监听(snoop)其他 cache 的读写操作
    - CPU 在读数据时，如果缓存行状态是 I，则需要从主内存中取出，并把缓存行状态置为 S；如果不是 I，则可以直接读取缓存中的值，但在此之前必须要等待对其他 CPU 的监听结果，如果其他 CPU 也有该数据的缓存并且状态是 M，则需要等待其把缓存更新到内存后再读取
    - CPU 可以将状态为 M/E/S 的缓存写入内存，其中如果缓存行状态为 S，则其他 CPU 缓存了相同数据的缓存行会无效化
