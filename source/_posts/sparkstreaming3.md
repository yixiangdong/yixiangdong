---
layout: sparkstreaming3
title: SparkStreaming实时计算系统(三)之sparkstreaming整合mongo代码编写
date: 2020-04-02 02:32:09
tags: sparkstreaming
comments: true
---

### 本文的目的

处理并实时统计各个用户行为日志。

同时:

熟悉java编程中sparkstreaming整合mongodb的使用方法

熟悉java编程中sparkstreaming中map,flatmap和reduce算子的使用

<!--more-->

### 搭建maven项目SparkStreamingrevice

在pom.xml中添加依赖

```xml
<modelVersion>4.0.0</modelVersion>
  <groupId>com.test</groupId>
  <artifactId>SparkStreamingrevice</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  
  <properties>
    	<spark.version>1.4.1</spark.version>
    	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
  
  <dependencies>
  	<dependency>
            <groupId>jdk.tools</groupId>
            <artifactId>jdk.tools</artifactId>
            <version>1.8</version>
            <scope>system</scope>
            <systemPath>C:\Program Files\Java\jdk1.8.0_144/lib/tools.jar</systemPath>
       </dependency>
  	<dependency>
            <groupId>org.codehaus.jackson</groupId>
            <artifactId>jackson-mapper-asl</artifactId>
            <version>1.4.2</version>
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
		    <groupId>commons-io</groupId>
		    <artifactId>commons-io</artifactId>
		    <version>2.4</version>
		</dependency>
		<dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-core_2.10</artifactId>
      <version>${spark.version}</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-streaming_2.10</artifactId>
      <version>${spark.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-streaming-kafka_2.10</artifactId>
      <version>${spark.version}</version>
    </dependency>
     <dependency>
      <groupId>org.scala-lang</groupId>
      <artifactId>scala-library</artifactId>
      <version>2.10.4</version>
    </dependency>
    
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
  	<groupId>com.lidahai</groupId>
  	<artifactId>AppInfoCommon</artifactId>
  	<version>0.0.1-SNAPSHOT</version>
  </dependency>
  <dependency>
      <groupId>org.mongodb</groupId>
      <artifactId>mongodb-driver</artifactId>
      <version>3.0.1</version>
</dependency>
  </dependencies>
```

### 添加配置文件application.properties

```reStructuredText
zkaddress=192.168.37.141:2181
topicerror=apperrorspark
appstartupspark=appstartupspark
recievenums=1
mongoaddr=192.168.37.141
mongoport=27017
```

### 添加日志处理文件log4j

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="info" monitorInterval="300">

  <Appenders>
    <Console name="STDOUT">
      <PatternLayout pattern="%d %-5p [%t] %C{2} (%F:%L) - %m%n"/>
    </Console>
  </Appenders>
   
  <Loggers>
    <Root level="info">
      <AppenderRef ref="STDOUT"/>
    </Root>
    <Logger name="input" level="debug" additivity="false">
      <AppenderRef ref="STDOUT" />
    </Logger>
  </Loggers>

</Configuration>
```

创建dao层com.lidahai.dao并添加类ErrorDao

作用：联系mongo把错误日志保存到mongodb

```java
package com.lidahai.dao;

import java.io.IOException;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.bson.Document;
import org.bson.types.ObjectId;
import org.codehaus.jackson.JsonGenerationException;
import org.codehaus.jackson.map.JsonMappingException;
import org.codehaus.jackson.map.ObjectMapper;

import com.lidahai.Utils.PropertyRead;
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;


public class ErrorDao {
	private static Log log = LogFactory.getLog(ErrorDao.class);
	private MongoClient mongoClient = new MongoClient(PropertyRead.getkey("mongoaddr"),Integer.valueOf(PropertyRead.getkey("mongoport")));
	ObjectMapper objectMapper = new ObjectMapper();
	public ErrorDao(){
		log.info("ErrorDao is created!!");
		
		
	}
	
	public Document findoneby(String tablename,String appId,String appVersion,String appChannel,
			String appPlatform,String osType,
			String deviceStyle,String timeValue,String errorId){
		MongoDatabase  mongoDatabase = mongoClient.getDatabase(appId);
		MongoCollection mongoCollection = mongoDatabase.getCollection(tablename);
		Document  doc = new Document();
		doc.put("appId", appId);
		doc.put("appVersion", appVersion);
		doc.put("appChannel", appChannel);
		doc.put("appPlatform", appPlatform);
		doc.put("osType", osType);
		doc.put("deviceStyle", deviceStyle);
		doc.put("timeValue", timeValue);
		doc.put("errorId", errorId);
		FindIterable<Document> itrer = mongoCollection.find(doc);
		MongoCursor<Document> mongocursor = itrer.iterator();
		if(mongocursor.hasNext()){
			return mongocursor.next();
		}else{
			return null;
		}
	}
	

