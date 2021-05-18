---
title: redis面试知识点总结(持续更新)
tags: [redis, interview]
abbrlink: 8df47b3e
date: 2021-03-06 14:17:38
---

## redis 数据丢失

### Redis 哨兵，当主节点发生故障时，需要进行主备切换，可能会导致数据丢失

### 异步复制数据导致的数据丢失

主节点异步同步数据给备用节点的过程中，主节点宕机了，导致有部分数据未同步到备用节点。而这个从节点又被选举为主节点，这时候就会有部分数据丢失

### 脑裂导致的数据丢失

主节点所在机器脱离了集群网络，实际上自身还是运行着的。但这时原来的集群中主节点的从节点升级成了主节点，这个时候就有两个主节点都在运行，这就是脑裂

发生脑裂后，客户端还没有来得及切换到新的主节点，连的还是原来的，有些数据还是写入到了第一个节点里面，新的节点没有这些数据。等到第一个节点恢复后，会降级为从节点连接到集群，而且自身数据会被清空，重新从主节点复制数据。而新的主节点因为没有客户端之前写入的数据，所以导致数据丢失了一部分

### 如何避免

- 配置 min-slaves-to-write 1，表示至少有一个从节点

- 配置 min-slaves-max-lag 10，表示数据复制和同步的延迟不能超过 10 秒。最多丢失 10 秒的数据

## redis 为什么这么快

1. 纯内存操作，一般都是简单的存取操作，线程占用的时间很多，时间的花费主要集中在 IO 上，所以读取速度快
2. 整个 redis 就是一个全局 hash 表，他的时间复杂度是 O(1)，而且为了防止 hash 冲突导致链表过长，redis 会执行 reHash 操作，扩充 hash 桶数量，减少 hash 冲突。并且防止一次性重新映射数据过大导致线程阻塞，采用渐进式 rehash，巧妙的将一次性拷贝分摊到多次请求过程中，避免阻塞。
3. redis 采用的是非阻塞 IO，IO 多路复用。采用了单线程来轮询描述符，将数据库的开、关、读、写都转换成了事件，redis 采用自己实现的事件分离器，效率比较高
4. 单线程,省去了上下文切换的消耗,CPU 利用率高
5. redis 全程使用 hash 结构，读取速度快，还有一些特殊的数据结构，对数据存储进行了优化，如压缩表，对短数据进行压缩存储，再如跳表，使用有序的数据结构加快了读写速度
6. 根据实际存储的数据类型选择不同编码

## 1.redis 支持的数据结构

redis 支持 5 种基本数据结构，分别是 string、list、set、zset 和 hash

同时 redis 还支持 3 种特殊的数据类型，分别是 geo、bitmap 和 hyperloglog

string:

二进制安全的数据类型，可以用来存储图片二进制流或者序列化之后的对象，最大上限 512M

可以保存（1）二进制序列字符串（2）整形数据（3）浮点数据

可以用来做原子计数器

可以用来实现分布式锁（拓展：分布式锁）

list:

简单的 string 列表，相当于 java 中的 LinkedList，按照插入顺序排序，写操作时间复杂度是 O(1)，读操作时间复杂度是 O(n)

双向链表，可以头插（LPUSH）或者尾插（RPUSH）

list 最大长度是 2^32-1

发布订阅，可以实现简单的消息队列

通过 list 裁剪功能可以实现排行榜

set:

string 的无序集合，相当于 java 中的 HashSet，set 内部使用 hash 表保证唯一性，添加、删除、查找的时间复杂度都是 O(1)

set 最大成员个数是 2^32-1

可以通过指令实现交集、并集等操作

可以用于实现去重操作

hash:

string 的键值对集合，相当于 java 中的 HashMap，特别适合用来存储对象结构

hash 中最大可以存储 2^32-1 个键值对

可以用来实现购物车

redis 数据库就是由 hash 来实现的

渐进式 rehash：在 redis 中针对哈希冲突采用的是渐进式 rehash 操作，渐进式 rehash 会保留新旧两个 hash 结构，查询时同时间查询两个 hash 结构，在后续的定时 任务及 hash 操作的过程中完成从旧 hash 结构迁移到新 hash 结构的过程

zset:

zset 即 sorted set，是带有排序功能的 string 集合，不允许重复的成员

通过为每个对象设置一个 double 类型的 score（分数）来实现排序，根据 score 值从小到大对对象进行排序，不同的成员可以设置相同的 score

