---
title: spring面试知识点(持续更新)
tags: [spring, interview]
abbrlink: adbddb32
date: 2021-03-27 16:38:12
---

### spring 是如何实现 ioc 的

### spring bean 什么时候初始化,什么时候销毁的

### spring lazy-init 注解

lazy-init 是 Spring 中延迟加载 bean 的属性

设置为 lazy-init=true 的 bean 将不会在 ApplicationContext 启动时提前被实例化,而是在第一次向容器通过 getBean 索取 bean 实例时实例化的

如果一个设置了立即加载的 bean1 引用了一个延迟加载的 bean2,那么 bean1 在容器启动时会被实例化,而 bean2 由于被 bean1 引用,也会被实例化,这种情况也符合延迟加载的 bean 在第一次被调用时才实例化的规则

lazy-init 只在 scope 属性为 singleton 时才会有效,如果 scope 属性值为 pototype,那么即使设置了 lazy-init="false",容器启动时也不会被实例化,而是 getBean 方法实例化

### 拦截器（Interceptor）和过滤器（Filter）的执行顺序和区别

1.过滤器,依赖于 servlet 容器,基于函数回调实现,可以对几乎所有请求过滤,但是缺点是一个过滤器实例只能在容器初始化时调用一次,使用过滤器的目的,是用来做一些过滤操作,比如获取我们需要的数据,或者提前设置一些参数

2.拦截器,依赖于 web 框架,基于 java 反射机制实现,属于 aop 的一种应用,就是在 service 或者一个方法前,或者方法后等多个位置进行拦截,同一个拦截器实例在一个 controller 生命周期中可以多次调用,但是缺点是只能对 controller 请求进行拦截,对于其他一些比如直接访问静态资源的请求则没有办法进行拦截
