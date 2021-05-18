---
title: redisTemplate反序列化时遇到的问题
tags: [redis]
abbrlink: d84d4fa
date: 2020-08-22 01:01:57
---

最近在开发中使用了 RedisTemplate 来操作 redis 缓存,SpringBoot 集合 RedisTemplate 感觉用起来也不错,同时在开发过程中也遇到了一些问题,在这里记录一下整个过程

### 配置

- 创建 SrpingBoot 项目或者模块工程,引入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-commons</artifactId>
        </dependency>
```

- 添加 redis 配置参数

```yml
spring:
  redis:
    host: localhost
    port: 6379
    jedis:
      pool:
        min-idle: 50
        max-idle: 10
        max-wait: 200
        max-active: 300
```

- 添加配置类,修改 redis 中存储 key 和 value 的序列化和反序列化方式,RedisTemplate 默认配置的是使用 Jdk 序列化,这种序列化方式对存储对象不是很方便

```java
@Configuration
public class RedisConfig {
    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    @Bean
    public RedisTemplate<String, Object> redisTemplateInit() {
        // 设置key的序列化方式为String
        redisTemplate.setKeySerializer(RedisSerializer.string());
        // 设置Hash key的序列化方式为String
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        // new 一个jackson方式的valueSerializer 序列化和反序列化方式都为json
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer =
                new Jackson2JsonRedisSerializer<>(Object.class);
        // 自定义ObjectMapper
        ObjectMapper objectMapper = new ObjectMapper();
        // 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会跑出异常
        objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        // 设置value的序列化方式为json
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        // 设置hash value的序列化方式为json
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
```

这样就配置好了 RedisTemplate

### 问题

记录一下遇到的一些问题

- 存到 redis 中的数据,读取出来之后类型不是原来的类型,而是 LinkedHashMap

在 jackson2JsonRedisSerializer 中没有配置自定义 objectMapper,而 jackson2JsonRedisSerializer 默认的 ObjectMapper 没有配置 DefaultTyping 属性,jackson 将使用简单的数据绑定具体的 java 类型,其中 Object 就会在反序列化的时候变成 LinkedHashMap

- 之前也试过 FastJsonRedisSerializer,利用这种序列化方式,如果没有一个包含无参构造函数或者一个包含全部参数的构造函数,在反序列化之后会出现属性丢失,因为 Serializer 在反序列化时会调用对象类的构造器去进行属性注入

- 用 jackson2JsonRedisSerializer 或者 GenericJackson2JsonRedisSerializer, 如果没有一个包含无参构造函数,反序列化时会报错
