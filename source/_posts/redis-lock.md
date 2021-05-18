---
title: redis实现分布式锁-单redis实例实现分布式锁
tags: [redis]
abbrlink: 8dec7256
date: 2020-08-04 21:50:11
---

### 为什么要用分布式锁

在开发过程中,涉及到单服务多实例同时运行,某些操作例如定时任务跑批/更新数据等,为了避免多个实例同时对同一共享资源进行多次操作或者重复操作,我们需要分布式锁来进行并发协调

### 单 Redis 实例实现分布式锁

获取锁使用命令: SET resource_name my_random_value NX PX 30000

这个命令利用了 redis 2.6.12 版本以后提供的 set (ex nx px xx) 指令, 这个指令支持对 set 指令设置参数

其中:

- EX seconds : 将键的过期时间设置为 seconds 秒

- PX: 将键的过期时间设置为 milliseconds 毫秒

- NX: 只有键不存在时,才会对键进行设置操作

- XX: 只有键已经存在时,才对键进行设置操作

这个指令只有在不存在 resource_name 这个 key 的时候才能执行成功,并且为这个 key 设置了一个 30s 的过期时间,这个 key 的值是一个 random_value(随机数)

设置过期时间是为了保证操作线程如果在获取锁之后挂掉了,锁到了过期时间依旧可以自动释放,避免了死锁的情况发生

value 值必须是随机数主要是为了能够更安全地释放锁,释放锁的操作需要结合 lua 脚本实现

```lua
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

这个指令的意思是:只有 key 存在并且存储的值和传入的值一样才能进行删除操作,否则返回 0

这样操作的是为了避免一个实例删除了其他实例获取的锁, 删除锁不能先 get 再 del 因为这两步没有原子性，假设 get 后锁刚好时间到了，过期了，就会把别人上的锁删掉，而 lua 脚本就有原子性的

key 的失效时间，被称作“锁定有效期”。它不仅是 key 自动失效时间，而且还是一个客户端持有锁多长时间后可以被另外一个客户端重新获得

用 Java 简单实现的效果

加锁:

```java
public static boolean tryLock(String key, String randomValue, int seconds) {
    return "OK".equals(jedis.set(key, randomValue, "NX", "EX", seconds));
}
```

解锁:

```java
public static boolean releaseLock(String key, String randomValue) {
    String luaScript = "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "return redis.call('del', KEYS[1]) else return 0 end";
    return jedis.eval(
        luaScript,
        Collections.singletonList(key),
        Collections.singletonList(randomValue)
    ).equals(1L);
}
```

优点:

- 代码简洁,单 redis 实例安全

缺点:

- 只支持单 redis 实例,集群/主从模式下,如果加锁成功后,锁的 value 从 Master 复制到 Slave 的时候挂了,也是会出现同一资源被多个 Client 加锁的

- 如果锁住的代码块逻辑执行时间过长,可能会超过缓存的过期时间,锁自动释放,相同的资源可能会被操作多次
