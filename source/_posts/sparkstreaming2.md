---
layout: sparkstreaming2
title: SparkStreaming实时计算系统(二)之错误日志上报服务
date: 2020-04-01 20:12:48
tags: sparkstreaming
comments: true
---

### 构建开发环境

开发工具:

Eclipse

maven

开发语言： 

Java1.8

### 本次目的

数据上报：

把设备错误日志和启动日志分别通过kafka的topic消息队列发送到hive和spark streaming 端然后做后续处理，其中启动日志宁存到mongodb，用于更新mongdb的数据。

#### 单独新建maven项目,添加依赖

项目名称AppInfoCommon简称common模块，用于封装公共实体类

<!--more-->

```xml
<modelVersion>4.0.0</modelVersion>
  <groupId>com.lidahai</groupId>
  <artifactId>AppInfoCommon</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  
  <dependencies>
  	<dependency>
      <groupId>com.typesafe</groupId>
      <artifactId>config</artifactId>
      <version>1.2.1</version>
    </dependency>
  </dependencies>
```

再单独新建一个项目AppInfoServices简称InfoServices,并添加依赖，用于完成测试数据上报

```xml
<modelVersion>4.0.0</modelVersion>
  <groupId>com.lidahai</groupId>
  <artifactId>AppInfoServices</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>
  
  <dependencies>
  <dependency>
  	<groupId>com.lidahai</groupId>
  	<artifactId>AppInfoCommon</artifactId>
  	<version>0.0.1-SNAPSHOT</version>
  </dependency>
  <dependency>
  	<groupId>com.lidahai</groupId>
  	<artifactId>KafkaUtil</artifactId>
  	<version>0.0.1-SNAPSHOT</version>
  </dependency>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-webmvc</artifactId>
  		<version>3.1.0.RELEASE</version>
  	</dependency>
  	<dependency>
  		<groupId>javax.servlet</groupId>
  		<artifactId>servlet-api</artifactId>
  		<version>2.4</version>
  	</dependency>
  	<dependency>
            <groupId>jdk.tools</groupId>
            <artifactId>jdk.tools</artifactId>
            <version>1.8</version>
            <scope>system</scope>
            <systemPath>C:\\Program Files\\Java\\jdk1.8.0_211\\lib\\tools.jar</systemPath>
        </dependency>
  	 <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>2.3</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-web</artifactId>
      <version>2.3</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-slf4j-impl</artifactId>
      <version>2.3</version>
    </dependency>
  	<dependency>
            <groupId>net.sf.json-lib</groupId>
            <artifactId>json-lib</artifactId>
            <version>2.4</version>
            <classifier>jdk15</classifier>
        </dependency>
         <dependency>
      <groupId>com.typesafe</groupId>
      <artifactId>config</artifactId>
      <version>1.2.1</version>
    </dependency>
     <dependency>
      <groupId>org.mongodb</groupId>
      <artifactId>mongodb-driver</artifactId>
      <version>3.0.1</version>
</dependency>
 <dependency>
            <groupId>net.sf.json-lib</groupId>
            <artifactId>json-lib</artifactId>
            <version>2.4</version>
            <classifier>jdk15</classifier>
        </dependency>
<dependency>
            <groupId>org.codehaus.jackson</groupId>
            <artifactId>jackson-mapper-asl</artifactId>
            <version>1.4.2</version>
        </dependency>
        <dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
  </dependencies>
  
  <build>
  <finalName>AppInfoServices</finalName>
  <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.3</version>
        <configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>utf8</encoding>
		</configuration>
      </plugin>
    </plugins>
  </build>
```

### 添加实体类

在common 模块新建package: com.lidahai.logentity并添加错误日志实体类

