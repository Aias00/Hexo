---
title: synchronized最好的锁对象
abbrlink: b56d6cb5
date: 2021-04-17 20:26:14
tags: [synchronized]
---

之前在网上看到一个面试题目：synchronzied 最好的锁对象是什么

当时第一反应是 new Object，但是后来看到这篇文章：[Object vs byte[0] as lock](https://stackoverflow.com/questions/2120437/object-vs-byte0-as-lock)

大意就是 new byte[0]作为锁对象会更好，能够减少字节码操作的次数

下面来验证一下

```java
public class ObjectSynchronizedDemo {
    private final Object obj = new Object();
}

public class ArraySynchronizedDemo {
    private final byte[] lockObj = new byte[0];
}

```

```class
ObjectSynchronizedDemo.class
{
  public com.aias.learn.test.ObjectSynchronizedDemo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: new           #2                  // class java/lang/Object
         8: dup
         9: invokespecial #1                  // Method java/lang/Object."<init>":()V
        12: putfield      #3                  // Field obj:Ljava/lang/Object;
        15: return
      LineNumberTable:
        line 7: 0
        line 8: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      16     0  this   Lcom/aias/learn/test/ObjectSynchronizedDemo;
}

ArraySynchronizedDemo.class
{
  public com.aias.learn.test.ArraySynchronizedDemo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: iconst_0
         6: newarray       byte
         8: putfield      #2                  // Field lockObj:[B
        11: return
      LineNumberTable:
        line 7: 0
        line 8: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      12     0  this   Lcom/aias/learn/test/ArraySynchronizedDemo;
}

```

可以看出，new byte[0]确实比 new Object() 少 4 条字节码操作。

再计算一下内存占用情况：

new Object()内存占用：

Mark Word(8 字节)+klass pointer(开启指针压缩，4 字节)+对齐填充(4 字节) = 16 字节

new byte[0]内存占用：

Mark Word(8 字节)+klass pointer(开启指针压缩，4 字节)+数组长度(int，4 字节) = 16 字节

所以两种锁对象占用内存是相等的。
