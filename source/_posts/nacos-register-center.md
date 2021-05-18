---
title: 使用Nacos注册中心功能
tags: [spring-cloud-alibaba-nacos,spring-cloud-alibaba,spring-cloud]
abbrlink: bc887465
date: 2020-12-13 12:35:18
---

开始学习使用 spring-cloud-alibaba

### 启动部署 Nacos

1.下载链接 <https://github.com/alibaba/nacos/releases>

2.选择一个版本下载之后解压,之后进入到 bin 目录

- linux 执行 `sh startup.sh -m standalone`
- windows 执行 `cmd startup.cmd -m standalone`

命令参数中-m 表示模式 mode，standalone 表示启动的是单机版模式

3.启动日志如下
![image-20210401133617644](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401133617644.png)

![image-20210401133938392](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401133938392.png)

从启动日志中我们可以看出:

- Nacos 默认启动的端口号是 8848
- Nacos 本次启动的进程 pid 是 13640
- Nacos 控制台页面访问 url:<http://localhost:8848/nacos/index.html>
- 根据下面一些的日志,很容易就能确定 Nacos 是基于 spring 开发的

4. 访问 Nacos 控制台页面

![image-20210401133917153](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401133917153.png)

默认用户名和密码都是 nacos

### 使用 Nacos 注册中心功能

1.创建一个 maven 父工程,引入 spring-cloud-alibaba 相关依赖

```xml
    <dependencyManagement>
        <dependencies>
            <!--spring-boot-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.4.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- spring-cloud -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- spring-cloud-alibaba -->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.2.1.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

2.创建一个 provider 模块和一个 consumer 模块,引入 alibaba-nacos-discovery 依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--nacos注册中心-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
```

3.为 provider 模块创建服务提供接口,并添加 bootstrap.yml 作为配置文件

```java
@RestController
public class ProviderController {

    @GetMapping("/echo")
    public String echo(String string) {
        return "Hello Nacos Discovery " + string;
    }

}
```

```yml
spring:
  application:
    name: coco-provider
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.70.1:8848 #注册中心地址
server:
  port: 9990
```

4.启动 provider 服务,服务启动成功之后查看 nacos 控制台->左侧服务管理->服务列表,可以看见 provider 服务的相关信息
![image-20210401134207281](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401134207281.png)

5.配置 consumer 服务,在启动类上增加`@EnableDiscoveryClient`注解,添加 bootstrap.yml 作为配置文件,并添加接口利用 RestTemplate 调用 provider 提供的方法

启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class CocoConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(CocoConsumerApplication.class, args);
    }
}
```

restTemplate 配置类

```java
@Configuration
public class RestConfig {
    @LoadBalanced
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

消费者接口类

```java
@RestController
public class ConsumerController {
    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/echo")
    public String echo(String str) {
        return restTemplate.getForObject("http://coco-provider/echo/?string=" + str, String.class);
    }
}
```

配置文件

```yml
spring:
  application:
    name: coco-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.70.1:8848 #注册中心地址
server:
  port: 9991
```

6.启动 consumer 服务,可以在 Nacos 服务列表看见两个服务
![image-20210401134230450](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401134230450.png)

7.调用 consumer 接口,可以实现通过 consumer 接口调用 provider 提供的方法
![image-20210401134245658](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401134245658.png)

8.本文相关代码提交到 gitee: <https://gitee.com/AtlsHY/coco>
