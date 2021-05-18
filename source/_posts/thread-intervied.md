---
title: 多线程面试知识点(持续更新)
tags: [thread, interview]
abbrlink: 1bbbbb30
date: 2021-03-13 14:48:38
---

### 1.线程间同步

#### 为何要使用同步?

java 允许多线程并发控制,当多个线程同时操作一个可共享的资源变量时,将会导致数据的不准确,相互之间产生冲突,因此可以加入同步锁以避免在该线程没有完成对数据的操作之前,数据被其他线程调用,从而保证了该变量的唯一性和准确性

#### 同步的方式

1.同步方法或同步代码块:synchronized 关键字或 Lock 锁

2.volatile 关键字

3.使用 ThreadLocal

4.阻塞队列

5.使用原子变量

### 2.ThreadLocal

#### ThreadLocal 是什么

ThreadLocal 是一个本地线程副本变量工具类;主要用于将私有线程和该线程存放的副本对象做一个映射,各个线程之间的变量互不干扰,在高并发场景下,可以实现无状态的调用,适合各个线程不共享变量值的操作

#### ThreadLocal 工作原理

每个线程内部都维护了一个 ThreadLocalMap,是一个键值对数据格式,key 是一个弱引用,也就是 ThreadLocal 本身,value 存的是线程变量的值

也就是说 ThreadLocal 本身并不存储线程的变量值,它只是一个工具,用来维护线程内部的 Map,帮助存和取变量

#### ThreadLocal 如何解决 Hash 冲突

与 HashMap 不同,ThreadLocalMap 结构非常简单,没有 next 引用,也就是说 ThreadLocalMap 中解决 hash 冲突的方式并非链表的方式,而是采用线性探测的方式.所谓线性探测,就是根据初始 Key 的 hashcode 值确定元素在 table 数组中的位置,如果发现这个位置上已经被其他 key 值占用,则利用固定的算法寻找一定步长的下个位置,依次判断,直至找到能够存放的位置

