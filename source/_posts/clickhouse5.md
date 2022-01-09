---
layout: clickhouse5
title: Flink+ClickHouse电商实时数据分析系统之ClickHouse初识以及环境安装,常用操作和java使用
date: 2020-04-06 23:42:28
tags: ClickHouse
comments: true
---

### 概述

clickhouse 是俄罗斯的“百度”Yandex公司在2016年开源的，一款针对大数据实时分析的高性能分布式数据库，与之对应的有hadoop生态hive，Vertica和百度出品的palo。**这是战斗民族继nginx后，又开源的一款“核武器”。**

ClickHouse作为分析型数据库，有三大特点：一是跑分快，二是功能多，三是文艺范。

### 背景

提到大数据不得不提 Hadoop，当下的 Hadoop 已不仅仅是当初的HDFS + MR(MapReduce) 这么简单。基于 Hadoop 而衍生的 Hive、Pig、Spark、Presto、Impala 等一系列组件共同构成了 Hadoop 生态体系。Hadoop 生态为今天的大数据领域提供着稳定可靠的数据服务。

Hadoop 生态体系解决了大数据界的大部分问题，当然其也存在缺点。Hadoop 体系的最大短板在于数据处理时效性。基于 Hadoop 生态的数据处理场景大部分对时效要求不高，按照传统的做法一般是 T + 1 的数据时效。即 Trade + 1，数据产出在交易日 + 1 天。

ClickHouse 的产生就是为了解决大数据量处理的时效性。**独立于Hadoop生态圈**。

<!--more-->

### 特性

采用列式存储
数据压缩
CPU 利用率高，在计算时会使用机器上的所有 CPU 资源
支持分片，并且同一个计算任务会在不同分片上并行执行，计算完成后会将结果汇总
支持SQL，SQL 几乎成了大数据的标准工具，使用门槛较低
支持联表查询

支持实时更新
自动多副本同步
支持索引
分布式存储查询

### 性能

低延迟：对于数据量（几千行，列不是很多）不是很大的短查询，如果数据已经被载入缓存，且使用主码，延迟在50MS左右。
并发量：虽然 ClickHouse 是一种在线分析型数据库，也可支持一定的并发。当单个查询比较短时，官方建议100 Queries / second。

写入速度：在使用 MergeTree 引擎的情况下，写入速度大概是 50 - 200M / s，如果按照 1K一条记录来算，大约每秒可写入50000 ~ 200000条记录每秒。如果每条记录比较小的话写入速度会更快。
1亿条数据的文件2.2GB，导入Clickhouse用时24秒
查询统计时，用时0.185秒

### 下载

下载安装包

Rpm包下载
http://repo.red-soft.biz/repos/clickhouse/stable/el7/

### 检查是否支持SSE 4.2

需要确保您使用的是x86_64处理器构架的Linux并且支持SSE 4.2指令集
grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported“
返回结果应为：“SSE 4.2 supported”

### 安装

rpm -ivh clickhouse-server-common-1.1.54236-4.el7.x86_64.rpm
rpm -ivh clickhouse-server-1.1.54236-4.el7.x86_64.rpm

缺少libltdl包和libodbc包，下载对应系统版本的依赖包
libtool-ltdl-2.4.2-21.el7_2.x86_64.rpm
unixODBC-2.3.1-11.el7.x86_64.rpm

### 安装server
rpm -ivh clickhouse-server-1.1.54236-4.el7.x86_64.rpm
rpm -ivh clickhouse-debuginfo-1.1.54236-4.el7.x86_64.rpm
rpm -ivh clickhouse-client-1.1.54236-4.el7.x86_64.rpm
rpm -ivh clickhouse-compressor-1.1.54236-4.el7.x86_64.rpm

### 修改配置文件

clickhouse-server配置文件目录
cd /etc/clickhouse-server/
config.xml配置相应的IP地址(《listen host》)
users.xml(配置相应的IP地址)(<networks><ip>)

### 启动服务

clickhouse-server --config-file=/etc/clickhouse-server/config.xml

### client链接测试

clickhouse-client --host=xx.xx.xx.xx  --port=1234

### ClickHouse  数据类型以及常用操作

#### 数据类型

UInt8，UInt16，UInt32，UInt64，Int8，Int16，Int32，Int64
固定长度的整数，带或不带标志。

Float32, Float64

就像java语言中的“float”和“double”一样

字符串

任意长度的字符串。长度不限。该值可以包含任意字节集，包括空字节。String类型替换了其他DBMS类型的VARCHAR，BLOB，CLOB和其他类型。

Date

一个Date。以1970-01-01（无符号）以来的天数存储在两个字节中

DateTime

日期与时间。以四个字节存储为Unix时间戳（无符号）。允许将值存储在与日期类型相同的范围内。最小值输出为0000-00-00 00:00:00。时间储存精度高达1秒（不闰秒）。

建库

CREATE DATABASE  test ENGINE = Ordinary;

创建表

CREATE TABLE test02( id UInt16,col1 String,col2 String,create_date date ) ENGINE = MergeTree(create_date, (id), 8192);
CREATE TABLE youfantest( id UInt16,name String,create_date Date ) ENGINE = MergeTree(create_date, (id), 8192);

插入数据

#方式1-交互式 INSERT INTO [db.]table [(c1, c2, c3)] VALUES (v11, v12, v13), ... INSERT INTO [db.]table [(c1, c2, c3)] SELECT ...
 #方式2-批量 cat file.csv | clickhouse-client --database=test --query="INSERT   INTO test FORMAT CSV" 
