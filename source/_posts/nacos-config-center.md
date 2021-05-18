---
title: 使用nacos配置中心功能
tags:
  - nacos
  - spring-cloud-alibaba
abbrlink: 24f85607
date: 2021-05-18 12:48:36
---

开始学习使用 spring-cloud-alibaba

### 使用 nacos 配置中心功能

1.创建一个 spring-cloud-alibaba 模块，添加以下依赖

```xml
<!--nacos配置中心-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2.创建测试 controller

```java
@RestController
@RefreshScope
public class ConfigController {
    @Value("${useLocalCache:false}")
    private boolean useLocalCache;

    @GetMapping("/get")
    public boolean get() {
        return useLocalCache;
    }
}
```

3.添加配置文件

bootstarp.yml

```yml
spring:
  application:
    name: coco-web
  profiles:
    active: dev
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.70.1:8848 #注册中心地址
      config:
        server-addr: 192.168.70.1:8848 # 配置中心地址
        file-extension: yaml # 指定yaml格式的配置
server:
  port: 9992
```

4.在 nacos 配置管理->配置列表中点击右侧+号新增配置

![](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401133424970.png)

并点击发布

5.启动服务，访问接口

![](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401133441929.png)

6.查看配置列表，可以看见刚才发布的配置数据

![](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401133458659.png)

7.点击编辑，修改对应配置项的数值，点击发布

![](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401133516106.png)

8.再次访问接口，可以发现数值跟随 nacos 中的配置进行变更

![](https://gitee.com/AtlsHY/picgo/raw/master/images/image-20210401133530606.png)

本文代码：<https://gitee.com/AtlsHY/coco/tree/master/coco-web>
