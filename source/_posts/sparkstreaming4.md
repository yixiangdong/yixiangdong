---
layout: sparkstreaming4
title: SparkStreaming实时计算系统(四)之[精讲]dubbo远程调用接口设计以及前端展示服务
date: 2020-04-02 02:32:18
tags: sparkstreaming
comments: true
---

### Dubbo讲解

## SOA

面向服务编程，是一种思想，一种方法论，一种分布式的服务架构SOA是Service-Oriented Architecture的首字母简称，它是一种支持面向服务的架构样式。从服务、基于服务开发和服务的结果来看，面向服务是一种思考方式。其实SOA架构更多应用于互联网项目开发。随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，迫切需一个治理系统确保架构有条不紊的演进。

## **面向服务**

技术为业务而生，架构也为业务而出现。随着业务的发展、用户量的增长，系统数量增多，调用依赖关系也变得复杂，为了确保系统高可用、高并发的要求，系统的架构也从单体时代慢慢迁移至服务SOA时代，根据不同服务对系统资源的要求不同，我们可以更合理的配置系统资源，使系统资源利用率最大化。

## Dubbo**简介**

Dubbo是一个分布式服务框架，是阿里巴巴的开源项目，被国内电商及互联网项目中所使用。Dubbo 致力于提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理方案。简单的说，dubbo就是个服务框架，如果没有分布式的需求，其实是不需要用的，只有在分布式的时候，才有dubbo这样的分布式服务框架的需求，并且本质上是个服务调用的东西，说白了就是个远程服务调用的分布式框架。

<!--more-->

## Dubbo**架构**

![1585974268653](1585974268653.png)

## **节点角色说明**

Provider	暴露服务的服务提供方

Consumer 调用远程服务的服务消费方

Registry	服务注册与发现的注册中心

Monitor	统计服务的调用次数和调用时间的监控中心(Monitor挂掉不会影响到Consumer和Provider之间的调用)

Container服务运行容器

### Dubbo的常用方式有两种

注解方式（常用）	

XML方式（常用）	

属性配置方式

API配置方式

### Dubbo的注册中心

Multicast 注册中心l	

Zookeeper注册中心（官方推荐)

Redis注册中心

Simple注册中心

### Duboo开发

maven依赖

```xml
<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>dubbo</artifactId>
			<version>2.5.7</version>			
		</dependency>
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.4.6</version>
		</dependency>
		<dependency>
			<groupId>com.github.sgroschupf</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.1</version>
		</dependency>
		
		<dependency>
			<groupId>javassist</groupId>
			<artifactId>javassist</artifactId>
			<version>3.11.0.GA</version>
		</dependency>
```

先看一个测试的例子:

#### 服务端

```java
package com.test;

public interface HelloT {
	public String sayhello(String name);
}
```

```java
import com.alibaba.dubbo.config.annotation.Service;

@Service
public class HelloImpl implements HelloT{

	public String sayhello(String name) {
		return "haha=="+name;
	}
}
```

#### 服务提供端dubbo配置文件

作用:  用于添加需要暴露的服务接口和接口实现

在resources文件夹添加application-context-provide.xml即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <dubbo:protocol name="dubbo" port="20880"></dubbo:protocol> 
    
	<dubbo:application name="dubbo-service-test"/>  
	<dubbo:registry address="zookeeper://localhost:2181" />
	<dubbo:service interface="com.test.HelloT" ref="helloImpl"/>
    <bean id="helloImpl" class="com.test.HelloImpl"/>
</beans>
```

### 服务消费端dubbo配置

在resources文件夹添加application-context-consumer.xml即可

作用：用于添加需要访问的接口

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-consumer"/>
    <dubbo:registry address="zookeeper://localhost:2181" />
    <dubbo:reference id="helloT" interface="com.test.HelloT"/>
</beans>
```

启动zookeeper服务后分别启动服务提供者,服务消费者.

### 测试服务提供者

```java
package com;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Provide {
	    public static void main(String[] args) throws Exception {
	        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(
	                new String[] {"application-context-dubbo.xml"});
	        context.start();
	        // press any key to exit
	        System.in.read();
	    }
}
```

### 测试消费服务

```java
 package com;

import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.test.HelloT;

public class consumer {
	    public static void main(String[] args) throws Exception {
	        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(
	                new String[]{"application-context-comsumter.xml"});
	        context.start();
	        // obtain proxy object for remote invocation
	        HelloT demoService = (HelloT) context.getBean("helloT");
	        // execute remote invocation
	        String hello = demoService.sayhello("laogao");
	        // show the result
	        System.out.println(hello);
	    }
}
```

以上的例子没问题的情况下，可以开始进行业务代码编写

## 搭建前端展示服务

### 搭建SearchService dubbo项目

项目作用：在错误日志存储到mongodb后，需要进一步对mongodb的数据进行增删改查从而对前端进行展现

#### 项目架构

![项目架构](1585986345264.png)

#### 添加pom依赖