增加、修改和删除元素的时间复制度为 O(logN)

可以实现排行榜

（拓展：skipList）

bitmap(2.2.0 版本新增):

位图，byte 数组，用二进制表示，只有 0 和 1 两个数字，基于 string 实现，节省内存

实现布隆过滤器（拓展：布隆过滤器）

可以用于统计员工每日打卡

统计用户在线状态

hyperloglog(2.8.9 版本新增):

用于基数统计

可以用于埋点统计

geo(3.2 版本新增):

存储地理位置信息，底层由 zset 实现

计算两个坐标点直接的距离

实现附近的人

## 2.与 memcached 比较

redis 支持多种数据类型，memcached 只支持 string 类型

redis 的 string 类型最大可以达到 512M，而 memcached 最大只支持 1M

memcached 只支持把数据存储到内存，而 redis 提供了持久化方式

memcached 本身不支持分布式，需要通过客户端的分布式算法实现分布式集群，而 redis 通过 redis cluster 提供了分布式集群功能（拓展：hash 算法）

memcached 支持多核多线程，而 redis 接收指令只支持单线程

## 3.redis 持久化

redis 提供两种持久化方式，分别是 RDB 和 AOF

RDB:

可以在指定的时间间隔内对存储在 redis 服务器中的某个时间点的数据做快照，主进程会 fork 出一个子进程将快照数据写入临时文件并定期刷新到磁盘，redis 在启动时会读取之前生成的快照文件内容加载到内存中。

AOF:

以指定的时间间隔将服务器执行的所有指令记录下来，以追加的方式写入到指定的日志文件中，当日志文件过大时，redis 会创建子进程对日志文件进行重写。redis 提供了 3 种同步策略：每秒同步/每次修改同步/不同步

比较：

redis 先加载 AOF 文件来恢复原始数据，因为 AOF 数据比 RDB 更完整。

但是 AOF 文件容易被损坏，损坏的 AOF 文件可以通过 redis-check-aof 进行修复

RDB 文件进行了压缩，所以占用体积小，传输速度快，但是如果设置备份时间间隔太长，丢失数据量较大，数据量越大，fork 子进程时间越长，可能阻塞主进程，可读性差

AOF 设置每秒追加一次，如果发生宕机只会丢失 1s 内的数据，生成文件较大，相对于 RDB 效率低，启动恢复慢

混合模式(4.0 版本新增)：

生成新的 AOF 文件前半段是 RDB 文件的全量数据，后半段是 AOF 模式的增量数据。在重启时加载此 AOF 文件进行恢复

## 4.redis 内存淘汰策略

redis 在进行内存淘汰时，采用两种方式：定时扫描主动清除+惰性删除（访问到已经过期的 key 时才进行删除）

noeviction：

当内存达到 maxmemory 限制时，直接报错

volatile-lru：

在所有设置了过期时间的 key 中挑选最近最少使用的数据进行淘汰（拓展：LRU 和 LFU）

volatile-random：

在所有设置了过期时间的 key 中随机挑选一些进行淘汰

volatile-ttl：

在所有设置了过期时间的 key 中，选择一些即将过期的 key 进行淘汰

allkeys-random：

在所有 key 中随机挑选一些进行淘汰

allkeys-lru：

在所有 key 中挑选最近最少使用的数据进行淘汰

volatile-lru, volatile-random 和 volatile-ttl 在没有符合条件的 key 可以被淘汰时，表现和 noeviction 一样

volatile-lfu(4.0 版本新增)：

在所有设置了过期时间的 key 中，挑选最近最不经常使用的 key 进行淘汰

allkeys-lfu(4.0 版本新增):

在所有 key 中挑选最近最不经常使用的 key 进行淘汰

## 5.缓存雪崩/缓存击穿/缓存穿透

缓存雪崩：

缓存数据批量过期，大量的请求穿过缓存直接打到数据库，给数据库带来很大压力，严重的情况可能导致数据库宕机

解决方案：

设置缓存 key 过期时间时增加随机值，避免缓存同时失效

对热点缓存 key 不设置过期时间

本地内存缓存+redis 缓存+服务降级

缓存击穿：

缓存中的热点数据存活时间到期或者被误删除，并发查询此缓存数据的请求穿过缓存打到数据库

解决方案：

对热点缓存 key 不设置过期时间

