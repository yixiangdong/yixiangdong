---
layout: clickhouse2
title: Flink+ClickHouse电商实时数据分析系统之构建spring cloud注册中心
date: 2020-04-05 21:01:12
tags: ClickHouse
comments: true
---

### 新建注册中心module

module名字: jiangziRegisterCenter

#### 整理依赖

```xml
    <modelVersion>4.0.0</modelVersion>
    <parent>    
    <groupId>com.jiangzi</groupId>
    <artifactId>FlinkAnanlySystem</artifactId>
    <version>1.0-SNAPSHOT</version>
	</parent>
   <artifactId>jiangziRegisterCenter</artifactId>
```

<!--more-->

#### 添加全局依赖到父项目的pom.xml文件中

```xml
 <!--全局依赖 添加在-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
```

#### 添加注册中心的依赖到pom.xml

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
```

同时添加依赖到数据**收集模块**jiangziDataCollect用来client注册到注册中心

```
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
       </dependency>
```



#### 添加配置文件和配置

文件名:

application.yml

```yaml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

spring:
  application:
    name: jiangziRegisterCenter
```

同时在数据收集模块jiangziDataCollect配置文件添加配置

```xml
server.port=8081
spring.application.name=jiangziDataCollect
<!--下面是新添加注册到注册中心-->
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
```

#### 为启动类添加注解

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
```

### 启动注册中心和数据收集模块

查看服务和注册的client

<http://localhost:8761/>

![注册中心](1586111699690.png)

错误分析:

如果以上看不到Application，那么请检查pom结构是否正确或者其他地方的依赖是否正确

父项目的pom结构是这样的：

```xml
   <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.2.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>com.jiangzi</groupId>
    <artifactId>FlinkAnanlySystem</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
```

子模块的pom顶部分别是这样的：

数据收集部分：

```xml
<modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>FlinkAnanlySystem</artifactId>
        <groupId>com.jiangzi</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <artifactId>jiangziDataCollect</artifactId>
```

注册中心部分:

```xml
<modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>FlinkAnanlySystem</artifactId>
        <groupId>com.jiangzi</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <artifactId>jiangziRegisterCenter</artifactId>
```