```xml
<modelVersion>4.0.0</modelVersion>
  <groupId>com.youfan</groupId>
  <artifactId>SearchSrevice</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  
  <dependencies>
	  	<dependency> 			
	  	        <groupId>com.alibaba</groupId> 	
	  			<artifactId>dubbo</artifactId> 			
	  			<version>2.5.7</version>	
	  </dependency> 
	  <dependency> 			
	 		<groupId>org.apache.zookeeper</groupId> 
	 					<artifactId>zookeeper</artifactId> 		
	 						<version>3.4.6</version> 		
	 						<exclusions>
	 							<exclusion>
	 								<artifactId>jmxri</artifactId>
	 								<groupId>com.sun.jmx</groupId>
	 							</exclusion>
	 							<exclusion>
	 								<artifactId>jmxtools</artifactId>
	 								<groupId>com.sun.jdmk</groupId>
	 							</exclusion>
	 						</exclusions>
  	</dependency>
<dependency> 
    <groupId>com.github.sgroschupf</groupId>
	<artifactId>zkclient</artifactId> 			
	<version>0.1</version> 		
	</dependency> 		
<dependency> 			
<groupId>javassist</groupId> 	
		<artifactId>javassist</artifactId> 		
			<version>3.11.0.GA</version> 	
				</dependency>
				
				<dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver</artifactId>
        <version>3.0.1</version>
    </dependency>
    
     <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>1.7.1</version>
	</dependency>
	
	   <dependency>
      <groupId>com.typesafe</groupId>
      <artifactId>config</artifactId>
      <version>1.2.1</version>
    </dependency>
       <dependency>
            <groupId>jdk.tools</groupId>
            <artifactId>jdk.tools</artifactId>
            <version>1.8</version>
            <scope>system</scope>
            <systemPath>C:/Program Files/Java/jdk1.8.0_211/lib/tools.jar</systemPath>
        </dependency>
        
        <dependency>
  	<groupId>com.lidahai</groupId>
  	<artifactId>AppInfoCommon</artifactId>
  	<version>0.0.1-SNAPSHOT</version>
  </dependency>
  </dependencies>
```



##### 添加mongo配置文件application.properties

```text
mongodbAddr=192.168.38.139
mongodbPort=27017
```

#### 添加错误日志分析接口

建立Dao层com.youfan.dao

```java
package com.youfan.dao;

import java.util.List;

import com.lidahai.analyentity.AppEorranaly;

public interface ErrorAnalyDao {
	public List<AppEorranaly> listErrorInfoBy(
			String appId,
			String timeFrom,
			String timeTo, 
			String timeUnit, 
			String appVersion,
			String appChannel, 
			String appPlatform, 
			String userGroup,
			String errorId,
			List<String> groupByConditons
			);
}
```

 在common模块添加实体类

```java
package com.lidahai.analyentity;

import java.io.Serializable;

public class AppEorranaly implements Serializable{
	private String timeValue;//�����洢��ʱ��,�������죬Сʱ���£���
	private String appId;//Ӧ��id			
	private String appVersion;//�汾
	private String appChannel;//����
	
	private String appPlatform;//ƽ̨
	private String deviceStyle;//�豸����			
	private String osType;//����ϵͳ				
	
	private Long errorCnt;//����ķ�������			
	private String errorId;//����id
	public String getTimeValue() {
		return timeValue;
	}
	public void setTimeValue(String timeValue) {
		this.timeValue = timeValue;
	}
	public String getAppId() {
		return appId;
	}
	public void setAppId(String appId) {
		this.appId = appId;
	}
	public String getAppVersion() {
		return appVersion;
	}
	public void setAppVersion(String appVersion) {
		this.appVersion = appVersion;
	}
	public String getAppChannel() {
		return appChannel;
	}
	public void setAppChannel(String appChannel) {
		this.appChannel = appChannel;
	}
	public String getAppPlatform() {
		return appPlatform;
	}
	public void setAppPlatform(String appPlatform) {
		this.appPlatform = appPlatform;
	}
	public String getDeviceStyle() {
		return deviceStyle;
	}
	public void setDeviceStyle(String deviceStyle) {
		this.deviceStyle = deviceStyle;
	}
	public String getOsType() {
		return osType;
	}
	public void setOsType(String osType) {
		this.osType = osType;
	}
	public Long getErrorCnt() {
		return errorCnt;
	}
	public void setErrorCnt(Long errorCnt) {
		this.errorCnt = errorCnt;
	}
	public String getErrorId() {
		return errorId;
	}
	public void setErrorId(String errorId) {
		this.errorId = errorId;
	}
}
```

添加实现接口实现类

包名:  com.youfan.dao.impl

