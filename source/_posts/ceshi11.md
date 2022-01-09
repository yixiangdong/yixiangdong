---
layout: ceshi11
title: 实时用户画像系统(六) 用户行为日志结构
date: 2020-03-27 11:45:39
tags: 机器学习
password: 2020-03-27!!
abstract: 该文章已加密, 请加博主微信号:books_111索要密码查看。
message: 该文章已加密, 请加博主微信号:books_111索要密码查看。
wrong_pass_message: 密码不正确，请重新输入！
wrong_hash_message: 文章不能被校验, 不过您还是能看看解密后的内容！
comments: true
---

### 用户行为定义

浏览商品行为：商品id 商品类别id 浏览时间、停留时间、用户id 终端类别,用户ip
收藏商品行为：商品id 商品类别id 操作时间、操作类型（收藏，取消）、用户id、终端类别、用户ip
购物车行为：商品id 商品类别id 、操作时间、操作类型（加入，删除）、用户id、终端类别、用户ip
关注商品:  商品id 商品类别id 操作时间、操作类型（关注，取消）、用户id、终端类别、用户ip

<!-- more -->

新建maven 模块,并加入依赖

```xml
 <dependencies>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.8.1</version>
        </dependency>
        <dependency>
            <groupId>com.typesafe</groupId>
            <artifactId>config</artifactId>
            <version>1.2.1</version>
        </dependency>

    </dependencies>
```

#### 浏览商品行为日志结构

```java
import java.io.Serializable;

/**
 * Created by li on 2019/1/6.
 */
public class ScanProductLog implements Serializable{
     private int productid;//商品id
     private int producttypeid;//商品类别id
     private String scantime;//浏览时间
     private String staytime;//停留时间
     private int userid;//用户id
     private int usetype;//终端类型：0、pc端；1、移动端；2、小程序端'
     private String ip;// 用户ip

    private String brand;//品牌

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public int getProductid() {
        return productid;
    }

    public void setProductid(int productid) {
        this.productid = productid;
    }

    public int getProducttypeid() {
        return producttypeid;
    }

    public void setProducttypeid(int producttypeid) {
        this.producttypeid = producttypeid;
    }

    public String getScantime() {
        return scantime;
    }

    public void setScantime(String scantime) {
        this.scantime = scantime;
    }

    public String getStaytime() {
        return staytime;
    }

    public void setStaytime(String staytime) {
        this.staytime = staytime;
    }

    public int getUserid() {
        return userid;
    }

    public void setUserid(int userid) {
        this.userid = userid;
    }

    public int getUsetype() {
        return usetype;
    }

    public void setUsetype(int usetype) {
        this.usetype = usetype;
    }

    public String getIp() {
        return ip;
    }

    public void setIp(String ip) {
        this.ip = ip;
    }
}
```

#### 收藏商品行为日志结构

```java
import java.io.Serializable;
/**
 * Created by li on 2019/1/6.
 */
public class CollectProductLog implements Serializable {
    private int productid;//商品id
    private int producttypeid;//商品类别id
    private String opertortime;//操作时间
    private int opertortype;//操作类型，0、收藏，1、取消
    private int userid;//用户id
    private int usetype;//终端类型：0、pc端；1、移动端；2、小程序端'
    private String ip;// 用户ip

    private String brand;//品牌

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public int getProductid() {
        return productid;
    }

    public void setProductid(int productid) {
        this.productid = productid;
    }

    public int getProducttypeid() {
        return producttypeid;
    }

    public void setProducttypeid(int producttypeid) {
        this.producttypeid = producttypeid;
    }

    public String getOpertortime() {
        return opertortime;
    }

    public void setOpertortime(String opertortime) {
        this.opertortime = opertortime;
    }

    public int getOpertortype() {
        return opertortype;
    }

    public void setOpertortype(int opertortype) {
        this.opertortype = opertortype;
    }

    public int getUserid() {
        return userid;
    }

    public void setUserid(int userid) {
        this.userid = userid;
    }

    public int getUsetype() {
        return usetype;
    }

    public void setUsetype(int usetype) {
        this.usetype = usetype;
    }

    public String getIp() {
        return ip;
    }

    public void setIp(String ip) {
        this.ip = ip;
    }
}

```

#### 购物车行为日志结构

```java
import java.io.Serializable;

/**
 * Created by li on 2019/1/6.
 */
public class BuyCartProductLog implements Serializable{
     private int productid;//商品id
     private int producttypeid;//商品类别id
     private String operatortime;//操作时间
     private int operatortype;//操作类型 0、加入，1、删除
     private int userid;//用户id
     private int usetype;//终端类型：0、pc端；1、移动端；2、小程序端'
     private String ip;// 用户ip

    private String brand;//品牌

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public int getProductid() {
        return productid;
    }

    public void setProductid(int productid) {
        this.productid = productid;
    }

    public int getProducttypeid() {
        return producttypeid;
    }

    public void setProducttypeid(int producttypeid) {
        this.producttypeid = producttypeid;
    }

    public String getOperatortime() {
        return operatortime;
    }

    public void setOperatortime(String operatortime) {
        this.operatortime = operatortime;
    }

    public int getOperatortype() {
        return operatortype;
    }

    public void setOperatortype(int operatortype) {
        this.operatortype = operatortype;
    }

    public int getUserid() {
        return userid;
    }

    public void setUserid(int userid) {
        this.userid = userid;
    }

    public int getUsetype() {
        return usetype;
    }

    public void setUsetype(int usetype) {
        this.usetype = usetype;
    }

    public String getIp() {
        return ip;
    }

    public void setIp(String ip) {
        this.ip = ip;
    }
}

```

#### 关注商品日志结构

```java
import java.io.Serializable;

/**
 * Created by li on 2019/1/6. 关注商品
 */
public class AttentionProductLog implements Serializable{
     private int productid;//商品id
     private int producttypeid;//商品类别id
     private String opertortime;//操作时间
     private int operatortype;//操作类型，0、关注，1、取消
     private String staytime;//停留时间
     private int userid;//用户id
     private int usetype;//终端类型：0、pc端；1、移动端；2、小程序端'
     private String ip;// 用户ip
     private String brand;//品牌

    public int getProductid() {
        return productid;
    }

    public void setProductid(int productid) {
        this.productid = productid;
    }

    public int getProducttypeid() {
        return producttypeid;
    }

    public void setProducttypeid(int producttypeid) {
        this.producttypeid = producttypeid;
    }

    public String getOpertortime() {
        return opertortime;
    }

    public void setOpertortime(String opertortime) {
        this.opertortime = opertortime;
    }

    public int getOperatortype() {
        return operatortype;
    }

    public void setOperatortype(int operatortype) {
        this.operatortype = operatortype;
    }

    public String getStaytime() {
        return staytime;
    }

    public void setStaytime(String staytime) {
        this.staytime = staytime;
    }

    public int getUserid() {
        return userid;
    }

    public void setUserid(int userid) {
        this.userid = userid;
    }

    public int getUsetype() {
        return usetype;
    }

    public void setUsetype(int usetype) {
        this.usetype = usetype;
    }

    public String getIp() {
        return ip;
    }

    public void setIp(String ip) {
        this.ip = ip;
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }
}

```

