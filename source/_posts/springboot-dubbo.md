---
title: 搭建SpringBoot+dubbo+zookeeper简单demo工程
tags: [springboot,dubbo]
abbrlink: ce309dda
date: 2019-09-07 21:23:56
---

### 1.创建一个基于 maven 的 SpringBoot 工程，type 选择 maven pom

![](https://i.loli.net/2021/04/01/zs5V1yP2Bu3fWNX.png)

![](https://i.loli.net/2021/04/01/1rGqw4MkodtyZRF.png)

![](https://i.loli.net/2021/04/01/eZMuIxrgkGLWT9N.png)

![](https://i.loli.net/2021/04/01/AwG4ina6TJkFsDQ.png)

### 2.创建完成之后的工程结构如下，生成的 pom.xml 内容如下

![](https://i.loli.net/2021/04/01/iMjXfLvqCYV2Any.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.aias</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>
</project>
```

### 3.在父类项目上右键->New->Module，添加三个子 module，demoProvider（服务提供者）\demoConsumer（服务调用者）\common（公共模块），过程和创建父类工程类似

![](https://i.loli.net/2021/04/01/TUcaNkHXCKVQR6P.png)

![](https://i.loli.net/2021/04/01/hEo6K78jcTluaMt.png)

![](https://i.loli.net/2021/04/01/qPKyXiRLgaFjG5S.png)

### 4.修改子模块的 pom.xml，将<parent>里的内容替换成我们父工程的内容

### 5.common 工程里创建一个 User.java 和 IUserService，common 工程之后是打包成被其他 Module 项目依赖的 jar 包，不需要独立部署，所以这里把其他的无用文件夹删除，完成之后的目录结构如下。

![](https://i.loli.net/2021/04/01/RWIwmfKGxvpDgUo.png)

### common 项目的 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.aias</groupId>
        <artifactId>demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <groupId>com.aias</groupId>
    <artifactId>common</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>common</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
</project>

```

### 6.demoProvider 工程中添加了 MyBatis、druid 和 mysql-connector 的依赖，实现了 IUserService 接口类，实现了对数据库的实际操作，目录结构如下。在不添加其他配置和依赖的情况下，可以通过单元测试

![](https://i.loli.net/2021/04/01/qzUnk4xPr5tXI1O.png)

### 7.添加 dubbo 和 dubbo-spring-boot-starter 依赖，在 application.properties 中添加 dubbo 相关配置信息，然后启动单元测试，这时候会报连接失败的错误，需要启动 zookeeper 才能成功。至此 provider 已经配置完成

![](https://i.loli.net/2021/04/01/hHBmAnWISjpyktb.png)

![](https://i.loli.net/2021/04/01/w4nzBJ32VHsQAti.png)

### demoProvider 工程的完整 pom 文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.aias</groupId>
        <artifactId>demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <groupId>com.aias</groupId>
    <artifactId>demoProvider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demoProvider</name>
    <description>Demo project for Spring Boot</description>
    <dependencies>
        <dependency>
            <groupId>com.aias</groupId>
            <artifactId>common</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        <!-- mybatis -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>
        <!-- druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
        <!-- mysql-connector -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!-- dubbo -->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.3</version>
        </dependency>
    </dependencies>
</project>

```

### 8.在 demoConsumer 工程中添加相关依赖，和 dubbo 配置信息，通过注解注入的方式调用服务提供者的 service 实现

![](https://i.loli.net/2021/04/01/xmX5YfPC3KweNbp.png)

![](https://i.loli.net/2021/04/01/s4fNegUxRMiZlTV.png)

### demoConsumer 完整 pom 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.aias</groupId>
        <artifactId>demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <groupId>com.aias</groupId>
    <artifactId>demoConsumer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demoConsumer</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>com.aias</groupId>
            <artifactId>common</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        <!-- dubbo -->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.3</version>
        </dependency>
    </dependencies>
</project>

```

### 9.启动 zookeeper，启动 demoProvider 和 demoConsumer 项目，浏览器访问http://localhost:8080/hello

### 成功页面

![](https://i.loli.net/2021/04/01/93rmxG2EsKkWueq.png)

### 10.项目源码详见[springboot 集成 dubbo 示例项目](https://github.com/Aias00/demoSpringBootDubbo)