```java
package com.lidahai.logentity;

/**
 * ������־ʵ��
 * @author Administrator
 *
 */
public class AppEorrLog {
	private Long createdAtMs;			
	private String appId;				
	private String deviceId;			
	private String appVersion;			
	private String appChannel;			
	private String appPlatform;			
	private String osType;				
	private String deviceStyle;			
	private String errorBrief;		
	private String errorDetail;
	public Long getCreatedAtMs() {
		return createdAtMs;
	}
	public void setCreatedAtMs(Long createdAtMs) {
		this.createdAtMs = createdAtMs;
	}
	public String getAppId() {
		return appId;
	}
	public void setAppId(String appId) {
		this.appId = appId;
	}
	public String getDeviceId() {
		return deviceId;
	}
	public void setDeviceId(String deviceId) {
		this.deviceId = deviceId;
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
	public String getOsType() {
		return osType;
	}
	public void setOsType(String osType) {
		this.osType = osType;
	}
	public String getDeviceStyle() {
		return deviceStyle;
	}
	public void setDeviceStyle(String deviceStyle) {
		this.deviceStyle = deviceStyle;
	}
	public String getErrorBrief() {
		return errorBrief;
	}
	public void setErrorBrief(String errorBrief) {
		this.errorBrief = errorBrief;
	}
	public String getErrorDetail() {
		return errorDetail;
	}
	public void setErrorDetail(String errorDetail) {
		this.errorDetail = errorDetail;
	}

}

```

### 添加设备启动日志实体类

在common模块新建package: com.lidahai.logentity并添加启动日志实体类

```
package com.lidahai.logentity;

/**
 * 启动日志
 * @author Administrator
 *
 */
public class StartupLog {
	private Long createdAtMs; //启动时间	
	private String appId;				//应用唯一标识
	private String deviceId;			//设备唯一标识
	private String appVersion;			//版本
	private String appChannel;			//渠道
	private String appPlatform;			//平台
	private String network;				//网络
	private String carrier;				//运营商

	private String brand;               //品牌
	private String deviceStyle;			//机型
	private String screenSize;			//分辨率
	private String osType;				//操作系统
	private Boolean isFirstVisitToday = false;	//当日首次访问

	private Boolean isNewUser = false;	//新安装用户

	public Long getCreatedAtMs() {
		return createdAtMs;
	}

	public void setCreatedAtMs(Long createdAtMs) {
		this.createdAtMs = createdAtMs;
	}

	public String getAppId() {
		return appId;
	}

	public void setAppId(String appId) {
		this.appId = appId;
	}

	public String getDeviceId() {
		return deviceId;
	}

	public void setDeviceId(String deviceId) {
		this.deviceId = deviceId;
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

	public String getNetwork() {
		return network;
	}

	public void setNetwork(String network) {
		this.network = network;
	}

	public String getCarrier() {
		return carrier;
	}

	public void setCarrier(String carrier) {
		this.carrier = carrier;
	}

	public String getBrand() {
		return brand;
	}

	public void setBrand(String brand) {
		this.brand = brand;
	}

	public String getDeviceStyle() {
		return deviceStyle;
	}

	public void setDeviceStyle(String deviceStyle) {
		this.deviceStyle = deviceStyle;
	}

	public String getScreenSize() {
		return screenSize;
	}

	public void setScreenSize(String screenSize) {
		this.screenSize = screenSize;
	}

	public String getOsType() {
		return osType;
	}

	public void setOsType(String osType) {
		this.osType = osType;
	}

	public Boolean getIsFirstVisitToday() {
		return isFirstVisitToday;
	}

	public void setIsFirstVisitToday(Boolean isFirstVisitToday) {
		this.isFirstVisitToday = isFirstVisitToday;
	}

	public Boolean getIsNewUser() {
		return isNewUser;
	}

	public void setIsNewUser(Boolean isNewUser) {
		this.isNewUser = isNewUser;
	}

}
```

### 添加设备实体类

```java
package com.lidahai;

/**
 * �豸��Ϣ
 * @author Administrator
 *
 */
public class Widget {
	private String id;			//��Ӧmongodb�е�_id����
	private String appId;       //Ӧ��id
	private String deviceId;   //�豸��
	private String appPlatform; //ƽ̨
	private Long firstVisitAtMs;	//�״η���ʱ��
	private Long lastVisitAtMs;		//���һ�η���ʱ��
	private Long lastVistAtMsToday;			//ÿ���һ�η��ʵ�ʱ��
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getAppId() {
		return appId;
	}
	public void setAppId(String appId) {
		this.appId = appId;
	}
	public String getDeviceId() {
		return deviceId;
	}
	public void setDeviceId(String deviceId) {
		this.deviceId = deviceId;
	}
	public String getAppPlatform() {
		return appPlatform;
	}
	public void setAppPlatform(String appPlatform) {
		this.appPlatform = appPlatform;
	}
	public Long getFirstVisitAtMs() {
		return firstVisitAtMs;
	}
	public void setFirstVisitAtMs(Long firstVisitAtMs) {
		this.firstVisitAtMs = firstVisitAtMs;
	}
	public Long getLastVisitAtMs() {
		return lastVisitAtMs;
	}
	public void setLastVisitAtMs(Long lastVisitAtMs) {
		this.lastVisitAtMs = lastVisitAtMs;
	}
	public Long getLastVistAtMsToday() {
		return lastVistAtMsToday;
	}
	public void setLastVistAtMsToday(Long lastVistAtMsToday) {
		this.lastVistAtMsToday = lastVistAtMsToday;
	}
	
	

}
```



