---
title: dubbo面试知识点(持续更新)
tags: [dubbo, interview]
abbrlink: 33ce6d02
date: 2021-03-09 14:59:07
---

### 1.dubbo 负载均衡策略

- 随机模式 RandomLoadBalance：按权重设置随机概率,在一个截面上碰撞的概率较高,但调用量越大分布越均匀,而且按概率使用权重后也比较均匀,有利于动态调整提供者权重

- 轮询模式 RoundRobinLoadBlance：按公约后的权重设置轮询比例,但存在响应慢的服务提供者会堆积请求

- 最少活跃调用 LeastActiveLoadBlance：相同活跃数的随机,活跃数指调用前后计数差,使慢的提供者收到更少请求,因为越慢的提供者的调用前后计数差会越大

- 一致性 hash 调用 ConsistentHashLoadBalance：相同参数的请求总是发到统一提供者,当某台提供者挂掉时,原本发往该提供者的请求,基于虚拟节点,平摊到其他提供者,不会引起剧烈变动

### 2.dubbo 集群容错

- Failover cluster：失败重试,当服务消费者调用服务提供者失败后自动切换到其他服务提供者进行重试,通常用于读操作或者具有幂等的写操作,需要注意的是重试会带来更长延迟.可通过 retries=2 来设置重试次数（不含第一次）

- Failfast cluster：快速失败,当服务消费方调用服务提供者失败后,立即报错,也就是只调用一次,通常这种模式用于非幂等性的写操作

- Failsafe cluster：失败安全,当服务消费者调用服务提供者出现异常时,直接忽略异常.这种模式通常用于写入审计日志等操作

- Failback cluster：失败自动恢复.当服务消费者调用服务提供者出现异常之后,在后台记录失败的请求,并按照一定的策略后期再进行重试,这种模式通常用于消息通知操作

- Forking cluster：并行调用.当消费者调用一个接口方法后,dubboclient 会并行调用多个提供者提供的服务,只要一个成功返回,这种模式通常用于实时性要求较高的读操作,但需要浪费更多服务资源,可通过 forks=2 来设置最大并行数

- Broadcast cluster：广播调用.当消费者调用一个接口方法后,dubboclient 会逐个调用所有服务提供者,任意一台调用异常则这次调用就标志失败,这种模式通常用于通知所有提供者更新缓存或日志等本地资源信息.

### 3.dubbo 支持的协议

#### dubbo 协议（默认）

- 采用单一长连接和 NIO 异步通讯,适合小数据量大并发的服务调用,以及服务消费者机器远大于服务提供者机器数的情况,不适合传送大数据量的服务,比如传文件、传视频等,除非请求量很低;

- 连接个数：单连接,

- 连接方式：长连接,

- 传输协议: TCP,

- 传输方式: NIO 异步传输,

- 序列化方式: Hessian 二进制序列化,

- 适用场景：常规远程服务方法调用

- dubbo 协议缺省每服务每提供者每消费者使用单一长连接,如果数据量较大,可以使用多个连接

- 为防止大量连接撑挂,可以在服务提供方限制大连接接受数,以实现服务提供方自我保护

##### 常见问题

- 为什么要消费者比提供者个数多：因为 dubbo 协议采用单一长连接,假设网络为千兆网卡（1024Mbit=128MByte）,根据测试经验数据每条连接最多只能压满 7MByte,理论上 1 个服务提供者需要 20 个服务消费者才能压满网卡

- 为什么不能传大包：因为 dubbo 协议采用单一长连接,如果每次请求的数据包大小为 500KByte,假设网络为千兆网卡,每条连接最大 7MByte,单个服务提供者的 TPS（每秒处理事务数）最大为：128MByte/500KByte=262.单个消费者调用单个服务提供者的 TPS 最大为：7MByte/500KByte=14.如果能接受,可以考虑使用,否则网络将成为瓶颈

- 为什么采用异步单一长连接：因为服务的现状大都是服务提供者少,通常只有几台机器,而服务的消费者多,可能整个网站都在访问该服务,如果采用常规的 Hessian 服务,服务提供者很容易就被压垮,通过单一连接,保证单一消费者不会压死提供者,长连接,减少连接握手验证,并采用异步 IO,复用连接池,防止 C10K 问题

#### rmi 协议

- 采用 JDK 标准的 java.rmi.\*实现,采用阻塞式短连接和 JDK 标准序列化方式