分布式锁，遇到热点缓存 key 失效时，去请求分布式锁，拿到锁的线程才能去请求数据库，从数据库中查询到有效的数据之后放入缓存，这样后续进来的请求都可以直接从缓存中拿到数据，减轻了数据库压力，也可以避免多个线程同时查询数据库更新缓存，以免造成缓存脏数据

缓存穿透：

查询不存在于缓存也不存在于数据库的数据，这样请求相当于缓存不存在直接请求数据库，当并发请求量大时会对数据库造成很大压力

解决方案：

接口层增加参数校验

缓存空值，设置较短的过期时间

布隆过滤器

## 6.redis 集群模式

单机模式:

优点:简单

缺点:内存容量有限,处理能力有限,无法实现高可用

主从复制:

主从复制允许一个 redis 服务器创建多个复制品,被复制的服务器为 master,从 master 复制的服务器为 slave,master 会将自身的数据通过网络更新同步给 slave

优点:可以通过 master 写入数据,通过 master/slave 读取数据

缺点:无法实现高可用，没有解决 master 写的压力

哨兵 sentinel:

监控 redis 主从服务器,并在 master 下线时自动进行故障转移

监控(Monitoring):sentinel 会不断检查 master 和 slave 是否正常运行

提醒(Notification):当被监控的某个 redis 服务器出现问题时,sentinel 可以通过 API 向管理员或其他应用程序发送通知

自动故障迁移(Automatic failover):当一台 master 不能正常运行时,sentinel 会开始一次自动故障迁移操作,将此 master 的一台 slave 升级成为 master,当原来下线的 master 故障恢复之后,会自动降级为新的 master 的 slave（拓展：哨兵主动切换）

优点:保证高可用，提供节点监控及报警功能

缺点：占用服务器资源，切换需要时间，可能存在丢失数据的风险，没有解决 master 写的压力

集群（代理）:

通过 Twemproxy 或者 codis 第三方代理实现 redis 集群的方式

优点：支持多种 hash 算法，提供第三方功能

缺点：引入代理，维护成本高，failover 逻辑需要自己实现，其本身不能支持故障的自动转移可扩展性差，进行扩缩容都需要手动干预

集群（直连，3.0 版本新增）:

Redis-Cluster 集群，采用无中心结构，每个节点保存数据和整个集群状态,每个节点都和其他所有节点连接。

无中心架构。数据按照 slot 存储在多个节点，节点间数据共享，可以动态调整数据分布（拓展：一致性 hash 算法，hash 槽）。可拓展，节点可以动态添加或删除。高可用，部分节点不可用不影响整体集群，可以通过对节点增加 slave 的方式进行数据备份。实现故障自动 failover,节点间通过 gossip 协议（拓展：gossip 协议）交换状态信息，用投票机制完成主从角色切换。

缺点：资源隔离性较差，容易相互影响。数据通过异步复制，不保证数据的强一致性。

## 7.常见的 redis 客户端

Jedis
比较全面的 redis 操作指令，阻塞 IO，调用方法是同步的，不支持异步，不是线程安全的，所以需要配合连接池来使用

Redission
基于 Netty 框架的事件驱动的通信层，提供 redLock 实现，调用方法是异步的，线程安全，不支持字符串操作，不支持排序、事务、管道、分区等 redis 特性

Lettuce
基于 Netty 框架的事件驱动的通信层，调用方法是异步的，线程安全

## 8.其他 redis 问题

redis 实现分布式锁

单机

加锁：setnx px(ex)

释放锁：lua 脚本原子性操作

优点：简单，单实例安全

缺点：主从或者集群模式下，如果加锁成功后，锁对象未来得及从 master 同步到 slave，slave 挂了，可能也会出现多实例获取到锁的现象。如果锁住的代码块逻辑执行时间过长，超过缓存的过期时间，锁自动释放，相同的逻辑可能会执行多次

集群

redLock（拓展：redLock）

redis 和 mysql 如何保持数据一致性

先读取缓存，缓存中没有再去读取数据库，读取到数据之后将数据维护进缓存

设置缓存过期时间

删除缓存失败重试

1.延时双删

先删除缓存，再更新数据库，休眠一定时间之后再更次删除缓存

2.异步延时删除

先删除缓存，再更新数据库，发送 MQ 消息再次删除缓存

3.监听 binlog

更新数据库，监听程序监听到 binlog 变化，删除缓存，如果删除失败，发送 mq 消息重试，直至删除成功

redis cluster 只支持 db0