### 添加上报服务接口类

在InfoServices项目的src/main/java文件下新建package: com.service.contorller并添加上报服务接口类

```java
package com.service.contorller;

import java.io.IOException;
import java.io.PrintWriter;
import java.text.DateFormat;
import java.text.SimpleDateFormat;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.bson.Document;
import org.bson.types.ObjectId;
import org.codehaus.jackson.JsonGenerationException;
import org.codehaus.jackson.map.JsonMappingException;
import org.codehaus.jackson.map.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpRequest;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Controller;
import org.springframework.web.HttpRequestMethodNotSupportedException;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.google.gson.Gson;
import com.lidahai.ProudceSenderUtils;
import com.lidahai.Widget;
import com.lidahai.Utils.DateTimeTools;
import com.lidahai.logentity.AppEorrLog;
import com.lidahai.logentity.StartupLog;
import com.lidahai.utils.PropertityUtils;
import com.service.base.WidgetDao;

import net.sf.json.JSONObject;

@Controller
@RequestMapping("/appinfoservice")
public class AppInfoService {
	@Autowired
	private ProudceSenderUtils kafkaMessageProducer;
	private final static String apperrorspark = PropertityUtils.getKey("apperrorspark");//������־��spark��Ϣ����
	private final static String apperorrhive= PropertityUtils.getKey("apperorrhive");//������־��hive��Ϣ����

	
	private final static String appstartupspark = PropertityUtils.getKey("appstartupspark");//������־����Ϣ����
	Logger logger = LogManager.getLogger();//�����־
	private WidgetDao widgetDao = new WidgetDao(); 
	private ObjectMapper mapper = new ObjectMapper();
	
	@RequestMapping("/hello")
	public void sayhello(HttpServletRequest req,HttpServletResponse resp){
		logger.debug("������Ŷ ����������");
		PrintWriter printWriter = getJsonWriter(resp);
		resp.setStatus(HttpStatus.OK.value());
		printWriter.println("hello "+req.getRemoteAddr());
		closePringWriter(printWriter);
	}

	private void closePringWriter(PrintWriter printWriter) {
		if(printWriter != null){
			printWriter.flush();
			printWriter.close();
		}
		
	}
	
	@RequestMapping(value="/applogService",method=RequestMethod.POST)
	public void appInfoService(@RequestBody String jsonstr,HttpServletResponse reqs){
		logger.debug("������־�ϱ������ˣ���������");
		JSONObject jsonObject = JSONObject.fromObject(jsonstr);
		String hivemessage = transerformtohive(jsonObject);//hive��װ��������ݸ�ʽ
		String sparkstreamingmessage = "com.lidahai.logentity.AppEorrLog:"+jsonstr;
		logger.debug("hive errorLog message"+hivemessage);
        //发送到kafka，apperorrhive是topic,hivemessage是消息
		 kafkaMessageProducer.sendMessage(apperorrhive, hivemessage);
		logger.debug("spark errorLog message"+sparkstreamingmessage);
        //发送到sparkstreaming，apperrorspark是topic，sparkstreamingmessage是消息
		kafkaMessageProducer.sendMessage(apperrorspark,sparkstreamingmessage);
        
		PrintWriter printWriter = getJsonWriter(reqs);
		reqs.setStatus(HttpStatus.OK.value());
		printWriter.println("success");
		closePringWriter(printWriter);
	}

	//转换
	private String transerformtohive(JSONObject jsonObject) {
		AppEorrLog appEorrLog = (AppEorrLog) jsonObject.toBean(jsonObject, AppEorrLog.class);
		Long  createdAtMs = appEorrLog.getCreatedAtMs();
		String appId = appEorrLog.getAppId();
		String deviceId = appEorrLog.getDeviceId();
		String appVersion=appEorrLog.getAppVersion();
		String appChannel=appEorrLog.getAppChannel();
		String appPlatform=appEorrLog.getAppPlatform();
		String osType=appEorrLog.getOsType();
		String deviceStyle=appEorrLog.getDeviceStyle();
		String errorBrief=appEorrLog.getErrorBrief();
		String errorDetail=appEorrLog.getErrorDetail();
		String timesecond = transfertime(createdAtMs);
		String result = createdAtMs + "\t"+appId+"\t"+deviceId+"\t"+appVersion+"\t"+appChannel
				+"\t"+appPlatform+"\t"+osType+"\t"+deviceStyle+"\t"+errorBrief+"\t"+errorDetail+"\t"+timesecond;
		return result;
	}
	//时间格式化
	private String transfertime(Long time){
		DateFormat dateFormat = new SimpleDateFormat("yyyyMMddhhmmss");
		String result =dateFormat.format(time);
		return result;
	}

	private PrintWriter getJsonWriter(HttpServletResponse resp) {
		resp.setCharacterEncoding("utf-8");
		resp.setContentType("application/json");
		PrintWriter printWriter = null;
		try {
			printWriter = resp.getWriter();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			logger.error("����������"+e.getMessage());
		}
		return printWriter;
	}
    
    、、
	@RequestMapping(value="/startuplogService",method=RequestMethod.POST)
	public void startuplogService(@RequestBody String jsonstr,HttpServletResponse reqs){
		logger.debug("������־�ϱ������ˣ���������");
		JSONObject jsonObject = JSONObject.fromObject(jsonstr);
		StartupLog startupLog = (StartupLog) JSONObject.toBean(jsonObject,StartupLog.class);
		Widget widget = getWidget(startupLog);
		setupstaruplog(widget,startupLog);
		//������Ϣ
		String msgBody = "";
		try {
			msgBody = StartupLog.class.getName() + ":" + mapper.writeValueAsString(startupLog);
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} 
        //发送消息到kafka的topic"appstartupspark"，msgBody是消息
		kafkaMessageProducer.sendMessage(appstartupspark, msgBody);
        //宁外一份保存到mongodb
		saveOrUpdateWidget(widget,startupLog);
        
		PrintWriter printWriter = getJsonWriter(reqs);
		reqs.setStatus(HttpStatus.OK.value());
		printWriter.println("success");
		closePringWriter(printWriter);
	}
	
	
	private Widget getWidget(StartupLog startupLog){
		String appId = startupLog.getAppId();
		String deviceId = startupLog.getDeviceId();
		Widget widget = null;
		Document widgetDoc = this.widgetDao.findBy(appId, deviceId);
		if (widgetDoc != null) { //如果mongodb的记录已经存在
			Gson gson = new Gson();
			widget = gson.fromJson(gson.toJson(widgetDoc), Widget.class);
			widget.setId(widgetDoc.getObjectId("_id").toHexString()); //就用原来的_id
		} else {  d
					widget = new Widget();
					widget.setAppId(appId);
					widget.setDeviceId(deviceId);
					widget.setAppPlatform(startupLog.getAppPlatform());
					widget.setFirstVisitAtMs(startupLog.getCreatedAtMs());
                    //否则九新建ObjectId
					widget.setId(new ObjectId().toHexString());
		}
		
		return widget;
	}
	
	private void setupstaruplog(Widget widget,StartupLog startupLog){
		Long firstVisitAtMs = widget.getFirstVisitAtMs(); //mongodb里面存在的时间
		Long timeNowMs = startupLog.getCreatedAtMs();  //上报的启动日志时间
		//判断是否是新用户
		if (timeNowMs == firstVisitAtMs) {
			startupLog.setIsNewUser(true); //判定是新用户
		}
		//
		if (widget.getLastVistAtMsToday() == null   //mongodb里面没有最后的访问记录
						|| widget.getLastVistAtMsToday() < //或者mongo里面最后的访问记录小于启动日志的时间
            DateTimeTools.getThisDayStartedAtMs(timeNowMs)) {
			startupLog.setIsFirstVisitToday(true);  //判定是今天第一次访问
		}

	}
	
	private void saveOrUpdateWidget(Widget widget, StartupLog startupLog){
		widget.setLastVisitAtMs(startupLog.getCreatedAtMs());
		if (startupLog.getIsFirstVisitToday()) {
					widget.setLastVistAtMsToday(startupLog.getCreatedAtMs());
		}
		ObjectMapper mapper = new ObjectMapper();
		Document widgetDoc = null;
		try {
					widgetDoc = Document.parse(mapper.writeValueAsString(widget));
		} catch (Exception e) {
					e.printStackTrace();
		}
		widgetDoc.put("_id", new ObjectId(widget.getId()));
		widgetDao.saveOrUpdate(widgetDoc);

	}
	
}
```