```java
package com.youfan.dao.impl;


import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Date;
import java.util.List;
import java.util.Map;

import org.bson.Document;
import org.springframework.stereotype.Repository;

import com.google.gson.Gson;
import com.lidahai.Utils.DateTimeTools;
import com.lidahai.analyentity.AppEorranaly;
import com.mongodb.client.AggregateIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.model.Sorts;
import com.youfan.base.BaseMongo;
import com.youfan.dao.ErrorAnalyDao;


@Repository
public class ErrorAnalyDaoImpl extends BaseMongo implements ErrorAnalyDao {

	public List<AppEorranaly> listErrorInfoBy(String appId, String timeFrom, String timeTo, String timeUnit,
			String appVersion, String appChannel, String appPlatform, String userGroup, String errorId,
			List<String> groupByConditons) {
		List<AppEorranaly> result = new ArrayList<AppEorranaly>();

		if (appId == null || appId.trim().isEmpty()) {
			return result;
		}
		
		MongoDatabase db = mongoClient.getDatabase(appId);
		MongoCollection<Document> collection =  null;
		if(timeUnit == null || timeUnit.trim().isEmpty()){
			timeUnit = "daily";
		}
		if("daily".equals(timeUnit)){
			collection = db.getCollection("ErrorInfoDaily");
		}else{
			return result;
		}
		
		Long timeNow = System.currentTimeMillis();
	   
		if(timeFrom==null || timeFrom.trim().isEmpty()) {
			timeFrom = DateTimeTools.getDateStrBy(timeNow - (long)7*24*3600*1000);
		}
		if(timeTo==null || timeTo.trim().isEmpty()) {
			timeTo = DateTimeTools.getDateStrBy(timeNow);
		}
		
		Document matchFields = new Document();
		matchFields.put("timeValue", 
				new Document().append("$gte", timeFrom)
				.append("$lte", timeTo));
        if (appVersion != null && !appVersion.trim().isEmpty()) {
        	matchFields.put("appVersion", new Document().append("$eq", appVersion));
        }
        if (appChannel != null && !appChannel.trim().isEmpty()) {
        	matchFields.put("appChannel", new Document().append("$eq", appChannel));
        }
        
        if (appPlatform != null && !appPlatform.trim().isEmpty()) {
        	matchFields.put("appPlatform", new Document().append("$eq", appPlatform));
        }
        if (errorId != null && !errorId.trim().isEmpty()) {
        	matchFields.put("errorId", new Document().append("$eq", errorId));
        }
        Document match = new Document("$match", matchFields);
        
        
        Document sort = new Document("$sort", Sorts.ascending("timeValue"));
		
		
        Document groupFields = new Document();
        Document idFields = new Document();
        idFields.put("errorId", "$errorId");
        idFields.put("timeValue", "$timeValue");
        idFields.put("appId", "$appId");
        if (appVersion != null && !appVersion.trim().isEmpty()) {
        	idFields.put("appVersion", "$appVersion");
        }
        if (appChannel != null && !appChannel.trim().isEmpty()) {
        	idFields.put("appChannel", "$appChannel");
        }
        
        if (appPlatform != null && !appPlatform.trim().isEmpty()) {
        	idFields.put("appPlatform", "$appPlatform");
        }
        
        if(groupByConditons != null && groupByConditons.size() >0){
	        for(int i=0;i<groupByConditons.size();i++){
	        	String groupByConditon = groupByConditons.get(i);
	        	if ("appVersion".equals(groupByConditon)) {
	            	idFields.put("appVersion", "$appVersion");
	            }
	            if ("appChannel".equals(groupByConditon)) {
	            	idFields.put("appChannel", "$appChannel");
	            }
	            
	            if ("appPlatform".equals(groupByConditon)) {
	            	idFields.put("appPlatform", "$appPlatform");
	            }
	        }
        }
        
        
        idFields.put("deviceStyle", "$deviceStyle");
        idFields.put("osType", "$osType");
        groupFields.put("_id", idFields);
        
        
        
        groupFields.put("errorCnt", new Document("$sum", "$errorCnt"));
		Document group = new Document("$group", groupFields);
		
		
        Document projectFields = new Document();
        projectFields.put("_id", false);
        projectFields.put("errorId", "$_id.errorId");
        projectFields.put("timeValue", "$_id.timeValue");
        projectFields.put("appId", "$_id.appId");
        if (appVersion != null && !appVersion.trim().isEmpty()) {
        	projectFields.put("appVersion", "$_id.appVersion");
        }
        if (appChannel != null && !appChannel.trim().isEmpty()) {
        	projectFields.put("appChannel", "$_id.appChannel");
        }
        
        if (appPlatform != null && !appPlatform.trim().isEmpty()) {
        	projectFields.put("appPlatform", "$_id.appPlatform");
        }
        
        
        if(groupByConditons != null && groupByConditons.size() >0){
	        for(int i=0;i<groupByConditons.size();i++){
	        	String groupByConditon = groupByConditons.get(i);
	        	if ("appVersion".equals(groupByConditon)) {
	        		projectFields.put("appVersion", "$_id.appVersion");
	            }
	            if ("appChannel".equals(groupByConditon)) {
	            	projectFields.put("appChannel", "$_id.appChannel");
	            }
	            
	            if ("appPlatform".equals(groupByConditon)) {
	            	projectFields.put("appPlatform", "$_id.appPlatform");
	            }
	        }
        }
        
        projectFields.put("osType", "$_id.osType");
        projectFields.put("deviceStyle", "$_id.deviceStyle");
        projectFields.put("errorCnt", true);
		Document project = new Document("$project", projectFields);
		AggregateIterable<Document> iterater = collection.aggregate(
				(List<Document>) Arrays.asList(match,  sort, group, project)
				);
		
		MongoCursor<Document>  cursor = iterater.iterator();
		Gson gson = new Gson();
		while(cursor.hasNext()){
			Document document = cursor.next();
			String jsonString = gson.toJson(document);
			AppEorranaly errorInfo = gson.fromJson(jsonString, AppEorranaly.class);
			result.add(errorInfo);
		}
		return result;
	}
	
	public static void main(String[] args) {
		List<AppEorranaly> list = new ErrorAnalyDaoImpl().listErrorInfoBy("youmeng568", "180203", "180508", null, null, null, null, null, null, null);
		System.out.println(list);
	}
}
```

#### 添加mongo类

类名: BaseMongo

```java
package com.youfan.base;

import java.util.ArrayList;
import java.util.List;

import com.lidahai.Utils.PropertyRead;
import com.mongodb.MongoClient;
import com.mongodb.ServerAddress;

public class BaseMongo {
	protected static MongoClient mongoClient ;
		
		static {
			List<ServerAddress> addresses = new ArrayList<ServerAddress>();
			String[] addressList = PropertyRead.getkey("mongodbAddr").split(",");
			String[] portList = PropertyRead.getkey("mongodbPort").split(",");
			for (int i = 0; i < addressList.length; i++) {
				ServerAddress address = new ServerAddress(addressList[i], Integer.parseInt(portList[i]));
				addresses.add(address);
				
			}
			mongoClient = new MongoClient(addresses);
		}
	
}

```

添加mongo接口实现类

