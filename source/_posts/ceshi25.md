---
layout: ceshi25
title: 实时用户画像系统(二十一)加入后端接口让图表动态显示数据1后端服务接口构建
date: 2020-03-30 20:46:15
tags: 机器学习
comments: true
---

### 后端服务接口构建

#### 使用idea构建maven项目模块

![maven项目](1585626060548.png)

![模块名字](1585626107081.png)

点击下一步,然后Finish

组织一下子youfanSearchInfo模块的web.xml,并加入依赖,如下:

<!--more-->

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
        <groupId>com.youfan.test</groupId>
        <artifactId>youfanSearchInfo</artifactId>
        <version>1.0-SNAPSHOT</version>

        <parent>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>2.0.2.RELEASE</version>
                <relativePath/>
        </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <dependency>
            <groupId>com.youfan.test</groupId>
            <artifactId>youfancommon_test</artifactId>
            <version>1.0</version>
        </dependency>
        <dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongodb-driver</artifactId>
            <version>3.6.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <version>1.2.0</version>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
    
        
</project>
```

#### 新建package

com.youfan.search

#### 添加application.properties到resources文件夹

```reStructuredText
server.port=8763
spring.application.name=youfanSearchInfo
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
```

#### 新建control层

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;

/**
 * Created by li on 2019/1/6.
 */
@RestController
@RequestMapping("test")
public class Test {

    @RequestMapping(value = "helloworld",method = RequestMethod.GET)
    public String hellowolrd(HttpServletRequest req){
        String ip =req.getRemoteAddr();
        String result = "hello world from "+ ip;
        return result;
    }

}
```

#### 添加启动类

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

/**
 * Created by li on 2019/1/6.
 */
@SpringBootApplication
@EnableEurekaClient
public class Startupmain {
    public static void main(String[] args) {

        SpringApplication.run( Startupmain.class, args );
    }
}
```

#### 添加mongo base类

```java
import com.mongodb.MongoClient;
import com.mongodb.ServerAddress;
import com.youfan.utils.ReadProperties;

import java.util.ArrayList;
import java.util.List;

public class BaseMongo {
    protected static MongoClient mongoClient ;
      
      static {
         List<ServerAddress> addresses = new ArrayList<ServerAddress>();
         String[] addressList = ReadProperties.getKey("mongoaddr","youfansearch.properties").split(",");
         String[] portList = ReadProperties.getKey("mongoport","youfansearch.properties").split(",");
         for (int i = 0; i < addressList.length; i++) {
            ServerAddress address = new ServerAddress(addressList[i], Integer.parseInt(portList[i]));
            addresses.add(address);
            
         }
         mongoClient = new MongoClient(addresses);
      }
   
}
```

#### 添加mongo配置文件youfansearch.properties到resources

```reStructuredText
mongoaddr=192.168.37.141
mongoport=27017
```

#### 添加mongo接口类

```java
import com.youfan.entity.AnalyResult;
import com.youfan.search.service.MongoDataServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by li on 2019/1/19.
 */

/**
 * 年代：yearbasestatics
 终端偏好：usetypestatics
 邮件运营商：emailstatics
 消费水平：consumptionlevelstatics
 潮男潮女：chaoManAndWomenstatics
 手机运营商：carrierstatics
 品牌偏好：brandlikestatics
 */
@RestController
@RequestMapping("yearBase")
public class MongodataControl {

    @Autowired
    private MongoDataServiceImpl mongoDataServiceImpl;