#### 添加widgetDao类

widget是设备的意思,作用: 保存设备的信息到mongodb

```java
import org.codehaus.jackson.map.ObjectMapper;

import com.lidahai.Utils.PropertyRead;
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;

public class WidgetDao {
	private MongoClient mongoClient = new MongoClient(PropertyRead.getkey("mongoaddr"),Integer.valueOf(PropertyRead.getkey("mongoport")));
	ObjectMapper objectMapper = new ObjectMapper();
	public WidgetDao(){
		
		
	}
    //根据mongodb的对应的表Collection查找对应的行doc
	public Document findBy(String appId, String deviceId) {
		MongoDatabase db = mongoClient.getDatabase(appId);
		FindIterable<Document> iterable = db.getCollection("Widget").find(
		        new Document("deviceId", deviceId));
		if (iterable.iterator().hasNext()) {
			return iterable.first();
		} else {
			return null;
		}
	}
    //插入或者更新
	public void saveOrUpdate(Document widgetDoc) {
		String tableName = "Widget";
		MongoDatabase db = mongoClient.getDatabase(widgetDoc.getString("appId"));
		MongoCollection<Document> collection = db.getCollection(tableName);

		if (!widgetDoc.containsKey("_id")) {
			ObjectId objId = new ObjectId();
			widgetDoc.put("_id", objId);
			collection.insertOne(widgetDoc);
			return;
		}
        
		
		String objectId =  widgetDoc.get("_id").toString();
		Document matchFields = new Document();
		matchFields.put("_id", new ObjectId(objectId));
		if(collection.find(matchFields).iterator().hasNext()){
            //有数据就执行更新操作
			collection.updateOne(matchFields, new Document("$set",widgetDoc));
		}else{
            //没有数据就执行添加操作
			collection.insertOne(widgetDoc);
		}
	}

	
	
}

```