```java
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Date;
import java.util.List;
import java.util.Map;

import org.bson.Document;
import org.springframework.stereotype.Repository;

import com.google.gson.Gson;
import com.lidahai.Utils.DateTimeTools;
import com.lidahai.analyentity.AppEorranaly;
import com.mongodb.client.AggregateIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.model.Sorts;
import com.youfan.base.BaseMongo;
import com.youfan.dao.ErrorAnalyDao;


@Repository
public class ErrorAnalyDaoImpl extends BaseMongo implements ErrorAnalyDao {

	public List<AppEorranaly> listErrorInfoBy(String appId, String timeFrom, String timeTo, String timeUnit,
			String appVersion, String appChannel, String appPlatform, String userGroup, String errorId,
			List<String> groupByConditons) {
		List<AppEorranaly> result = new ArrayList<AppEorranaly>();

		if (appId == null || appId.trim().isEmpty()) {
			return result;
		}
		
		MongoDatabase db = mongoClient.getDatabase(appId);
		MongoCollection<Document> collection =  null;
		if(timeUnit == null || timeUnit.trim().isEmpty()){
			timeUnit = "daily";
		}
		if("daily".equals(timeUnit)){
			collection = db.getCollection("ErrorInfoDaily");
		}else{
			return result;
		}
		
		Long timeNow = System.currentTimeMillis();
	   
		if(timeFrom==null || timeFrom.trim().isEmpty()) {
			timeFrom = DateTimeTools.getDateStrBy(timeNow - (long)7*24*3600*1000);
		}
		if(timeTo==null || timeTo.trim().isEmpty()) {
			timeTo = DateTimeTools.getDateStrBy(timeNow);
		}
		
		Document matchFields = new Document();
		matchFields.put("timeValue", new Document().append("$gte", timeFrom).append("$lte", timeTo));
        if (appVersion != null && !appVersion.trim().isEmpty()) {
        	matchFields.put("appVersion", new Document().append("$eq", appVersion));
        }
        if (appChannel != null && !appChannel.trim().isEmpty()) {
        	matchFields.put("appChannel", new Document().append("$eq", appChannel));
        }
        
        if (appPlatform != null && !appPlatform.trim().isEmpty()) {
        	matchFields.put("appPlatform", new Document().append("$eq", appPlatform));
        }
        if (errorId != null && !errorId.trim().isEmpty()) {
        	matchFields.put("errorId", new Document().append("$eq", errorId));
        }
        Document match = new Document("$match", matchFields);
        
        
        Document sort = new Document("$sort", Sorts.ascending("timeValue"));
		
		
        Document groupFields = new Document();
        Document idFields = new Document();
        idFields.put("errorId", "$errorId");
        idFields.put("timeValue", "$timeValue");
        idFields.put("appId", "$appId");
        if (appVersion != null && !appVersion.trim().isEmpty()) {
        	idFields.put("appVersion", "$appVersion");
        }
        if (appChannel != null && !appChannel.trim().isEmpty()) {
        	idFields.put("appChannel", "$appChannel");
        }
        
        if (appPlatform != null && !appPlatform.trim().isEmpty()) {
        	idFields.put("appPlatform", "$appPlatform");
        }
        
        if(groupByConditons != null && groupByConditons.size() >0){
	        for(int i=0;i<groupByConditons.size();i++){
	        	String groupByConditon = groupByConditons.get(i);
	        	if ("appVersion".equals(groupByConditon)) {
	            	idFields.put("appVersion", "$appVersion");
	            }
	            if ("appChannel".equals(groupByConditon)) {
	            	idFields.put("appChannel", "$appChannel");
	            }
	            
	            if ("appPlatform".equals(groupByConditon)) {
	            	idFields.put("appPlatform", "$appPlatform");
	            }
	        }
        }
        
        
        idFields.put("deviceStyle", "$deviceStyle");
        idFields.put("osType", "$osType");
        groupFields.put("_id", idFields);
        
        
        
        groupFields.put("errorCnt", new Document("$sum", "$errorCnt"));
		Document group = new Document("$group", groupFields);
		
		
        Document projectFields = new Document();
        projectFields.put("_id", false);
        projectFields.put("errorId", "$_id.errorId");
        projectFields.put("timeValue", "$_id.timeValue");
        projectFields.put("appId", "$_id.appId");
        if (appVersion != null && !appVersion.trim().isEmpty()) {
        	projectFields.put("appVersion", "$_id.appVersion");
        }
        if (appChannel != null && !appChannel.trim().isEmpty()) {
        	projectFields.put("appChannel", "$_id.appChannel");
        }
        
        if (appPlatform != null && !appPlatform.trim().isEmpty()) {
        	projectFields.put("appPlatform", "$_id.appPlatform");
        }
        
        
        if(groupByConditons != null && groupByConditons.size() >0){
	        for(int i=0;i<groupByConditons.size();i++){
	        	String groupByConditon = groupByConditons.get(i);
	        	if ("appVersion".equals(groupByConditon)) {
	        		projectFields.put("appVersion", "$_id.appVersion");
	            }
	            if ("appChannel".equals(groupByConditon)) {
	            	projectFields.put("appChannel", "$_id.appChannel");
	            }
	            
	            if ("appPlatform".equals(groupByConditon)) {
	            	projectFields.put("appPlatform", "$_id.appPlatform");
	            }
	        }
        }
        
        projectFields.put("osType", "$_id.osType");
        projectFields.put("deviceStyle", "$_id.deviceStyle");
        projectFields.put("errorCnt", true);
		Document project = new Document("$project", projectFields);
		AggregateIterable<Document> iterater = collection.aggregate(
				(List<Document>) Arrays.asList(match,  sort, group, project)
				);
		
		MongoCursor<Document>  cursor = iterater.iterator();
		Gson gson = new Gson();
		while(cursor.hasNext()){
			Document document = cursor.next();
			String jsonString = gson.toJson(document);
			AppEorranaly errorInfo = gson.fromJson(jsonString, AppEorranaly.class);
			result.add(errorInfo);
		}
		return result;
	}
	
	public static void main(String[] args) {
		List<AppEorranaly> list = new ErrorAnalyDaoImpl().listErrorInfoBy("youmeng568", "180203", "180508", null, null, null, null, null, null, null);
		System.out.println(list);
	}
}
```

