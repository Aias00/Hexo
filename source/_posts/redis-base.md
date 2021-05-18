---
title: redis基本数据类型
tags: [redis]
abbrlink: dbc52038
date: 2020-08-02 09:47:49
---

Redis 支持 5 种基本数据类型: string(字符串) / hash(哈希) / list(列表) / set(无序集合) / zset(有序集合)

### 1.string

string 是 Redis 最基本的类型,string 类型是二进制安全的,string 类型可以包含任何数据,例如图片二进制流或者序列化之后的对象

string 类型最大能存储 512M 的内容

![操作string](https://gitee.com/AtlsHY/picgo/raw/master/images/1.png)

### 2.hash

hash 是一个键值对的集合

hash 是一个 string 类型的 field 和 value 的映射表,对应 java 集合中的 map 类型,特别适合用来存储对象结构

每个 hash 可以存储 2^32 -1 个键值对（40 多亿）

![操作hash](https://gitee.com/AtlsHY/picgo/raw/master/images/2.png)

### 3.list

list 是简单的字符串列表，按照插入顺序排序

你可以添加一个元素到列表的头部（左边）或者尾部（右边）,即双向链表

list 最多可存储 2^32 - 1 个元素 (4294967295, 每个列表可存储 40 多亿)

![操作list](https://gitee.com/AtlsHY/picgo/raw/master/images/3.png)

### 4.set

set 是 string 类型的无序集合

set 是通过哈希表实现的,所以添加/删除/查找的复杂度都是 O(1)

添加一个 string 元素到 key 对应的 set 集合中，成功返回 1，如果元素已经在集合中返回 0

集合中最大的成员数为 2^32 - 1(4294967295, 每个集合可存储 40 多亿个成员)

![操作set](https://gitee.com/AtlsHY/picgo/raw/master/images/4.png)

### 5.zset

zset 即 sorted set, 有序集合

zset 和 set 一样也是 string 类型元素的集合,且不允许重复的成员

不同的是每个元素都会关联一个 double 类型的分数(score)。redis 正是通过分数来为集合中的成员进行从小到大的排序

zset 的成员是唯一的,但分数却可以重复

![操作zset](https://gitee.com/AtlsHY/picgo/raw/master/images/5.png)