    @RequestMapping(value = "searchYearBase",method = RequestMethod.POST)
    public List<AnalyResult> searchYearBase(){
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();
        //40年代，50年代，60年代，70年代，80年代，90年代，00年代 10后
        analyResult.setCount(50l);
        analyResult.setInfo("40年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(60l);
        analyResult.setInfo("50年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(100l);
        analyResult.setInfo("60年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(90l);
        analyResult.setInfo("70年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(500l);
        analyResult.setInfo("80年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(600l);
        analyResult.setInfo("90年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(300l);
        analyResult.setInfo("00年代");
        list.add(analyResult);
        analyResult = new AnalyResult();
        analyResult.setCount(70l);
        analyResult.setInfo("10后");

        list.add(analyResult);

        return list;
//        return mongoDataServiceImpl.listMongoInfoby("yearbasestatics");
    }

    @RequestMapping(value = "searchUseType",method = RequestMethod.POST)
    public List<AnalyResult> searchUseType(){
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();
        //pc端，小程序端，移动端
        analyResult.setCount(50l);
        analyResult.setInfo("pc端");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(60l);
        analyResult.setInfo("小程序端");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(40l);
        analyResult.setInfo("移动端");
        list.add(analyResult);

        return list;
//        return mongoDataServiceImpl.listMongoInfoby("usetypestatics");
    }

    @RequestMapping(value = "searchEmail",method = RequestMethod.POST)
    public List<AnalyResult> searchEmail(){
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();
        //qq邮箱，139邮箱，网易邮箱,阿里邮箱
        analyResult.setCount(150l);
        analyResult.setInfo("qq邮箱");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(60l);
        analyResult.setInfo("139邮箱");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(240l);
        analyResult.setInfo("网易邮箱");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(540l);
        analyResult.setInfo("阿里邮箱");
        list.add(analyResult);

        return list;

//        return mongoDataServiceImpl.listMongoInfoby("emailstatics");
    }

    @RequestMapping(value = "searchConsumptionlevel",method = RequestMethod.POST)
    public List<AnalyResult> searchConsumptionlevel(){
        //高消费 中等消费  低消费
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();
        //qq邮箱，139邮箱，网易邮箱,阿里邮箱
        analyResult.setCount(50l);
        analyResult.setInfo("高消费");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(560l);
        analyResult.setInfo("中等消费");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(760l);
        analyResult.setInfo("低消费");
        list.add(analyResult);

        return list;

//        return mongoDataServiceImpl.listMongoInfoby("consumptionlevelstatics");
    }

    @RequestMapping(value = "searchChaoManAndWomen",method = RequestMethod.POST)
    public List<AnalyResult> searchChaoManAndWomen(){
        //潮男 潮女
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();

        analyResult.setCount(350l);
        analyResult.setInfo("潮男");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(560l);
        analyResult.setInfo("潮女");
        list.add(analyResult);

        return list;

//        return mongoDataServiceImpl.listMongoInfoby("chaoManAndWomenstatics");
    }

    @RequestMapping(value = "searchCarrier",method = RequestMethod.POST)
    public List<AnalyResult> searchCarrier(){
        //联通 移动 电信 其他
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();

        analyResult.setCount(1350l);
        analyResult.setInfo("联通");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(1560l);
        analyResult.setInfo("移动");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(560l);
        analyResult.setInfo("电信");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(4560l);
        analyResult.setInfo("其他");
        list.add(analyResult);

        return list;
//        return mongoDataServiceImpl.listMongoInfoby("carrierstatics");
    }

    @RequestMapping(value = "searchBrandlike",method = RequestMethod.POST)
    public List<AnalyResult> searchBrandlike(){
        //李宁 爱迪达斯 森马 海尔
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        AnalyResult analyResult = new AnalyResult();

        analyResult.setCount(1350l);
        analyResult.setInfo("李宁");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(1560l);
        analyResult.setInfo("爱迪达斯");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(560l);
        analyResult.setInfo("森马");
        list.add(analyResult);

        analyResult = new AnalyResult();
        analyResult.setCount(4560l);
        analyResult.setInfo("海尔");
        list.add(analyResult);

        return list;

//        return mongoDataServiceImpl.listMongoInfoby("brandlikestatics");
    }

}
```

添加service层类

```java
import com.alibaba.fastjson.JSONObject;
import com.mongodb.client.AggregateIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import com.youfan.entity.AnalyResult;
import com.youfan.search.base.BaseMongo;
import org.bson.Document;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * Created by li on 2019/1/19.
 */
@Service
public class MongoDataServiceImpl extends BaseMongo{


    public List<AnalyResult> listMongoInfoby(String tablename) {
        List<AnalyResult> result = new ArrayList<AnalyResult>();


        MongoDatabase db = mongoClient.getDatabase("youfanPortrait");
        MongoCollection<Document> collection =  db.getCollection(tablename);


        Document groupFields = new Document();
        Document idFields = new Document();
        idFields.put("info", "$info");
        groupFields.put("_id", idFields);
        groupFields.put("count", new Document("$sum", "$count"));

        Document group = new Document("$group", groupFields);


        Document projectFields = new Document();
        projectFields.put("_id", false);
        projectFields.put("info", "$_id.info");
        projectFields.put("count", true);
        Document project = new Document("$project", projectFields);
        AggregateIterable<Document> iterater = collection.aggregate(
                (List<Document>) Arrays.asList(group, project)
        );

        MongoCursor<Document> cursor = iterater.iterator();
        while(cursor.hasNext()){
            Document document = cursor.next();
            String jsonString = JSONObject.toJSONString(document);
            AnalyResult brandUser = JSONObject.parseObject(jsonString,AnalyResult.class);
            result.add(brandUser);
        }
        return result;
    }
}
```

#### 注册到注册中心

启动注册中心，并访问

<http://localhost:8761/>

![注册中心界面](1585628714405.png)

启动mongodb

![启动mongodb](1585629226018.png)

启动客户端

![客户端已经注册到注册中心](1585629833233.png)

测试接口

<http://localhost:8763/test/helloworld>

![测试接口](1585629983004.png)