```java
/
 * Increment i modulo len.
 */
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}

/
 * Decrement i modulo len.
 */
private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

#### ThreadLocal 的内存泄露是怎么回事

ThreadLocal 在 ThreadLocalMap 中是以一个弱引用身份被 Entry 中的 key 引用的,因此如果 ThreadLocal 没有外部强引用来引用它,那么 ThreadLocal 会在下次 JVM 垃圾回收时被回收;这个时候 Entry 中的 Key 已经被回收,但是 value 又是一强引用所以不会被垃圾回收器回收,这样 ThreadLocal 的线程如果一直持续运行,value 就一直得不到回收,这样就会发生内存泄漏

如何解决:

使用完 ThreadLocal 后,及时调用 remove()方法释放内存空间

#### 为什么 ThreadLocal 的 key 是弱引用

- key 使用强引用,这样会导致一个问题:引用 ThreadLocal 的对象被回收了,但是 ThreadLocalMap 还持有 ThreadLocal 的强引用,如果没有手动删除,ThreadLocal 不会被回收,则导致内存泄漏

- key 使用弱引用,这样的话,引用 ThreadLocal 的对象被回收了,由于 ThreadLocalMap 持有 ThreadLocal 的弱引用,即使没有手动删除,ThreadLocal 也会被回收;value 在下一次 ThreadLocalMap 调用 set,get,remove 的时候会被清除

比较以上两种情况,我们可以发现:由于 ThreadLocalMap 的生命周期跟 Thread 一样长,如果都没有手动删除对应 key,都会导致内存泄漏,但是使用弱引用可以多一层保障,弱引用的 ThreadLocal 不会内存泄漏,对应的 value 在下一次 ThreadLocalMap 调用 set,get,remove 的时候被清除,算是最优的解决方案

#### ThreadLocal 的应用场景

会话管理

### CountDownLatch 与 CyclicBarrier 区别

CountDownLatch: 一个或者多个线程,等待其他线程完成某件事情之后才能执行

CyclicBarrier: 多个线程互相等待,直到到达同一个同步点,再继续一起执行

CountDownLatch 是递减计数,CyclicBarrier 是递增计数

CountDownLatch 不可重复利用,CyclicBarrier 可以重复利用

CountDownLatch 初始值为 N,N>0,CyclicBarrier 初始值 N=0

CountDownLatch 调用 countDown(),N-1,CyclicBarrier 调用 await(),N+1

CountDownLatch 在 N>0 时,调用 await() 一直阻塞,CyclicBarrier 在 N 小于指定值时,一直阻塞

CountDownLatch 在计数为 0 时释放等待线程,CyclicBarrier 在计数达到指定值时释放等待线程

### 为什么要使用线程池

- 管理线程,避免增加创建线程和销毁线程的系统资源消耗
- 提高相应速度
- 资源重复利用

### 线程池核心的执行流程

- 提交任务,判断核心线程是否已满,未满则创建新的核心线程执行任务
- 如果核心线程池已满,判断队列是否已满,未满则将任务存入队列
- 如果队列已满,判断非核心线程数是否已满,未满则创建非核心线程执行任务
- 如果非核心线程已满,执行拒绝策略

### 线程池提供的四种拒绝策略

- AbortPolicy(抛出一个异常,默认策略)
- DiscardPolicy(直接丢弃任务)
- DiscardOldestPolicy(丢弃队列里最老的任务,将当前任务继续存入队列)
- CallerRunsPolicy(由调用线程池所在线程执行任务)

### 几种工作阻塞队列

- ArrayBlockingQueue(用数组实现的有界阻塞队列,按 FIFO 顺序)
- LinkedBlockingQueue(基于链表结构的阻塞队列,按 FIFO 排序,可以选择性设置容量,不设置默认为 Integer.MAX_VALUE,即无界队列)
- DelayQueue(支持延迟获取元素的阻塞队列)
- PriorityBlockingQueue(具有优先级的无界队列)
- SynchronousQueue(一个不存储元素的阻塞队列,每个插入操作必须等到另外一个线程执行移除操作,否则插入一直处于阻塞状态)

### 线程生命周期

- NEW:创建后尚未启动的线程处于此状态
- RUNNABLE:包括操作系统线程状态中的 Running 和 Ready,也就是线程可能处于执行中状态,也可能正在等待 CPU 为之分配执行时间
- WAITING:无限期等待状态,处于此状态的线程不会被分配 CPU 执行时间,需要等待被其他线程显式唤醒(notify/notifyAll),主要包括:没有显示设置 timeout 的 Object.wait();没有显示设置 timeout 的 Thread.join();LockSupport.park();
- TiMED_WAITING:有限期等待状态,处于此状态的线程不会被分陪 CPU 执行时间,无须等待其他线程显式唤醒,在一定时间之后会自动唤醒,主要包括:Thread.sleep(timeout);Object.wait(timeout);Thread.join(timeout);LockSupport.parkNanos(timeout);LockSupport.parkUnit(timeout)
- BLOCKED:阻塞状态,处于此状态的线程不会被分配 CPU 执行时间,发生情况:竞争对象监视器锁或 Object.wait()结束后重新竞争对象监视器控制权
- TERMINATED:终止状态,线程执行完毕,生命周期结束

### 阻塞与等待的区别

#### 阻塞

当一个线程试图获取对象锁(非 JUC 中的锁,即 synchronized),而该锁被其他线程持有,则该线程进入阻塞状态;它的特点是使用简单,由 JVM 调度器来唤醒自己,而不需要另一个线程来显式唤醒自己,不响应中断

#### 等待

当一个线程等待另一个线程通知调度器一个条件时,该线程进入等待状态;它的特点是需要等待另外一个线程显式地唤醒自己,实现灵活,语义更丰富,可响应中断;例如调用:Object.waite(),Thread.join().以及等待 Lock 或 Condition

虽然 synchronized 和 JUC 里的 Lock 都实现锁的功能,但线程进入的状态是不一样的;synchronized 会让线程进入阻塞状态,而 JUC 中的 Lock 是用 park()/unpark()来实现阻塞/唤醒的,会让线程进入等待状态;虽然等锁时进入的状态不一样,但被唤醒后又都进入 Runnable 状态,从行为效果来看是一样的

### yield()和 sleep() 的区别

1. yield 和 sleep 都能暂停当前线程,都不会释放锁资源,sleep 可以指定具体休眠的时间,而 yield 依赖 cpu 的时间划分

2. sleep 给其他线程运行机会时不考虑线程的优先级,因此会给低优先级的线程以运行的机会;yield 方法只会给相同或者更高优先级的线程以运行的机会

3. 调用 sleep 方法使线程进入等待状态,等待睡眠时间到达,而调用 yield 方法,线程会进入就绪状态,也就是 sleep 需要等待设置的时间后才会进入的就绪状态,而 yield 会立即进入就绪状态

4. sleep 方法会抛出 InterruptedException,而 yield 不抛出任何异常

5. yield 不能被中断,而 sleep 方法可以接受中断

6. sleep 方法比 yield 方法有更好的移植性

### wait 和 sleep 的区别

1. wait 来自 Object,sleep 来自 Thread

2. wait 释放锁,sleep 不释放锁

3. wait 必须在同步代码块中使用,sleep 不需要

4. wait 不需要捕获异常,sleep 需要

### 死锁

#### 产生条件

- 互斥条件: 一个资源或者说一个锁只能被一个线程占用,当一个线程首先获取到这个锁之后,在该线程释放该锁之前,其他的线程都无法获取到这个锁

- 占有且等待: 一个线程已经获取到一个锁,再获取另外一个锁时,即使获取不到也不会释放已经获得的锁

- 不可剥夺条件: 任何一个线程都无法强制获得其他线程已经拥有的锁

- 循环等待条件: 线程 A 拿着线程 B 想要获取的锁,线程 B 拿着线程 A 想要获取的锁

#### 如何避免

- 加锁顺序: 线程按照相同的顺序加锁

- 限时加锁: 线程获取锁的过程中限定一些时间,如果给定的时间内无法获取锁就放弃,这里需要用到 Lock 的 api

### wait 虚假唤醒

### notify 底层

#### 为何 wait 和 notify 要依赖 synchronized

synchronized 代码块通过 javap 生成的字节码中包含 monitorenter 和 monitorexit 指令,执行 monitorenter 可以获得对象的 monitor,而 wait 方法通过调用 native 方法 wait(0)实现,wait(0)方法要求线程必须拥有对象的 monitor

#### notify 执行后唤醒的线程立马被 CPU 执行吗

notify/notifyAll 调用时并不会真正释放对象锁,只是把等待中的线程唤醒然后放入到对象的锁池中,但是锁池中的所有线程都不会马上运行,只有拥有锁的线程执行完代码块释放锁,别的线程拿到锁之后才能被执行