#### 添加时间处理类

```java
package com.lidahai.Utils;

import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.Date;
import java.util.List;

public class DateTimeTools {
	
	public static String getDateStrBy(Long timeMs) {
		SimpleDateFormat sd = new SimpleDateFormat("yyyyMMdd");
		return sd.format(new Date(timeMs));
	}
	
	public static String tranferDateStrBy(String time,String dataformatbefor,String dataformatafter) throws ParseException {
		SimpleDateFormat sd = new SimpleDateFormat(dataformatbefor);
		Date date = sd.parse(time);
		sd = new SimpleDateFormat(dataformatafter);
		String resulttime = sd.format(date);
		return resulttime;
	}
	
	public static Long getThisDayStartedAtMs(Long timeMs) {
		Calendar cal =Calendar.getInstance();
		if (null != timeMs) {
			cal.setTimeInMillis(timeMs);
		}
		cal.set(Calendar.HOUR_OF_DAY, 0);
		cal.set(Calendar.MINUTE, 0);
		cal.set(Calendar.SECOND, 0);
		cal.set(Calendar.MILLISECOND, 0);
		return cal.getTimeInMillis();
	}

	
}
```



### mongo脚本语句的分组求和例子

**1、mongo脚本语句的分组求和**

***mongo文档集合结构：***

![mongo分组求和](clipboard.png)

##### 在服务提供端暴露错误日志类的接口

在common service模块新建公共错误日志接口类(因为这个接口类是公共类，应该要放在common模块供大家调用)

```java
package com.lidahai.service;

import java.util.List;

import com.lidahai.analyentity.AppEorranaly;

public interface ErrorService {
	public List<AppEorranaly> listErrorInfoBy(
			String appId,
			String timeFrom,
			String timeTo, 
			String timeUnit, 
			String appVersion,
			String appChannel, 
			String appPlatform, 
			String userGroup,
			String errorId,
			List<String> groupByConditons
			);
}

```

##### 在SearchSrevice模块新建接口实现类

包名:com.youfan.dao.impl

类名：ErrorAnalyDaoImpl

```java
package com.youfan.service.impl;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;

import com.alibaba.dubbo.config.annotation.Service;
import com.lidahai.analyentity.AppEorranaly;
import com.lidahai.service.ErrorService;
import com.youfan.dao.ErrorAnalyDao;

@Service
public class ErrorAnalyServiceImpl implements ErrorService {

	@Autowired
	ErrorAnalyDao errorAnalyDao;
	
	public List<AppEorranaly> listErrorInfoBy(String appId, String timeFrom, String timeTo, String timeUnit,
			String appVersion, String appChannel, String appPlatform, String userGroup, String errorId,
			List<String> groupByConditons) {
		// TODO Auto-generated method stub
		return errorAnalyDao.listErrorInfoBy(appId, timeFrom, timeTo, timeUnit, appVersion, appChannel, appPlatform, userGroup, errorId, groupByConditons);
	}

}
```

##### 添加配置文件自动扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.springframework.org/schema/beans"  
  xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context"  
  xmlns:tx="http://www.springframework.org/schema/tx" xmlns:cache="http://www.springframework.org/schema/cache"  
  xmlns:p="http://www.springframework.org/schema/p" xsi:schemaLocation="http://www.springframework.org/schema/beans  
     http://www.springframework.org/schema/beans/spring-beans.xsd  
    http://www.springframework.org/schema/aop  
    http://www.springframework.org/schema/aop/spring-aop.xsd  
    http://www.springframework.org/schema/context  
    http://www.springframework.org/schema/context/spring-context.xsd  
    http://www.springframework.org/schema/tx  
    http://www.springframework.org/schema/tx/spring-tx.xsd  
     http://www.springframework.org/schema/cache  
    http://www.springframework.org/schema/cache/spring-cache.xsd">  
      
    <!-- 自动扫描 -->  
    <context:component-scan base-package="com.youfan.**.**"/>
   
</beans>
```

#### 添加dubbo服务提供端需要暴露的接口

暴露错误日志服务接口

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<beans xmlns="http://www.springframework.org/schema/beans"        
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"     
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd 
       http://code.alibabatech.com/schema/dubbo 
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd"> 
            <dubbo:protocol name="dubbo" port="20880"></dubbo:protocol>       
            	<dubbo:application name="dubbo-service-test"/>   	
            	<dubbo:registry address="zookeeper://192.168.37.141:2181" /> 	
            	<dubbo:service interface="com.lidahai.service.ErrorService" ref="errorServiceImpl"/>  
   	<bean id="errorServiceImpl" class="com.youfan.service.impl.ErrorAnalyServiceImpl"/> 
    </beans> 
```

### 搭建前端页面展示maven web项目

#### 搭建ViewService

项目架构分层

![架构分层](1586006839835.png)