	public void saveorupdatemongo(String tablename, Document doc) {
		String appId = doc.getString("appId");
		MongoDatabase mongoDatabase = mongoClient.getDatabase(appId);
		MongoCollection<Document> mongocollection = mongoDatabase.getCollection(tablename);
		if(!doc.containsKey("_id")){
			ObjectId objectid = new ObjectId();
			doc.put("_id", objectid);
			 mongocollection.insertOne(doc);
			 return;
		}
		Document matchDocument = new Document();
		String objectid = doc.get("_id").toString();
		matchDocument.put("_id", new ObjectId(objectid));
		FindIterable<Document> findIterable =  mongocollection.find(matchDocument);
		if(findIterable.iterator().hasNext()){
			mongocollection.updateOne(matchDocument, new Document("$set",doc));
			try {
				log.info("come into saveorupdatemongo ---- update---"+objectMapper.writeValueAsString(doc));
			} catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}else{
			mongocollection.insertOne(doc);
			try {
			log.info("come into saveorupdatemongo ---- insert---"+objectMapper.writeValueAsString(doc));
			}catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	

//	public static void main(String[] args) {
//		log.info("test");
//	}
}

```

### 创建map层

作用:

包名: com.lidahai.service.map 添加错误日志处理类ErrorMapService传入一个list,输入错误日志类

所有的列组成一个key,并映射每一个key为1，方便后面的reduce统计1的个数。这个个数之和就是错误日志的条数

```java
package com.lidahai.service.map;

import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.List;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.codehaus.jackson.map.ObjectMapper;

import com.lidahai.logentity.AppEorrLog;
import com.lidahai.service.reduce.ErrorReduceService;
import com.lidahai.utils.Constant;

import scala.Tuple2;

public class ErrorMapService {
	
	private Log log = LogFactory.getLog(ErrorMapService.class);
	
	public ErrorMapService(){
		log.info(" come into ErrorMapService !!");
	}
	//这里的输入是AppEorrLog appEorrLog，输出是tuple2list
   public void processmap(List<Tuple2<String, Integer>> tuple2list,AppEorrLog appEorrLog ){
			Long createdAtMs = appEorrLog.getCreatedAtMs();//����ʱ��			
			String appId = appEorrLog.getAppId();//Ӧ��id	
			String appVersion = appEorrLog.getAppVersion();//�汾			
			String appChannel = appEorrLog.getAppChannel();//����			
			String appPlatform = appEorrLog.getAppPlatform();//ƽ̨		
			String osType = appEorrLog.getOsType();	//����ϵͳ			
			String deviceStyle = appEorrLog.getDeviceStyle();//�豸����			
			String errorBrief = appEorrLog.getErrorBrief();	//����ժҪ	
			String errorDetail = appEorrLog.getErrorDetail();//��������
			//����Ψһ��
			String errorId = errorDetail.substring(0, errorDetail.indexOf("\n")==-1?errorDetail.length():errorDetail.indexOf("\n"))+"=====>"+errorBrief;
			String key = "ErrorInfoDaily"+Constant.SPLITSTRING+"errorCnt"+Constant.SPLITSTRING+appId+
					Constant.SPLITSTRING+appVersion+Constant.SPLITSTRING+appChannel+Constant.SPLITSTRING+appPlatform+
					Constant.SPLITSTRING+osType+Constant.SPLITSTRING+deviceStyle+Constant.SPLITSTRING+errorId+Constant.SPLITSTRING+transfertime(createdAtMs);
			Tuple2<String,Integer> tuple2 = new Tuple2<String, Integer>(key, 1);
			tuple2list.add(tuple2);
		}
		//错误日志的时间
		private String transfertime(Long time){
			DateFormat dateFormat = new SimpleDateFormat("yyMMdd");
			String result =dateFormat.format(time);
			return result;
		}
}
```

### 创建reduce层

包名: com.lidahai.service.reduce

类名: ErrorReduceService

作用: 统计错误日志中错误发生的条数以及错误日志的详细信息

```java
package com.lidahai.service.reduce;

import java.io.IOException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.bson.Document;
import org.bson.types.ObjectId;
import org.codehaus.jackson.map.ObjectMapper;

import com.lidahai.analyentity.AppEorranaly;
import com.lidahai.dao.ErrorDao;
import com.lidahai.utils.Constant;

import scala.Tuple2;

public class ErrorReduceService {
	ObjectMapper objectmapper = new ObjectMapper();
	private Log log = LogFactory.getLog(ErrorReduceService.class);
	private ErrorDao erorDao = null;
	public ErrorReduceService(){
		log.info("create ReduceUtils");
		erorDao = new ErrorDao();
	}
	public void dosave(Map<String,Document> cachemap){
		Set<Map.Entry<String,Document>> sets = cachemap.entrySet();
		for(Map.Entry<String,Document> map:sets){
			String key = map.getKey();
			String tablename = key.split(Constant.SPLITSTRING)[0];
			Document doc = map.getValue();
			erorDao.saveorupdatemongo(tablename,doc);
		}
		
	}
	public void doReduceProcess(List<Tuple2<String, Integer>> collect) {
		
		Map<String,Document> cachemap = new HashMap<String,Document>();
		
		for(Tuple2<String, Integer> tuple:collect){
			String keys = tuple._1;
			if(!keys.startsWith("ErrorInfoDaily")){
				continue;
			}
			String[] splitvalues = keys.split(Constant.SPLITSTRING);
			String tablename = splitvalues[0];
			String fieldname = splitvalues[1];
			String appId = splitvalues[2];
			String appVersion = splitvalues[3];
			String appChannel = splitvalues[4];
			String appPlatform = splitvalues[5];
			String osType = splitvalues[6];
			String deviceStyle = splitvalues[7];
			String errorId = splitvalues[8];
			String timeValue = splitvalues[9];
			
			Document doc = null;
			String keytemp =  tablename + Constant.SPLITSTRING 
					+ appId + Constant.SPLITSTRING
					+appVersion +Constant.SPLITSTRING+ 
					appChannel+Constant.SPLITSTRING+
					appPlatform+Constant.SPLITSTRING+
					osType+Constant.SPLITSTRING+
					deviceStyle+Constant.SPLITSTRING+
					timeValue+Constant.SPLITSTRING+errorId;
			
			 if(cachemap.get(keytemp)!=null){
				 doc = cachemap.get(keytemp);
			 }else{
				 doc = erorDao.findoneby(tablename,appId,appVersion,appChannel,appPlatform,osType,deviceStyle,timeValue,errorId);
			 }
			 
			 if(doc == null){
				 AppEorranaly appEorranaly = new AppEorranaly();
				 appEorranaly.setTimeValue(timeValue);
				 appEorranaly.setAppChannel(appChannel);
				 appEorranaly.setAppId(appId);
				 appEorranaly.setAppPlatform(appPlatform);
				 appEorranaly.setDeviceStyle(deviceStyle);
				 appEorranaly.setOsType(osType);
				 appEorranaly.setErrorId(errorId);
				 appEorranaly.setAppVersion(appVersion);
				 try {
					String jsonstring =objectmapper.writeValueAsString(appEorranaly);
					doc = Document.parse(jsonstring);
					ObjectId objectid = new ObjectId();
					doc.put("_id", objectid);
				} catch (Exception e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				 
				 
			 }
			 
			 if(doc.getLong(fieldname)==null){
				 doc.put(fieldname, Long.valueOf(tuple._2+""));
			 }else{
				 Long sumpre = doc.getLong(fieldname);//֮ǰ��ֵ
				 sumpre = sumpre + Long.valueOf(tuple._2+"");
				 doc.put(fieldname, sumpre);
			 }

			 cachemap.put(keytemp, doc);
			 dosave(cachemap);
		}
		
		
	}

}
```

### 添加分隔符类

```java
package com.lidahai.utils;

public class Constant {
	public static final String SPLITSTRING = "####";//主要用于分隔key
}

```



#### 添加sparkstreaming计算类

作用聚合map和reduce的计算结果



```java
package com.test.spark;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFlatMapFunction;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaPairDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;
import org.codehaus.jackson.map.ObjectMapper;

import com.lidahai.Utils.PropertyRead;
import com.lidahai.logentity.AppEorrLog;
import com.lidahai.logentity.StartupLog;
import com.lidahai.service.map.BrandMapService;
import com.lidahai.service.map.CarrierMapService;
import com.lidahai.service.map.ChannelMapService;
import com.lidahai.service.map.DeviceStyleMapService;
import com.lidahai.service.map.ErrorMapService;
import com.lidahai.service.map.NetworkMapService;
import com.lidahai.service.map.OstypeMapService;
import com.lidahai.service.map.ScreenSizeMapService;
import com.lidahai.service.map.UserActiveMapService;
import com.lidahai.service.map.UserNewMapService;
import com.lidahai.service.map.UserStartupMapService;
import com.lidahai.service.map.VersionMapService;
import com.lidahai.service.reduce.BrandReduceService;
import com.lidahai.service.reduce.CarrierReduceService;
import com.lidahai.service.reduce.ChannelReduceService;
import com.lidahai.service.reduce.DeviceStyleReduceService;
import com.lidahai.service.reduce.ErrorReduceService;
import com.lidahai.service.reduce.NetworkReduceService;
import com.lidahai.service.reduce.OstypeReduceService;
import com.lidahai.service.reduce.ScreensizeReduceService;
import com.lidahai.service.reduce.UserActiveReduceService;
import com.lidahai.service.reduce.UserNewReduceService;
import com.lidahai.service.reduce.UserStartupReduceService;
import com.lidahai.service.reduce.VersionReduceService;

import scala.Tuple2;

public class SparktStreamingReicive {
	private final static String zkaddress = PropertyRead.getkey("zkaddress");
	private final static String groupid = "appgroupid1";
	private final static String topicerror = PropertyRead.getkey("topicerror");
	
	private final static String appstartupspark = PropertyRead.getkey("appstartupspark");
	
	private final static int recievenums = Integer.valueOf(PropertyRead.getkey("recievenums"));
	private final static ObjectMapper objectmaper = new ObjectMapper();
	
	private static ErrorMapService errorMapService = null;
	private static ErrorReduceService errorReduceService = null;
	

	
	private static void processmapresuce(JavaStreamingContext context,
			JavaPairDStream<String, String> javapairDstream2) {
		
		if(errorMapService == null){
			errorMapService = new ErrorMapService ();
		}
		if(errorReduceService == null){
			errorReduceService = new ErrorReduceService();
		}
		
		
		//transform
		JavaDStream<String> lines = javapairDstream2.map(new Function<Tuple2<String,String>, String>() {

			public String call(Tuple2<String, String> arg0) throws Exception {
				String temp = arg0._2;
				return temp;
			}
		});
		
		//transform
		JavaPairDStream<String, Integer> pairs = lines.mapPartitionsToPair(new PairFlatMapFunction<Iterator<String>, String, Integer>() {
			
			public Iterable<Tuple2<String, Integer>> call(Iterator<String> arg0) throws Exception {
				List<Tuple2<String, Integer>> tuple2list = new ArrayList<Tuple2<String, Integer>>();
				while(arg0.hasNext()){
					String line = arg0.next();
					//com.lidahai.logentity.AppEorrLog:{dasfd:dasfasdf,dasfads:dsafadsf}
					String[] splitss = line.split(":",2);
					String logtype = splitss[0];
					if("com.lidahai.logentity.AppEorrLog".equals(logtype)){
						AppEorrLog appErrorLog = (AppEorrLog) objectmaper.readValue(splitss[1], Class.forName(splitss[0]));
						errorMapService.processmap(tuple2list, appErrorLog);
					}
					
				}
				return tuple2list;
			}
		});
		
		JavaPairDStream<String, Integer> wordCounts  = pairs.reduceByKey(new Function2<Integer, Integer, Integer>() {
			
			public Integer call(Integer arg0, Integer arg1) throws Exception {
				// TODO Auto-generated method stub
				return arg0+arg1;
			}
		});
		
		
		wordCounts.foreach(new Function<JavaPairRDD<String,Integer>, Void>() {

			public Void call(JavaPairRDD<String, Integer> arg0) throws Exception {
				errorReduceService.doReduceProcess(arg0.collect());
				return null;
			}
		});
		
	}
	
	
	public static void main(String[] args) {
		SparkConf sparkconf = new SparkConf().setAppName("SparktStreamingReicive").setMaster("local[2]");
		JavaStreamingContext context = new JavaStreamingContext(sparkconf,Durations.seconds(10));
		Map<String,Integer> map = new HashMap<String,Integer>();
		map.put(topicerror, recievenums);
		map.put(appstartupspark, recievenums);
		map.put("productscanlog", 1);
		JavaPairDStream<String, String> javapairDstream = KafkaUtils.createStream(context,zkaddress , groupid, map);
		JavaPairDStream<String, String> javapairDstream2 = javapairDstream.repartition(2);
		processmapresuce(context,javapairDstream2);
		context.start();
		context.awaitTermination();
	}
}

```

此文到此借宿，接下来整合前端接口。