insert into ontime_all (FlightDate,Year)values('2001-10-12',2001);

查询数据

SELECT [DISTINCT] expr_list [FROM [db.]table | (subquery) | table_function] [FINAL] [SAMPLE sample_coeff] [ARRAY JOIN ...] [GLOBAL] ANY|ALL INNER|LEFT JOIN (subquery)|table USING columns_list [PREWHERE expr] [WHERE expr] [GROUP BY expr_list] [WITH TOTALS] [HAVING expr] [ORDER BY expr_list] [LIMIT n BY columns] [LIMIT [n, ]m] [UNION ALL ...] [INTO OUTFILE filename] [FORMAT format]

### ClickHouse   java使用

#### 插入数据

insert into youfantest (id,name,create_date)values(1,'xiaobai','2018-09-07');

insert into youfantest (id,name,create_date)values(2,'xiaohong','2018-09-08');

insert into youfantest (id,name,create_date)values(3,'xiaohei','2018-09-09');

CREATE TABLE youfantest( id UInt16,name String,create_date Date ) ENGINE = MergeTree(create_date, (id), 8192);

#### 添加依赖

``` xml
<dependencies>
 <dependency>
 <groupId>ru.yandex.clickhouse</groupId>
 <artifactId>clickhouse-jdbc</artifactId>
 <version>0.1.40</version>
 </dependency>

 <dependency>
 <groupId>org.apache.logging.log4j</groupId>
 <artifactId>log4j-slf4j-impl</artifactId>
 <version>2.11.0</version>
 </dependency>
</dependencies>

```

```java
String sql = "INSERT INTO student01 (sno,sname) VALUES(?,?,?,?,?,?,?)";		//插入sql语句
		try {
			ps = conn.prepareStatement(sql);
			
						ps.setString(1, stu.getSno());
			ps.setString(2, stu.getSname());
			
			ps.executeUpdate();			//执行sql语句

resultSet = preparedStatement.executeQuery(); while(resultSet.next()){

```

```java
public static void exeSql(String sql){
 
 String address = "jdbc:clickhouse://192.168.1:8123/test";
 
 Connection connection = null;
 
 Statement statement = null;
 
 ResultSet results = null;
 
 try {
 
 Class.forName("ru.yandex.clickhouse.ClickHouseDriver");
 
 connection = DriverManager.getConnection(address);
 
 statement = connection.createStatement();
 
 long begin = System.currentTimeMillis();
 
 results = statement.executeQuery(sql);
 
 long end = System.currentTimeMillis();
 
 System.out.println("执行（"+sql+"）耗时："+(end-begin)+"ms");
 
 ResultSetMetaData rsmd = results.getMetaData();
 
 List<Map> list = new ArrayList();
 
 while(results.next()){
 
 Map map = new HashMap();
 
 for(int i = 1;i<=rsmd.getColumnCount();i++){
 
 map.put(rsmd.getColumnName(i),results.getString(rsmd.getColumnName(i)));
 
 }
 
 list.add(map);
 
 }
 
 for(Map map : list){
 
 System.err.println(map);
 
 }
 
 } catch (Exception e) {
 
 e.printStackTrace();
 
 }finally {//关闭连接
 
 try {
 
 if(results!=null){
 
 results.close();
 
 }
 
 if(statement!=null){
 
 statement.close();
 
 }
 
 if(connection!=null){
 
 connection.close();
 
 }
 
 } catch (SQLException e) {
 
 e.printStackTrace();
 
 }
 
 }
 
 }

```

添加Log4j2.xml

```xml
<xml version="1.0" encoding="UTF-8">
 
<Configuration status="WARN">
 
 
 
 <!--全局参数-->
 
 <Properties>
 
 <Property name="pattern">%d{yyyy-MM-dd HH:mm:ss,SSS} %5p %c{1}:%L - %m%n</Property>
 
 <Property name="logDir">/data/logs/dust-server</Property>
 
 </Properties>
 
 
 
 <Loggers>
 
 <Root level="INFO">
 
 <AppenderRef ref="console"/>
 
 <AppenderRef ref="rolling_file"/>
 
 </Root>
 
 </Loggers>
 
 
 
 <Appenders>
 
 <!-- 定义输出到控制台 -->
 
 <Console name="console" target="SYSTEM_OUT" follow="true">
 
 <!--控制台只输出level及以上级别的信息-->
 
 <ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY"/>
 
 <PatternLayout>
 
 <Pattern>${pattern}</Pattern>
 
 </PatternLayout>
 
 </Console>
 
 <!-- 同一来源的Appender可以定义多个RollingFile，定义按天存储日志 -->
 
 <RollingFile name="rolling_file"
 
 fileName="${logDir}/dust-server.log"
 
 filePattern="${logDir}/dust-server_%d{yyyy-MM-dd}.log">
 
 <ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY"/>
 
 <PatternLayout>
 
 <Pattern>${pattern}</Pattern>
 
 </PatternLayout>
 
 <Policies>
 
 <TimeBasedTriggeringPolicy interval="1"/>
 
 </Policies>
 
 <!-- 日志保留策略，配置只保留七天 -->
 
 <DefaultRolloverStrategy>
 
 <Delete basePath="${logDir}/" maxDepth="1">
 
 <IfFileName glob="dust-server_*.log" />
 
 <IfLastModified age="7d" />
 
 </Delete>
 
 </DefaultRolloverStrategy>
 
 </RollingFile>
 
 </Appenders>
 
</Configuration>

```