#### 添加工具类

用于读取application.properties文件里面键值对配置

```java
package com.service.utils;


import com.typesafe.config.Config;
import com.typesafe.config.ConfigFactory;

public class PropertityUtils {
	public final static Config config = ConfigFactory.load();
	public static String getKey(String key){
		return config.getString(key).trim();
	}
}
```

#### 添加kafka工具类

新建一个KafkaUtil maven项目或者模块

添加pom依赖

```xml
<modelVersion>4.0.0</modelVersion>
  <groupId>com.lidahai</groupId>
  <artifactId>KafkaUtil</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  
  <dependencies>
  	<dependency>
	  <groupId>org.apache.kafka</groupId>
	  <artifactId>kafka_2.10</artifactId>
	  <version>0.8.2.1</version>
	</dependency>    
    <dependency>
	  <groupId>org.apache.kafka</groupId>
	  <artifactId>kafka-clients</artifactId>
	  <version>0.8.2.1</version>
	</dependency>    
    <dependency>  
      <groupId>org.springframework</groupId>  
      <artifactId>spring-context</artifactId>  
      <version>3.1.0.RELEASE</version>  
    </dependency>  
    <dependency>
      <groupId>com.typesafe</groupId>
      <artifactId>config</artifactId>
      <version>1.2.1</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>2.3</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-web</artifactId>
      <version>2.3</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-slf4j-impl</artifactId>
      <version>2.3</version>
</dependency>
  </dependencies>
```

###### 添加kafka工具类

