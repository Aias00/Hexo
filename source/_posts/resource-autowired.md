---
title: 注解@Resource和@Autowired的区别
tags: [spring]
abbrlink: 759d9b9f
date: 2020-08-21 23:18:56
---

在应用 spring 框架进行开发时,我们常常用@Resource 和@Autowired 注解进行依赖注入

在开发过程中仿佛这两者应用起来并没有什么区别,但是其实这两个注解本质上就是不一样的

区别如下:

- @Resource 是由 java 提供的注解,而@Autowired 是由 spring 提供的

- @Resource 是 ByName,而@Autowired 是 ByType

- @Autowire 注入时 ByType 如果要使用 ByName 需要配合@Qualifier

```java
  @Autowire
  @Qualifier（"orderService"）
  private OrderService orderService;

```

- @Resource 默认 ByName,当找不到与名称匹配的 bean 才会按照类型装配,可以通过 name 属性指定,如果没有指定 name 属性，当注解标注在字段上,即默认取字段的名称作为 bean 名称寻找依赖对象，当注解标注在属性的 setter 方法上,即默认取属性名作为 bean 名称寻找依赖对象

- 注意：如果没有指定 name 属性,并且按照默认的名称仍然找不到依赖的对象时候,会回退到按照类型装配,但一旦指定了 name 属性,就只能按照名称装配了
