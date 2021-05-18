---
title: 我是如何使用策略模式的
tags: [设计模式]
abbrlink: f4f8ffc6
date: 2020-08-10 21:51:20
---

### 关于

在工作中遇到数据类型不同,需要采用不同的方式处理,常见的写法是根据条件进行判断,采用 if...else... 或者 switch 的方式

比如我最近在写的工单,不同类型工单需要的数据和处理逻辑各有区别

由于工单类型比较多,想到要写一堆 if...else...我就头大,这样的代码太丑,我不喜欢...

略一思考,用策略模式去实现,就清爽很多

### 步骤

首先我们需要定义一个接口,定义一个方法,然后由各个类型工单的具体处理类去实现,代码:

```java

public interface OrderHandler{
  public void handle(Order order);
}

```

然后根据工单类型去写出各自相应的实现类,比如加油工单、洗车工单

```java

public class JiayouOrderHandler implements OrderHandler{

  @Override
  public void handle(Order order){
    System.out.println("去加油");
    // ...
  }

}

public class CleanOrderHandler implements OrderHandler{

  @Override
  public void handle(Order order){
    System.out.println("去洗车");
    // ...
  }

}

```

然后需要根据不同工单类型定义好枚举,将工单类型和实际的操作类去做一个绑定

```java
public enum OrderTypeEnum{

  JIAYOU("JIAYOU",new JiayouOrderHandler()),
  XICHE("XICHE",new CleanOrderHandler()),
  ;

  public String type;

  public OrderHandler handler;

  OrderTypeEnum(String type, OrderHandler handler) {
        this.type = type;
        this.handler = handler;
  }

  // 根据传入的工单类型匹配到具体的操作类
  public static OrderHandler getHandler(String type){
      OrderTypeEnum[] values = OrderTypeEnum.values();
      for (OrderTypeEnum value : values) {
          if(type.equals(value.type))){
              return value.handler;
          }
      }
      return null;
  }

  public String getType() {
      return type;
  }

  public OrderHandler getHandler() {
      return handler;
  }

}
```

然后调用的地方

```java

public static void main(String[] args){

  Order order = new Order();
  order.type = "JIAYOU";
  // ...
  OrderHandler handler = OrderTypeEnum.getHandler(order.getType());
  if(null == handler){
    System.out.println("不支持的工单类型");
    return ;
  }
  handler.handle(order);

}

```

没有一堆的 if...else...,真开心!