#### 添加配置文件application-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"   
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
       http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
       http://www.springframework.org/schema/context   
       http://www.springframework.org/schema/context/spring-context-3.1.xsd">

  <context:component-scan base-package="com" />
  <context:annotation-config />
</beans>
```

#### 添加配置pom依赖

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.youfan</groupId>
  <artifactId>ViewService</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>
  
  <dependencies>
  <dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-webmvc</artifactId>
  		<version>4.2.0.RELEASE</version>
  	</dependency>
  	<dependency>
  		<groupId>javax.servlet</groupId>
  		<artifactId>servlet-api</artifactId>
  		<version>2.4</version>
  	</dependency>
  	<dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
    <dependency> 			
  	<groupId>com.alibaba</groupId> 	
  			<artifactId>dubbo</artifactId> 			
  			<version>2.5.7</version>	
  </dependency> 		
  	<dependency> 			
	 		<groupId>org.apache.zookeeper</groupId> 
	 					<artifactId>zookeeper</artifactId> 		
	 						<version>3.4.6</version> 		
	 						</dependency> 		
<dependency> 
			<groupId>com.github.sgroschupf</groupId>
			 			<artifactId>zkclient</artifactId> 			
			 			<version>0.1</version> 		
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>
<dependency>
<groupId>com.lidahai</groupId>
  <artifactId>AppInfoCommon</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</dependency>
  </dependencies>
</project>
```

#### 修改WEB-INF下的配置文件spring-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx" xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd 
       http://www.springframework.org/schema/context 
       http://www.springframework.org/schema/context/spring-context.xsd 
       http://www.springframework.org/schema/tx 
       http://www.springframework.org/schema/tx/spring-tx.xsd
          http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">
       
<context:component-scan base-package="com.*" />
  <mvc:annotation-driven />
<!-- 
  <bean class="org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping"/>  
  <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter"/>    
 -->

  <mvc:view-controller path="/" view-name="index"/>  

  <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="order" value="1" />
    <property name="prefix" value="/WEB-INF/view/" />
    <property name="suffix" value=".jsp" />
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
  </bean>
  <!--jquery-easyui插件-->
   <mvc:resources mapping="/jquery-easyui-1.5.5.3/**" location="/WEB-INF/classes/js/jquery-easyui-1.5.5.3/" />
   <!--Highcharts插件-->
   <mvc:resources mapping="/Highcharts-6.1.0/**" location="/WEB-INF/classes/js/Highcharts-6.1.0/Highcharts-6.1.0/code/" />
  </beans>
```

把jquery-easyui插件和Highcharts插件放在js目录下即可！插件可以在easyui的官网和Highcharts官网下载。

#### 新建package 

包名:  com.youfan.action

```java
package com.youfan.action;

import java.text.ParseException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.jboss.netty.handler.codec.http.HttpRequest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import com.lidahai.Utils.DateTimeTools;
import com.lidahai.analyentity.AppEorranaly;
import com.lidahai.analyentity.UserInfoAnaly;
import com.lidahai.service.ErrorService;
import com.lidahai.service.UserService;

@Controller
@RequestMapping("errorinfoAction")
public class ErrorinfoAction {
	
	@Autowired
	ErrorService errorService;
	
	@Autowired
	UserService userService;
	