```java
package com.lidahai;

import java.util.Properties;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;
import org.apache.log4j.LogManager;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.DisposableBean;

import com.lidahai.utils.PropertityUtils;

public class ProudceSenderUtils implements DisposableBean{
	private Logger logger = LogManager.getLogger(ProudceSenderUtils.class);
	private KafkaProducer< String, String> producer = null;
	public ProudceSenderUtils(){
		Properties pro = new Properties();
		pro.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, PropertityUtils.getKey("brokerList"));
		pro.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
		pro.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
		producer = new KafkaProducer<String, String>(pro);
	}
	
	public void sendMessage(String topic,String message){
		ProducerRecord<String,String> producerRecord = new ProducerRecord<String,String>(topic,message);
		producer.send(producerRecord , new Callback() {
			
			public void onCompletion(RecordMetadata metadata, Exception exception) {
				if(exception != null){
					logger.error("发送消息失败！！");
				}else{
					logger.error("发送消息成功！！"+metadata.offset()+"=="+metadata.partition()+"=="+metadata.topic());
				}
				
			}
		});
	}
	
	public void destroy() throws Exception {
		if(producer != null){
			producer.close();
		}
		
	}

}
```

### 添加application.properties配置文件

brokerList=192.168.37.141:9092

添加全局配置文件application-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"   
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
       http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
       http://www.springframework.org/schema/context   
       http://www.springframework.org/schema/context/spring-context-3.1.xsd">

  <description>全局性配置文件</description>

  <bean id="kafkaMessageProducer" class="com.lidahai.ProudceSenderUtils" />

</beans>
```



### 在infoService项目添加文件application.properties添加配置

```java
brokerList=192.168.37.141:9092
apperrorspark=apperrorspark
apperorrhive=apperorrhive
appstartupspark=appstartupspark
mongoaddr=192.168.37.141
mongoport=27017
```

提示请根据上述文件提到的kafka的topic名字在kafka集群创建三个相应的topic: apperrorspark,apperorrhive,appstartupspark

由于创建的topic命令过于简单，可以百度自行搜索，这里不再详细讲解。

### 添加错误日志数据上报测试类

在InfoServices项目的test文件下新建package: com.lidahai.appclient并添加数据上报类appClient

```java
package com.lidahai.appclient;

import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

import com.lidahai.logentity.AppEorrLog;

import net.sf.json.JSONObject;

public class appClient {
	
    private static String url = "http://127.0.0.1:8080/AppInfoServices/appinfoservice/applogService";
    
    private static Random random = new Random();
    
    private static String appId = "youmeng568";
    
    private static String[] deviceIds = initDeviceId();
    private static String appVersion = "3.2.1";
    private static String appChannel = "baidu";
    private static String appPlatform = "android";

	private static String[] deviceStyles = {"iPhone 6","iPhone 6 Plus","�����ֻ�1s"};			//����
	private static String[] osTypes = {"8.3","7.1.1"};				//����ϵͳ
	private static Long[] times = inittime();
	
	private static String[] errorBriefs = {"at cn.xxx.appto55.web.AbstractBaseController.validInbound(AbstractBaseController.java:72)"
			,"at cn.xxx.appto55.control.CommandUtil.getInfo(CommandUtil.java:67)"
	};		//����ժҪ
	private static String[] errorDetails = {"java.lang.NullPointerException    " 
	+ "at cn.sbz.appto55.web.AbstractBaseController.validInbound(AbstractBaseController.java:72) "+
			"at cn.xxx.appto.web.AbstractBaseController.validInbound",
			"at cn.xxx.appto.control.CommandUtil.getInfo(CommandUtil.java:67) "+
							"at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)" +
							" at java.lang.reflect.Method.invoke(Method.java:606)"};		//��������
	private static AppEorrLog[] appErrorLogs = initAppErrorLogs();			//���������Ϣ������
    
    private static String[]  initDeviceId(){
    	String base = "device22";
    	String [] result = new String[100];
    	for(int i=0;i<100;i++){
    		result[i] = base+i+"";
    	}
    	return result;
    }
    
