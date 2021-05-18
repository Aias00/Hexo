---
title: 2021年春季面试记录
abbrlink: eba5d7a5
date: 2021-05-18 14:02:54
tags: [interview]
---

## 百度

### 百度一面

- rabbitmq 如何保证消息不丢失
- 双亲委派机制
- 类加载器如何确定自己能加载哪些类
- mysql innodb 引擎索引的数据结构
- redis 实现 缓存带过期时间及淘汰策略
- redis 集群
- 共识算法
- 分布式不用 queue 排队，如何实现抢工单
- 分布式锁
- dubbo 支持的协议，用的哪个协议，序列化，服务注册和发现
- springboot 和 springmvc 的区别
- 算法：倒排链表
- JVM 结构
- 垃圾回收算法
- 异常的类别
- 设计模式，用到了哪些
- mybatis 用了哪些数据结构
- sql 优化
- mysql 索引最左匹配原则
- apollo 分布式配置
- threadlocal

### 百度二面

- rabbitmq 如何保证消息不丢失，rabbitmq 顺序消费
- 加载一个类涉及到哪些内存区域
- dubbo 如何实现泛化（不引入 jar 包如何调用提供者的方法）
- 共识算法
- 分布式锁（redis、zookeeper）
- mysql 为什么用自增主键
- mysql 如何解决幻读（mvcc）
- 一些 Linux 操作指令
- mysql 客户端和服务器如何通信的，可以被中断吗
- hashmap 为什么用红黑树，为什么不用二叉树，为什么不用平衡二叉树
- A、B 两个数组，合并后排序（二分排序）
- concurrentHashMap1.7、1.8
- countDownLatch
- try 里面 return，finally 里面 return，会返回什么，finally 什么时候执行的
- spring bean 什么时候创建的，什么时候销毁的
- static/private static/final 哪些是线程安全的
- threadLocal
- redis 为什么快
- mysqL innodb 叶子节点的特点，mysql 主键如何存储的
- CAS 会出现同时替换的情况吗
- 92843 找到离他最近比他大的数
- sql 查出一个学生的平均分最大的

## 快手

### 快手一面

- 自我介绍
- 业务规模
- 联合索引在 mysql 中怎么存储
  - 叶子节点和非叶子节点都存储什么
- MVCC
- spring 事务隔离级别
- @Transactional 注解什么情况下会失效
- JVM 垃圾回收器了解吗
- redis 中 string 的底层结构
- 线程池提交任务流程
- 删除除了编号不同，其他都相同的学生的冗余信息
  - 自动编号 学号 姓名 课程 分数
  - id number name course score
  - 1 2005001 张三 数学 69
  - 2 2005002 李四 语文 89
  - 3 2005001 张三 数学 69
- 求无重复最大子串的长度
  - aabbcdefghhijk bcdefgh 7

## 火花思维

### 火花思维一面

- 自我介绍
- 业务规模
- redis 哪些数据结构，用过哪些，布隆过滤器用过吗
- 一次删除含有 500W 个 key 的 set（unlink）
- redlock 锁，如何实现阻塞（pub/sub）
- 题库中有 100 道题，如何随机插入到数据库
- name 和 age，做联合索引哪个在前
- 商家 id，用户 id，商品 id，如何分库分表，商家可以看到所有他卖出的商品，用户可以看到所有他买到的商品
- ReentrantLock和synchronized区别

### 火花思维二面

- vhost 有什么用
- 延时消息
- 消息幂等
- 用了什么类型的队列
- redis 淘汰策略
- redis 数据结构，用了哪些
- mysql 乐观锁和悲观锁举例
- 分布式事务