	@RequestMapping("index")
	public String index(){
		return "index";
	}
	
	
	@RequestMapping("errorlist")
	public String errorlist(HttpServletRequest req){
		List<AppEorranaly> list = errorService.listErrorInfoBy("youmeng568", "180203", "180508", null, null, null, null, null, null, null);
		Map<String,Long> mapdata = new HashMap<String,Long>();
		for(AppEorranaly appEorranaly :list){
			String timevalue = appEorranaly.getTimeValue();
			try {
				 timevalue = DateTimeTools.tranferDateStrBy(timevalue,"yyMMdd","yyyyMMdd");
			} catch (ParseException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			long cnt = appEorranaly.getErrorCnt();
			Long precnt = mapdata.get(timevalue)==null?0l:mapdata.get(timevalue);
			mapdata.put(timevalue, cnt+precnt);
		}
		req.setAttribute("xlist", mapdata.keySet());
		req.setAttribute("ylist", mapdata.values());
		System.out.println("hello!!"+list);
		return "ErrorInfo";
	}
	
	
	@RequestMapping("newuserlist")
	public String newuserlist(HttpServletRequest req){
		List<UserInfoAnaly> list = userService.listnewuserInfoBy("youmeng568", "180203", "180508", null, null, null, null, null);
		Map<String,Long> mapdata = new HashMap<String,Long>();
		for(UserInfoAnaly userInfoAnaly :list){
			String timevalue = userInfoAnaly.getTimeValue();
			try {
				 timevalue = DateTimeTools.tranferDateStrBy(timevalue,"yyMMdd","yyyyMMdd");
			} catch (ParseException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			long cnt = userInfoAnaly.getNewUsers();
			Long precnt = mapdata.get(timevalue)==null?0l:mapdata.get(timevalue);
			mapdata.put(timevalue, cnt+precnt);
		}
		req.setAttribute("xlist", mapdata.keySet());
		req.setAttribute("ylist", mapdata.values());
		System.out.println("hello!!"+list);
		return "newuserlist";
	}
	
	
	@RequestMapping("startuplist")
	public String startuplist(HttpServletRequest req){
		List<UserInfoAnaly> list = userService.listStartupInfoBy("youmeng568", "180203", "180508", null, null, null, null, null);
		Map<String,Long> mapdata = new HashMap<String,Long>();
		for(UserInfoAnaly userInfoAnaly :list){
			String timevalue = userInfoAnaly.getTimeValue();
			try {
				 timevalue = DateTimeTools.tranferDateStrBy(timevalue,"yyMMdd","yyyyMMdd");
			} catch (ParseException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			long cnt = userInfoAnaly.getStartupCnts();
			Long precnt = mapdata.get(timevalue)==null?0l:mapdata.get(timevalue);
			mapdata.put(timevalue, cnt+precnt);
		}
		req.setAttribute("xlist", mapdata.keySet());
		req.setAttribute("ylist", mapdata.values());
		System.out.println("hello!!"+list);
		return "startuplist";
	}
	
	
	@RequestMapping("activeuserlist")
	public String activeuserlist(HttpServletRequest req){
		List<UserInfoAnaly> list = userService.listactiveuserInfoBy("youmeng568", "180203", "180508", null, null, null, null, null);
		Map<String,Long> mapdata = new HashMap<String,Long>();
		for(UserInfoAnaly userInfoAnaly :list){
			String timevalue = userInfoAnaly.getTimeValue();
			try {
				 timevalue = DateTimeTools.tranferDateStrBy(timevalue,"yyMMdd","yyyyMMdd");
			} catch (ParseException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			long cnt = userInfoAnaly.getActiveUsers();
			Long precnt = mapdata.get(timevalue)==null?0l:mapdata.get(timevalue);
			mapdata.put(timevalue, cnt+precnt);
		}
		req.setAttribute("xlist", mapdata.keySet());
		req.setAttribute("ylist", mapdata.values());
		System.out.println("hello!!"+list);
		return "activeuserlist";
	}
}

```

##### 在WEB-INF目录下新建view目录添加ErrorInfo.jsp页面

```jsp
<%@ page language="java" contentType="text/html; charset=utf-8"
    pageEncoding="utf-8"%>
<!DOCTYPE HTML>
<html>
    <head>
        <meta charset="utf-8"><link rel="icon" href="/ViewService/Highcharts-6.1.0/images/favicon.ico">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <style>
            /* css 代码  */
        </style>
        <script src="/ViewService/Highcharts-6.1.0/highcharts.js"></script>
        <script src="/ViewService/Highcharts-6.1.0/modules/exporting.js"></script>
        <script src="/ViewService/Highcharts-6.1.0/modules/series-label.js"></script>
        <script src="/ViewService/Highcharts-6.1.0/modules/oldie.js"></script>
        <script src="/ViewService/Highcharts-6.1.0/highcharts-plugins/highcharts-zh_CN.js"></script>
    </head>
    <body>
        <div id="container" style="max-width:800px;height:400px"></div>
        <script>
        var columX = ${xlist};
        var dataY =  ${ylist};
        var chart = Highcharts.chart('container', {
    		title: {
    				text: '错误趋势分析'
    		},
    		yAxis: {
    				title: {
    						text: '发生次数'
    				}
    		},
    		legend: {
    				layout: 'vertical',
    				align: 'right',
    				verticalAlign: 'middle'
    		},
    		 xAxis: {  
  	            categories: columX //x轴标题
  	        },
    		series: [{
    				name: '错误趋势',
    				data: dataY
    		}],
    		responsive: {
    				rules: [{
    						condition: {
    								maxWidth: 500
    						},
    						chartOptions: {
    								legend: {
    										layout: 'horizontal',
    										align: 'center',
    										verticalAlign: 'bottom'
    								}
    						}
    				}]
    		}
    });
        </script>
    </body>
</html>
```

#### 在resources文件夹下添加服务消费端配置文件

文件名:  application-context-consumer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
 <beans xmlns="http://www.springframework.org/schema/beans"        
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"      
   xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"      
     xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">  
        <dubbo:application name="demo-consumer"/>
            <dubbo:registry address="zookeeper://192.168.37.141:2181" />
        <dubbo:reference id="errorService" interface="com.lidahai.service.ErrorService"/>
     </beans> 
```

#### 启动前端服务

![添加前端项目到tomcat8](1586011595225.png)

查看启动日志:

```verilog
2020-04-04 23:16:22,739 INFO  [localhost-startStop-1] logging.SLF4JLogger (SLF4JLogger.java:71) - Cluster created with settings {hosts=[192.168.37.141:27017], mode=SINGLE, requiredClusterType=UNKNOWN, serverSelectionTimeout='30000 ms', maxWaitQueueSize=500}
2020-04-04 23:16:22,792 DEBUG [localhost-startStop-1] logging.SLF4JLogger (SLF4JLogger.java:56) - Updating cluster description to  {type=UNKNOWN, servers=[{address=192.168.37.141:27017, type=UNKNOWN, state=CONNECTING}]
2020-04-04 23:16:22,912 INFO  [cluster-ClusterId{value='5e88a4c6dc18de1bac648734', description='null'}-192.168.37.141:27017] logging.SLF4JLogger (SLF4JLogger.java:71) - Opened connection [connectionId{localValue:1, serverValue:2}] to 192.168.37.141:27017
2020-04-04 23:16:22,912 DEBUG [cluster-ClusterId{value='5e88a4c6dc18de1bac648734', description='null'}-192.168.37.141:27017] logging.SLF4JLogger (SLF4JLogger.java:56) - Checking status of 192.168.37.141:27017
2020-04-04 23:16:22,915 INFO  [cluster-ClusterId{value='5e88a4c6dc18de1bac648734', description='null'}-192.168.37.141:27017] logging.SLF4JLogger (SLF4JLogger.java:71) - Monitor thread successfully connected to server with description ServerDescription{address=192.168.37.141:27017, type=STANDALONE, state=CONNECTED, ok=true, version=ServerVersion{versionList=[3, 2, 22]}, minWireVersion=0, maxWireVersion=4, maxDocumentSize=16777216, roundTripTimeNanos=618710}
2020-04-04 23:16:22,919 DEBUG [cluster-ClusterId{value='5e88a4c6dc18de1bac648734', description='null'}-192.168.37.141:27017] logging.SLF4JLogger (SLF4JLogger.java:56) - Updating cluster description to  {type=STANDALONE, servers=[{address=192.168.37.141:27017, type=STANDALONE, roundTripTime=0.6 ms, state=CONNECTED}]
2020-04-04 23:16:23,074 INFO  [localhost-startStop-1] config.AbstractConfig (AbstractConfig.java:113) - ProducerConfig values: 
	compression.type = none
	metric.reporters = []
	metadata.max.age.ms = 300000
	metadata.fetch.timeout.ms = 60000
	acks = 1
	batch.size = 16384
	reconnect.backoff.ms = 10
	bootstrap.servers = [192.168.38.139:9092]
	receive.buffer.bytes = 32768
	retry.backoff.ms = 100
	buffer.memory = 33554432
	timeout.ms = 30000
	key.serializer = class org.apache.kafka.common.serialization.StringSerializer
	retries = 0
	max.request.size = 1048576
	block.on.buffer.full = true
	value.serializer = class org.apache.kafka.common.serialization.StringSerializer
	metrics.sample.window.ms = 30000
	send.buffer.bytes = 131072
	max.in.flight.requests.per.connection = 5
	metrics.num.samples = 2
	linger.ms = 0
	client.id = 

2020-04-04 23:16:32,149 DEBUG [localhost-startStop-1] internals.Metadata (Metadata.java:141) - Updated cluster metadata version 1 to Cluster(nodes = [Node(192.168.38.139, 9092)], partitions = [])
2020-04-04 23:16:32,189 DEBUG [localhost-startStop-1] producer.KafkaProducer (KafkaProducer.java:231) - Kafka producer started
2020-04-04 23:16:32,189 DEBUG [kafka-producer-network-thread | producer-1] internals.Sender (Sender.java:117) - Starting Kafka producer I/O thread.
四月 04, 2020 11:16:32 下午 org.apache.catalina.core.ApplicationContext log
信息: Initializing Spring FrameworkServlet 'spring'
2020-04-04 23:16:32,599 INFO  [cluster-ClusterId{value='5e88a4d0dc18de1bac648735', description='null'}-192.168.37.141:27017] logging.SLF4JLogger (SLF4JLogger.java:71) - Opened connection [connectionId{localValue:2, serverValue:3}] to 192.168.37.141:27017
2020-04-04 23:16:32,599 DEBUG [cluster-ClusterId{value='5e88a4d0dc18de1bac648735', description='null'}-192.168.37.141:27017] logging.SLF4JLogger (SLF4JLogger.java:56) - Checking status of 192.168.37.141:27017
2020-04-04 23:16:32,605 INFO  [cluster-ClusterId{value='5e88a4d0dc18de1bac648735', description='null'}-192.168.37.141:27017] logging.SLF4JLogger (SLF4JLogger.java:71) - Monitor thread successfully connected to server with description ServerDescription{address=192.168.37.141:27017, type=STANDALONE, state=CONNECTED, ok=true, version=ServerVersion{versionList=[3, 2, 22]}, minWireVersion=0, maxWireVersion=4, maxDocumentSize=16777216, roundTripTimeNanos=5462544}
2020-04-04 23:16:32,605 DEBUG [cluster-ClusterId{value='5e88a4d0dc18de1bac648735', description='null'}-192.168.37.141:27017] logging.SLF4JLogger (SLF4JLogger.java:56) - Updating cluster description to  {type=STANDALONE, servers=[{address=192.168.37.141:27017, type=STANDALONE, roundTripTime=5.5 ms, state=CONNECTED}]
2020-04-04 23:16:32,922 DEBUG [cluster-ClusterId{value='5e88a4c6dc18de1bac648734', description='null'}-192.168.37.141:27017] logging.SLF4JLogger (SLF4JLogger.java:56) - Checking status of 192.168.37.141:27017
2020-04-04 23:16:32,924 DEBUG [cluster-ClusterId{value='5e88a4c6dc18de1bac648734', description='null'}-192.168.37.141:27017] logging.SLF4JLogger (SLF4JLogger.java:56) - Updating cluster description to  {type=STANDALONE, servers=[{address=192.168.37.141:27017, type=STANDALONE, roundTripTime=0.7 ms, state=CONNECTED}]
四月 04, 2020 11:16:33 下午 org.apache.catalina.core.ApplicationContext log
信息: 2020-04-04 23:16:32,585 INFO  [localhost-startStop-1] logging.SLF4JLogger (SLF4JLogger.java:71) - Cluster created with settings {hosts=[192.168.37.141:27017], mode=SINGLE, requiredClusterType=UNKNOWN, serverSelectionTimeout='30000 ms', maxWaitQueueSize=500}
2020-04-04 23:16:32,586 DEBUG [localhost-startStop-1] logging.SLF4JLogger (SLF4JLogger.java:56) - Updating cluster description to  {type=UNKNOWN, servers=[{address=192.168.37.141:27017, type=UNKNOWN, state=CONNECTING}]

四月 04, 2020 11:16:33 下午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["http-apr-8080"]
四月 04, 2020 11:16:33 下午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["ajp-apr-8009"]
四月 04, 2020 11:16:33 下午 org.apache.catalina.startup.Catalina start
信息: Server startup in 22645 ms
```

从日志上看默认端口是8080

浏览器输入:

localhost:8080/ViewService

