---
layout: ceshi27
title: 实时用户画像系统(二十三)前端查询服务构建2-RPC远程过程调用Foreign
date: 2020-03-31 13:02:56
tags: 机器学习
comments: true
---

前端查询服务构建2-RPC远程调用的方法

Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，**只需要创建一个接口并注解**。它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。ribbon是一个负载均衡客户端 类似nginx反向代理，可以很好的控制htt和tcp的一些行为

简而言之：

·Feign 采用的是基于接口的注解

·Feign 整合了ribbon

<!--more-->

##### 在common模块添加实体类

```java
/**
 * Created by li on 2019/1/19.
 */
public class AnalyResult {
    private String info;//分组条件
    private Long count;//总数

    public String getInfo() {
        return info;
    }

    public void setInfo(String info) {
        this.info = info;
    }

    public Long getCount() {
        return count;
    }

    public void setCount(Long count) {
        this.count = count;
    }
}

```

```java
import java.util.List;

/**
 * Created by li on 2019/1/19.
 */
public class ViewResultAnaly {
    private List<String> infolist;//分组list，x轴的值
    private List<Long> countlist;//数量list
    private String result;
    private String typename;//标签类型名称
    private String lablevalue;//标签类型对应的值

    private  List<ViewResultAnaly> list;//所有标签信息

    public List<ViewResultAnaly> getList() {
        return list;
    }

    public void setList(List<ViewResultAnaly> list) {
        this.list = list;
    }

    public String getTypename() {
        return typename;
    }

    public void setTypename(String typename) {
        this.typename = typename;
    }

    public String getLablevalue() {
        return lablevalue;
    }

    public void setLablevalue(String lablevalue) {
        this.lablevalue = lablevalue;
    }

    public String getResult() {
        return result;
    }

    public void setResult(String result) {
        this.result = result;
    }

    public List<String> getInfolist() {
        return infolist;
    }

    public void setInfolist(List<String> infolist) {
        this.infolist = infolist;
    }

    public List<Long> getCountlist() {
        return countlist;
    }

    public void setCountlist(List<Long> countlist) {
        this.countlist = countlist;
    }
}

```

##### 添加control 接口类

```java
import com.alibaba.fastjson.JSONObject;
import com.youfan.entity.AnalyResult;
import com.youfan.entity.ViewResultAnaly;
import com.youfan.form.AnalyForm;
import com.youfan.service.MongoDataService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by li on 2019/1/19.
 */
@RestController
@RequestMapping("mongoData")
@CrossOrigin
public class MongoDataViewControl {

    @Autowired
    MongoDataService mongoDataService;

    @RequestMapping(value = "resultinfoView",method = RequestMethod.POST,produces = "application/json;charset=UTF-8")
    public String resultinfoView(@RequestBody AnalyForm analyForm){
        String type = analyForm.getType();
        List<AnalyResult> list = new ArrayList<AnalyResult>();
        if("yearBase".equals(type)){
            list = mongoDataService.searchYearBase();
        }else if ("useType".equals(type)){
            list = mongoDataService.searchUseType();
        }else if ("email".equals(type)){
            list = mongoDataService.searchEmail();
        }else if ("consumptionlevel".equals(type)){
            list = mongoDataService.searchConsumptionlevel();
        }else if ("carrier".equals(type)){
            list = mongoDataService.searchCarrier();
        }else if ("chaoManAndWomen".equals(type)){
            list = mongoDataService.searchChaoManAndWomen();
        }else if ("brandlike".equals(type)){
            list = mongoDataService.searchBrandlike();
        }
        ViewResultAnaly viewResultAnaly = new ViewResultAnaly();
        List<String> infolist = new ArrayList<String>();//分组list，x轴的值
        List<Long> countlist =new ArrayList<Long>();//数量
        for(AnalyResult analyResult:list){
            infolist.add(analyResult.getInfo());
            countlist.add(analyResult.getCount());
        }
        viewResultAnaly.setInfolist(infolist);
        viewResultAnaly.setCountlist(countlist);
        String result = JSONObject.toJSONString(viewResultAnaly);
        return result;
    }
}
```

##### 添加form类

```java
/**
 * Created by li on 2019/1/20.
 */
public class AnalyForm {
    private String type;
    private String userid;

    public String getUserid() {
        return userid;
    }

    public void setUserid(String userid) {
        this.userid = userid;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }
} 
```

##### 添加service层

```java
import com.youfan.entity.AnalyResult;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import java.util.List;

/**
 * Created by li on 2019/1/19.
 */
@FeignClient(value = "youfanSearchInfo")
public interface MongoDataService {

    @RequestMapping(value = "yearBase/searchYearBase",method = RequestMethod.POST)
    public List<AnalyResult> searchYearBase();

    @RequestMapping(value = "yearBase/searchUseType",method = RequestMethod.POST)
    public List<AnalyResult> searchUseType();

    @RequestMapping(value = "yearBase/searchEmail",method = RequestMethod.POST)
    public List<AnalyResult> searchEmail();

    @RequestMapping(value = "yearBase/searchConsumptionlevel",method = RequestMethod.POST)
    public List<AnalyResult> searchConsumptionlevel();

    @RequestMapping(value = "yearBase/searchChaoManAndWomen",method = RequestMethod.POST)
    public List<AnalyResult> searchChaoManAndWomen();

    @RequestMapping(value = "yearBase/searchCarrier",method = RequestMethod.POST)
    public List<AnalyResult> searchCarrier();

    @RequestMapping(value = "yearBase/searchBrandlike",method = RequestMethod.POST)
    public List<AnalyResult> searchBrandlike();
}
```

##### 添加启动类

```java
import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
        import org.springframework.boot.autoconfigure.SpringBootApplication;
        import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
        import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
        import org.springframework.cloud.openfeign.EnableFeignClients;

/**
 * Created by li on 2019/1/6.
 */
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
@EnableDiscoveryClient
public class Startupmain {
    public static void main(String[] args) {

        SpringApplication.run( Startupmain.class, args );
    }
}
```

