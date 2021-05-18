---
title: 学习接入sentinel
tags: [sentinel]
abbrlink: 32240ede
date: 2020-12-15 19:54:09
---

学习使用 sentinel 流控功能

### 配置 sentinel 控制台

1.可以从 github 获取源码到本地编译或者直接下载官方提供的 jar 包

github 地址:<https://github.com/alibaba/Sentinel.git>

操作流程

```shell
git clone https://github.com/alibaba/Sentinel.git
cd Sentinel
mvn clean package
```

jar 包下载地址: <https://github.com/alibaba/Sentinel/releases>

2.启动控制台

`java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar`

其中 8080 是指定控制台程序的启动端口,如果端口被占用可以修改成别的。

启动成功之后打开 localhost:8080 可以看到 sentinel 控制台的登录界面

![](https://cdn.jsdelivr.net/gh/Aias00/picgo/images/1.png)

默认的用户名密码都是 sentinel

登录成功之后页面
![](https://cdn.jsdelivr.net/gh/Aias00/picgo/images/2.png)

在这个页面展示了所有连接到 sentinel 的服务相关信息

### 配置项目连接到 sentinel

1.建一个项目,配置文件指向 sentinel
application.yml

```yml
spring:
  application:
    name: coco-sentinel
  profiles:
    active: dev
  cloud:
    sentinel:
      transport:
        port: 8719
        dashboard: localhost:8080
server:
  port: 9993
```

pom.xml

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
        <!-- spring-cloud-alibaba-sentinel -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
    </dependencies>
```

测试类

```java
@RestController
public class SentinelController {
    @Resource
    private TestService service;

    @GetMapping(value = "/hello/{name}")
    public String apiHello(@PathVariable String name) {
        return service.sayHello(name);
    }
}
@Service
public class TestService {
    @SentinelResource(value = "sayHello")
    public String sayHello(String name) {
        return "Hello, " + name;
    }
}
```

@SentinelResource 注解用来标识资源是否被限流、降级。上述例子上该注解的属性 sayHello 表示资源名。
@SentinelResource 还提供了其它额外的属性如 blockHandler，blockHandlerClass，fallback 用于表示限流或降级的操作（注意有方法签名要求），更多内容可以参考 Sentinel 注解支持文档<https://github.com/alibaba/Sentinel/wiki/%E6%B3%A8%E8%A7%A3%E6%94%AF%E6%8C%81>。若不配置 blockHandler、fallback 等函数，则被流控降级时方法会直接抛出对应的 BlockException；若方法未定义 throws BlockException 则会被 JVM 包装一层 UndeclaredThrowableException

2.启动服务,请求一次,然后查看 sentinel 控制台

![](https://cdn.jsdelivr.net/gh/Aias00/picgo/images/3.png)

可以看到我们的服务已经在控制台展示出来了

### 流控配置

1.点击我们要设置流控的服务,点击簇点链路,找到刚才指定的资源,即 sayHello

![](https://cdn.jsdelivr.net/gh/Aias00/picgo/images/4.png)

2.点击流控,在弹出的对话框里设置单机阈值为 1

![](https://cdn.jsdelivr.net/gh/Aias00/picgo/images/5.png)

3.点击新增,保存成功之后可以看到刚才配置的流控规则信息

![](https://cdn.jsdelivr.net/gh/Aias00/picgo/images/6.png)

4.这时再次请求我们的接口,可以发现在 1 秒内只有一次请求能成功

![](https://cdn.jsdelivr.net/gh/Aias00/picgo/images/7.png)

而我们应用的控制台打印出了 UndeclaredThrowableException

![](https://cdn.jsdelivr.net/gh/Aias00/picgo/images/8.png)

本文代码地址:<https://gitee.com/AtlsHY/coco/tree/master/coco-sentinel>
更多详细内容请参考官方文档:<https://github.com/alibaba/spring-cloud-alibaba/wiki/Sentinel>
