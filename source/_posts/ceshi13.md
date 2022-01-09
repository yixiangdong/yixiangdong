---
layout: ceshi13
title: 实时用户画像系统(八) 测试接口以及kafka接收消息
date: 2020-03-27 14:36:26
tags: 机器学习
comments: true
---



### 使用postman模拟post请求

AttentionProductLog api接口:

http://127.0.0.1:8762/infolog/receivelog?receivelog=AttentionProductLog:{"product":2}

kafkaconsumer 端返回如下消息体,说明程序是ok的:

{"message":"{"operatortype\":0,\"productid\":2,\"producttypeid\":0,\"userid\":0,\"usetype\":0}","status":"success"}

<!-- more -->