    private static Long[]  inittime(){
    	Long[] timearray = new Long[10];
    	DateFormat dateFomat = new SimpleDateFormat("yyyy-MM-dd");
    	String[] teimstring = new String[]{"2018-02-03","2018-02-04","2018-02-05","2018-02-06","2018-02-07","2018-02-08","2018-02-09","2018-02-10","2018-02-11","2018-02-12"};
    	for(int i=0;i<timearray.length;i++){
    		Date timetemp;
			try {
				timetemp = dateFomat.parse(teimstring[i]);
	    		timearray[i] = timetemp.getTime();
			} catch (ParseException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
    		
    	}
    	return timearray;
    }
    

	//���������Ϣ������
	private static AppEorrLog[] initAppErrorLogs(){
		AppEorrLog[] result = new AppEorrLog[10];
		for(int i=0;i<10;i++){
			AppEorrLog appErrorLog = new AppEorrLog();
			appErrorLog.setAppId(appId);
			appErrorLog.setDeviceId(deviceIds[random.nextInt(deviceIds.length)]);
			appErrorLog.setAppVersion(appVersion);
			appErrorLog.setAppChannel(appChannel);
			appErrorLog.setAppPlatform(appPlatform);
			appErrorLog.setErrorBrief(errorBriefs[random.nextInt(errorBriefs.length)]);
			appErrorLog.setErrorDetail(errorDetails[random.nextInt(errorDetails.length)]);
			appErrorLog.setCreatedAtMs(times[random.nextInt(times.length)]);
			appErrorLog.setOsType(osTypes[random.nextInt(osTypes.length)]);
			appErrorLog.setDeviceStyle(deviceStyles[random.nextInt(deviceStyles.length)]);
    		result[i] = appErrorLog;
    	}
    	return result;
	}	
	
    //模拟浏览器请求数据
    private static void httpPost(String urlString, String params) {
        URL url;

        try {
            url = new URL(urlString);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("POST");
            conn.setDoOutput(true);
            conn.setDoInput(true);
            conn.setUseCaches(false);
            conn.setInstanceFollowRedirects(true);
            conn.setRequestProperty("User-Agent", "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:26.0) Gecko/20100101 Firefox/26.0");
            conn.setRequestProperty("Content-Type", "application/json");
            conn.setConnectTimeout(1000 * 5);
            conn.connect();
            conn.getOutputStream().write(params.getBytes("utf8"));
            conn.getOutputStream().flush();
            conn.getOutputStream().close();
            byte[] buffer = new byte[1024];  
		      StringBuffer sb = new StringBuffer();  
		      InputStream in = conn.getInputStream();  
		      int httpCode = conn.getResponseCode();  
		      System.out.println(in.available());  
		      while(in.read(buffer,0,1024) != -1) {  
		          sb.append(new String(buffer));  
		      }  
		      System.out.println("sb:" + sb.toString());  
		      in.close();  
		      System.out.println(httpCode);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
   

    public static final void main(String[] args) {
    	Test1();
//    	Map<String,Long> test = new HashMap<String,Long>();
//    	test.put("hehe", 1l);
//    	test.put("haha", 2l);
//    	JSONObject jsonObject = JSONObject.fromObject(test);
//		String jsonstring = jsonObject.toString();
//		System.out.println(jsonstring);
    }

	private static void Test1() {
        try {
            //��������
            for (int i = 1; i <= appErrorLogs.length-1; i++) {
            	AppEorrLog appErrorLog = appErrorLogs[i];
            	
            	try {
            		JSONObject jsonObject = JSONObject.fromObject(appErrorLog);
            		String jsonstring = jsonObject.toString();
                    httpPost(url,  jsonstring);
                    System.err.println("���ڷ������ݣ�" + i + "::" + jsonstring);
                } catch (Exception ex) {
                    System.out.println(ex);
                }
            }
        } catch (Exception ex) {
        	ex.printStackTrace();
        }
	}
   
}
```

### 添加设备启动日志数据上报测试类

在InfoServices项目的test文件下新建package: com.lidahai.appclient并添加数据上报类appStartupClient

```java
package com.lidahai.appclient;

import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

import com.lidahai.logentity.AppEorrLog;
import com.lidahai.logentity.StartupLog;

import net.sf.json.JSONObject;

public class appStartupClient {
	
    private static String url = "http://127.0.0.1:8080/AppInfoServices/appinfoservice/startuplogService";
    
    private static Random random = new Random();
    
    private static String appId = "youmeng568";
    
    private static String[] deviceIds = initDeviceId();
    private static String appVersion = "3.2.1";
    private static String appChannel = "baidu";
    private static String appPlatform = "android";

	private static String[] deviceStyles = {"iPhone 6","iPhone 6 Plus","红米手机1s"};			//机型
	private static String[] osTypes = {"8.3","7.1.1"};				//操作系统
	
	private static String[] network = {"百度","谷歌"};				//网络
	private static String[] carrier ={"移动","联通"};				//运营商
	private static String[] brand ={"小米","华为","魅族"};               //品牌
	private static String[] screenSize ={"600*1200","200*400"};			//分辨率
	private static Long[] times = inittime();
	

	private static StartupLog[] startuplogs = initStartupLogs();			//用户启动日志数据
    
    private static String[]  initDeviceId(){
    	String base = "device22";
    	String [] result = new String[100];
    	for(int i=0;i<100;i++){
    		result[i] = base+i+"";
    	}
    	return result;
    }
    
    private static Long[]  inittime(){
    	Long[] timearray = new Long[10];
    	DateFormat dateFomat = new SimpleDateFormat("yyyy-MM-dd");
    	String[] teimstring = new String[]{"2018-02-03","2018-02-04","2018-02-05","2018-02-06","2018-02-07","2018-02-08","2018-02-09","2018-02-10","2018-02-11","2018-02-12"};
    	for(int i=0;i<timearray.length;i++){
    		Date timetemp;
			try {
				timetemp = dateFomat.parse(teimstring[i]);
	    		timearray[i] = timetemp.getTime();
			} catch (ParseException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
    		
    	}
    	return timearray;
    }
    

	//错误相关信息的数组
	private static StartupLog[] initStartupLogs(){
		StartupLog[] result = new StartupLog[10];
		for(int i=0;i<10;i++){
			StartupLog startupLog = new StartupLog();
			startupLog.setAppId(appId);
			startupLog.setDeviceId(deviceIds[random.nextInt(deviceIds.length)]);
			startupLog.setAppVersion(appVersion);
			startupLog.setAppChannel(appChannel);
			startupLog.setAppPlatform(appPlatform);
			startupLog.setCreatedAtMs(times[random.nextInt(times.length)]);
			startupLog.setOsType(osTypes[random.nextInt(osTypes.length)]);
			startupLog.setNetwork(network[random.nextInt(network.length)]);
			startupLog.setCarrier(carrier[random.nextInt(carrier.length)]);
			startupLog.setBrand(brand[random.nextInt(brand.length)]);
			startupLog.setScreenSize(screenSize[random.nextInt(screenSize.length)]);
			startupLog.setDeviceStyle(deviceStyles[random.nextInt(deviceStyles.length)]);
    		result[i] = startupLog;
    	}
    	return result;
	}	
	
    private static void httpPost(String urlString, String params) {
        URL url;

        try {
            url = new URL(urlString);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("POST");
            conn.setDoOutput(true);
            conn.setDoInput(true);
            conn.setUseCaches(false);
            conn.setInstanceFollowRedirects(true);
            conn.setRequestProperty("User-Agent", "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:26.0) Gecko/20100101 Firefox/26.0");
            conn.setRequestProperty("Content-Type", "application/json");
            conn.setConnectTimeout(1000 * 5);
            conn.connect();
            conn.getOutputStream().write(params.getBytes("utf8"));
            conn.getOutputStream().flush();
            conn.getOutputStream().close();
            byte[] buffer = new byte[1024];  
		      StringBuffer sb = new StringBuffer();  
		      InputStream in = conn.getInputStream();  
		      int httpCode = conn.getResponseCode();  
		      System.out.println(in.available());  
		      while(in.read(buffer,0,1024) != -1) {  
		          sb.append(new String(buffer));  
		      }  
		      System.out.println("sb:" + sb.toString());  
		      in.close();  
		      System.out.println(httpCode);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
   

    public static final void main(String[] args) {
    	Test1();
//    	Map<String,Long> test = new HashMap<String,Long>();
//    	test.put("hehe", 1l);
//    	test.put("haha", 2l);
//    	JSONObject jsonObject = JSONObject.fromObject(test);
//		String jsonstring = jsonObject.toString();
//		System.out.println(jsonstring);
    }

	private static void Test1() {
        try {
            //发送数据
            for (int i = 1; i <= startuplogs.length-1; i++) {
            	StartupLog startupLog = startuplogs[i];
            	
            	try {
            		JSONObject jsonObject = JSONObject.fromObject(startupLog);
            		String jsonstring = jsonObject.toString();
                    httpPost(url,  jsonstring);
                    System.err.println("正在发送数据：" + i + "::" + jsonstring);
                } catch (Exception ex) {
                    System.out.println(ex);
                }
            }
        } catch (Exception ex) {
        	ex.printStackTrace();
        }
	}
    
    
}
```

此文, 到此结束！





