---
title: 代理模式面试知识点(持续更新)
tags: [proxy, interview]
abbrlink: f72b9de4
date: 2021-03-13 11:04:15
---

### 1.代理模式概念

为某个对象提供一个代理,以控制这个对象的访问;代理类负责为委托类预处理消息,过滤消息并转发消息,以及进行消息被委托类执行后的后续处理

通过代理类,能有效控制对委托类对象的直接访问,也可以很好的隐藏和保护委托类对象,同时也为实施不同控制策略预留了空间,从而在设计上获得了更大的灵活性

### 2.静态代理和动态代理的区别

- 静态代理要求委托类和代理类都实现同一个接口,代理对象的一个接口只服务于一种类型的对象

- 静态代理中代理类在编译期就已经确定,而动态代理则是运行期间动态生成

- 静态代理的效率相对来说更高一些,但是代码冗余大,一旦需要修改接口,代理类和委托类都需要修改

- 动态代理依靠反射来生成代理类

### 3.jdk 动态代理实现步骤

1.jdk 动态代理只能用于代理接口,因此需要先创建一个接口以及其实现类

2.创建代理类,实现 invocationHandler 接口

3.引入 Object 类作为目标对象,重写有参构造器,重写 invoke 方法,加入代理方法,并调用 Object 类的方法

4.利用 Proxy.newInstance()生成代理对象,并调用目标方法

### 4.cglib 动态代理实现步骤

CGLIB（Code Generation Library）是一个高性能开源的代码生成包,采用非常底层的字节码技术,对指定的目标类生成一个子类,并对子类进行增强

1.创建被代理类

2.创建代理类,实现 MethodInterceptor 接口,实现 intercept 方法

3.利用 Enhancer 绑定被代理对象和代理类,

4.利用 Enhancer.create()方法创建代理对象,调用代理对象方法

### 5.反射应用场景

- 逆向代码

- 注解

- 动态生成类

- 生成动态代理