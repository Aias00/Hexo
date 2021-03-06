---
title: redis两种持久化方式
tags:
  - redis
abbrlink: ab613fab
date: 2021-05-18 13:18:43
---

### 简介

- redis 的高性能是由于其将所有的数据都存储在了内存中，但是如果只是存在在内存中，重启之后数据就会全部丢失，为了尽量减少数据的丢失，我们需要将数据同步到磁盘上，这就是所谓的持久化
- Redis 支持两种方式的持久化，一种是 RDB 模式，一种是 AOF 模式
- RDB 模式可以在指定的时间间隔内为你存储在 redis 中的数据集做快照，子进程将快照数据写入临时文件，然后定时刷新到磁盘，redis 在启动时会将生成的快照文件中的内容读取到内存中
- AOF 模式是将服务器接执行的所有写入/删除指令记录下来，只允许追加操作，并在服务器启动时重构原有的数据集，日志以与服务器协议相同的格式记录，当日志文件过大时，redis 会对日志文件进行重写
- 我们可以不开启持久化配置，这样我们的数据只在 redis 服务启动时存在
- 我们也可以将 RDB 和 AOF 模式组合使用，在这种情况下，redis 服务器重启时会依据 AOF 方式读取数据，因为这种模式能够保证数据相对最完整

### RDB 模式

#### RBD 模式的优点

- RDB 是一种非常便捷的单文件实时存储模式
- RDB 文件非常便于备份，我们可以每小时保存一份过去 24 小时内的 RDB 文件，也可以每天备份一次过去 30 天内生成的 RDB 文件，我们可以保存不同版本的快照文件以避免数据的丢失
- RDB 模式非常适合数据恢复，我们可以将 RDB 文件复制到各种远程服务器进行数据加载
- 开启 RDB 模式，主进程只需要 Fork 出一个子进程来进行持久化操作，这样就可以极大的避免主进程执行 IO 操作了
- 相比于 AOF 模式，如果数据集很大，RDB 的启动效率会更高

#### RBD 模式的缺点

- 相对而言，如果只开启了 RDB 模式，在 redis 服务宕机时，没有刷新到磁盘的临时数据可能会丢失
- RDB 模式经常要通过 Fork 子进程来协助完成持久化工作，当内存中数据集过大时，Fork 操作可能会花费比较多的时间，如果 cpu 性能较差，redis 服务可能会出现几百毫秒甚至 1 秒的时间无法接收数据，AOF 模式也需要进行 Fork 操作但是你可以设置这个操作进行的频率

### AOF 模式

#### AOF 模式的优点

- 使用 AOF 模式可以保证更高的数据安全性
- redis 提供了 3 种同步策略:每秒同步/每次修改同步/不同步
- 每秒同步操作也是异步完成的，效率也很高，如果 redis 服务出现宕机，只会丢失 1 秒内的数据
- 每次修改同步即每次修改操作都会被同步，所以效率是最低的
- AOF 是以追加方式写入文件，在宕机时不会导致文件数据的损坏，即使一条指令只写入了一半，我们也可以利用 redis-check-aof 工具对其进行修复
- 在 AOF 文件过大时，redis 可以自动启用重写机制，redis 会创建一个老文件的最小集合，然后等新的文件创建完成之后交换两个文件开始对新的文件进行追加写入
- AOF 包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作

#### AOF 模式的缺点

- 对于相同的数据集，AOF 文件通常要大过 RDB 文件，RDB 文件在重启时恢复速度要快过 AOF 文件
- 根据同步策略的不同，AOF 在运行效率上往往会慢于 RDB。总之，每秒同步策略的效率是比较高的，不追加的效率和 RDB 模式一样高效

### 常用配置

#### RDB 模配置

Redis 会将数据集的快照 dump 到 dump.rdb 文件中。我们也可以通过配置文件来修改 Redis 服务器 dump 快照的频率，在打开 6379.conf 文件之后，我们搜索 save，可以看到下面的配置信息：

- save 900 1 #在 900 秒(15 分钟)之后，如果至少有 1 个 key 发生变化，则 dump 内存快照
- save 300 10 #在 300 秒(5 分钟)之后，如果至少有 10 个 key 发生变化，则 dump 内存快照
- save 60 10000 #在 60 秒(1 分钟)之后，如果至少有 10000 个 key 发生变化，则 dump 内存快照

#### AOF 持久化配置

在 Redis 的配置文件中存在三种同步方式，它们分别是：

- appendfsync always #每次有数据修改发生时都会写入 AOF 文件
- appendfsync everysec #每秒钟同步一次，该策略为 AOF 的缺省策略
- appendfsync no #从不同步。高效但是数据持久化完全依赖操作系统的策略
