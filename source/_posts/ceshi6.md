---
layout: ceshi6
title: 实时用户画像系统(一)业务说明以及环境搭建
date: 2020-03-23 01:15:56
tags: 机器学习
comments: true
---
## 实时用户画像系统

### 系统架构

![系统架构](1584897466155.png)

![系统架构](1584897595841.png)

<!-- more -->

### 系统架构以及来源

![系统架构以及来源](1584897673317.png)

### 数据说明

![数据说明](1584897722504.png)

基本信息:

用户表: 用户ID, 用户名,密码,性别，手机号，邮箱，年龄，户籍身份，身份证编码，注册时间，收货地址，终端类型。

用户表额外信息:

学历，收入，职业，婚姻, 是否有小孩，是否有房，电话的牌子

### 建库建表语句

```sql
CREATE DATABASE `youfanportrait`;

CREATE TABLE `youfanportrait`.`userinfo` (
  `userid` INT NOT NULL AUTO_INCREMENT,
  `username` VARCHAR(50) NULL,
  `password` VARCHAR(50) NULL,
  `sex` INT NULL,
  `telphone` VARCHAR(50) NULL,
  `email` VARCHAR(50) NULL,
  `age` INT NULL,
  `idCard` VARCHAR(50) NULL,
  `registertime` TIMESTAMP(0) NULL,
  `usertype` INT NULL,
  PRIMARY KEY (`userid`));

CREATE TABLE `userinfodetail` (
  `userdetailid` int(11) NOT NULL AUTO_INCREMENT,
  `userid` int(11) DEFAULT NULL,
  `edu` int(11) DEFAULT NULL,
  `profession` varchar(20) COLLATE utf8_bin DEFAULT NULL,
  `marriage` int(11) DEFAULT NULL,
  `haschild` int(11) DEFAULT NULL,
  `hascar` int(11) DEFAULT NULL,
  `hashourse` int(11) DEFAULT NULL,
  `telphonebrand` varchar(50) COLLATE utf8_bin DEFAULT NULL,
  PRIMARY KEY (`userdetailid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
```

### idea新建maven项目

新建maven项目后加入依赖:

```xml
    <parent>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-examples</artifactId>
        <version>1.7.0</version>
    </parent>

    <properties>
        <scala.version>2.11.12</scala.version>
        <scala.binary.version>2.11</scala.binary.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_2.11</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
```

搭建hadoop环境(本机是cdh5.13.0虚拟机环境)，搭建细节不在这里详细解说了，大家可以自己下载CDH VM

![cdh5.13.0虚拟机环境](1585107206803.png)

同时搭建单机的mongodb，flink1.9环境(这些需要手动安装)

![mongodb](1585107353216.png)

![flink 运行web端界面](1585107602547.png)