- 连接个数：多连接

- 连接方式：短连接

- 传输协议：TCP

- 传输方式：同步传输

- 序列化：Java 标准二进制序列化

- 适用范围：传入传出参数数据包大小混合,消费者与提供者个数差不多,可以传文件

- 适用场景：常规远程方法调用,与原生 rmi 服务互操作

#### Hessian 协议

- Hessian 协议用于集成 Hessian 服务,Hessian 底层采用 http 通讯,采用 servlet 暴露服务,dubbo 缺省内置 Jetty 作为服务器实现

- dubbo 的 Hessian 协议可以和原生的 Hessian 服务互操作,即提供者用 dubbo 的 Hessian 协议暴露服务,消费者直接用标准 Hessian 接口调用;或者提供方用标准 Hessian 暴露服务,消费方用 dubbo 的 Hessian 协议调用

- 连接个数：多连接

- 连接方式：短连接

- 传输协议： http

- 传输方式：同步传输

- 序列化： Hessian 二进制序列化

- 适用范围：传入传出参数数据包较大,提供者比消费者个数多,提供者压力较大,可传文件

- 适用场景：页面传输,文件传输,或与原生 Hessian 服务互操作

- 参数及返回值需实现序列化接口

- 参数及返回值不能自定义实现 List,Map,Number,Date,Calender 等接口,只能用 JDK 自带的实现,因为 Hessian 会做特殊处理,自定义实现类中的属性值都会丢失

#### http 协议

- 基于 http 表单的远程调用协议

- 连接个数：多连接

- 连接方式：短连接

- 传输协议：http

- 传输方式：同步传输

- 序列化：表单序列化,即 json

- 适用范围：传入传出参数数据包大小混合,提供者比消费者个数多,可用浏览器查看,可用表单或者 url 传入参数,暂不支持传文件

- 适用场景：需同时给应用程序和浏览器 JS 使用的服务

#### webservice 协议

- 基于 webservice 的远程调用协议,基于 Apache CXF 的 fronted-simple 和 transports-http 实现

- 可以和原生 webservice 服务互操作：提供者用 dubbo 的 webservice 协议暴露服务,消费者直接用标准的 webservice 接口调用;或提供者用标准的 webservice 暴露服务,消费者用 dubbo 的 webservice 协议调用

- 连接个数：多连接

- 连接方式：短连接

- 传输协议：http

- 传输方式：同步传输

- 序列化：soap 文本序列化

- 适用场景：系统集成,跨语言调用

- 参数及返回值需实现序列化接口

- 参数尽量使用基本类型和 POJO

#### thrift 协议

- 当亲 dubbo 支持的 thrift 协议是对 thrift 原生协议的拓展,在原生协议的基础上添加了一些额外的头信息,比如 service name, magic number 等.

- 使用 dubbo thrift 协议同样需要使用 thrift 的 idl compiler 编译生成相应的 Java 代码

- thrift 不支持 null 值,不能在协议中传 null

#### memcached 协议

- 基于 memcached 实现的 rpc 协议

#### redis 协议

- 基于 redis 实现的 rpc 协议

#### rest 协议

- 基于标准的 java rest api--JAX-RS2.0 实现的 REST 调用支持

### 4.dubbo 推荐用什么协议

默认使用 dubbo 协议

### 5.dubbo 默认使用什么序列化框架

dubbo 协议默认使用 Hessian2 序列化

rmi 协议默认使用 java 序列化

http 协议默认为 json

Hessian 协议默认 Hessian 序列化

webservice 协议默认 soap 文本序列化

也可以指定第三方的序列化框架,比如 kryo,FST 等

### 6.SpringCloud 和 dubbo 对比

- dubbo 由于是二进制的传输,占用带宽会更少

- SpringCloud 是 http 协议传输,带宽会比较多,同时使用 Http 协议一般会使用 json 报文,消耗更大

- dubbo 的开发难度较大,原因是 dubbo 的 jar 包依赖问题很多大型工程无法解决

- SpringCloud 的接口协议约定比较自由松散,需要有强有力的行政措施来限制接口无序升级

- dubbo 的注册中心可以选择 zk,redis 等,SpringCloud 的注册中心可以选 eureka 或者 Consul

- SpringCloud 提供了丰富的组件:如服务网关\断路器\分布式配置\链路追踪等
