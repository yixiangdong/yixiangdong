---
layout: ceshi7
title: 实时用户画像系统(二) 年代标签代码编写
date: 2020-03-25 12:14:15
tags: 机器学习
comments: true
---

### 年代标签定义

年代：40年代 50年代 60年代 70年代 80年代 90年代 00后 10后
统计每个年代群里的数量，做到近实时统计，每半小时会进行一次任务统计

### 年代标签代码编写

#### 设计思路

业务数据库->sqoop同步到->hdfs->flink读取readText->map转换每行数据设置count->reduce统计count

<!-- more --> 

```java
//时间工具类
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

public class DateUtils {
    public static String getYearbasebyAge(String age){
        Calendar cal = Calendar.getInstance();
        cal.setTime(new Date());
        cal.add(Calendar.YEAR,-Integer.valueOf(age));
        Date newdate = cal.getTime();
        DateFormat dateFormat = new SimpleDateFormat("yyyy");
        String newdatestr = dateFormat.format(newdate);
        Integer newdateinteger = Integer.valueOf(newdatestr);
        String yearbasetype = "";
        if(newdateinteger >= 1940 && newdateinteger < 1950){
            yearbasetype = "40年后";
        }else if (newdateinteger >= 1950 && newdateinteger < 1960){
            yearbasetype = "50年后";
        }else if (newdateinteger >= 1960 && newdateinteger < 1970){
            yearbasetype = "60年后";
       }else if (newdateinteger >= 1970 && newdateinteger < 1980){
            yearbasetype = "70年后";
        }else if (newdateinteger >= 1980 && newdateinteger < 1990){
            yearbasetype = "80年后";
        }else if (newdateinteger >= 1990 && newdateinteger < 2000){
            yearbasetype = "90年后";
        }else if (newdateinteger >= 2000 && newdateinteger < 2010){
            yearbasetype = "00年后";
        }else if (newdateinteger >= 2010 && newdateinteger < 2020){
            yearbasetype = "10年后";
        }
        return yearbasetype;
     }
}
```

### 年代数据保存

```java
//Hbase工具类
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.Map;
import java.util.Set;

public class HbaseUtil {
    private static Admin admin = null;
    private static Connection conn = null;
    static{
        // 创建hbase配置对象
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.rootdir","hdfs://192.168.37.141:9000/hbase");
        //使用eclipse时必须添加这个，否则无法定位
        conf.set("hbase.zookeeper.quorum","192.168.37.141");
        conf.set("hbase.client.scanner.timeout.period", "600000");
        conf.set("hbase.rpc.timeout", "600000");
        try {
            conn = ConnectionFactory.createConnection(conf);
            // 得到管理程序
            admin = conn.getAdmin();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    /**
     * 创建表 create "userflaginfo","baseinfo"
     */
    public void createTable(String tabName,String famliyname) throws Exception {
        HTableDescriptor tab = new HTableDescriptor(tabName);
        // 添加列族,每个表至少有一个列族.
        HColumnDescriptor colDesc = new HColumnDescriptor(famliyname);
        tab.addFamily(colDesc);
        // 创建表
        admin.createTable(tab);
        System.out.println("over");
    }

    /**
     * 插入数据，create "testinfo","time"
     */
    public static void put(String tablename, String rowkey, String famliyname, Map<String,String> datamap) throws Exception {
        Table table = conn.getTable(TableName.valueOf(tablename));
        // 将字符串转换成byte[]
        byte[] rowkeybyte = Bytes.toBytes(rowkey);
        Put put = new Put(rowkeybyte);
        if(datamap != null){
            Set<Map.Entry<String,String>> set = datamap.entrySet();
            for(Map.Entry<String,String> entry : set){
                String key = entry.getKey();
                Object value = entry.getValue();
                put.addColumn(Bytes.toBytes(famliyname), Bytes.toBytes(key), Bytes.toBytes(value+""));
            }
        }
        table.put(put);
        table.close();
        System.out.println("ok");
    }

    /**
     * 获取数据，create "testinfo","time"
     */
    public static String getdata(String tablename, String rowkey, String famliyname,String colum) throws Exception {
        Table table = conn.getTable(TableName.valueOf(tablename));
        // 将字符串转换成byte[]
        byte[] rowkeybyte = Bytes.toBytes(rowkey);
        Get get = new Get(rowkeybyte);
        Result result =table.get(get);
        byte[] resultbytes = result.getValue(famliyname.getBytes(),colum.getBytes());
        if(resultbytes == null){
            return null;
        }

        return new String(resultbytes);
    }

    /**
     * 插入数据，create "testinfo","time"
     */
    public static void putdata(String tablename, String rowkey, String famliyname,String colum,String data) throws Exception {
        Table table = conn.getTable(TableName.valueOf(tablename));
        Put put = new Put(rowkey.getBytes());
        put.addColumn(famliyname.getBytes(),colum.getBytes(),data.getBytes());
        table.put(put);
    }

    public static void main(String[] args) throws Exception {
        System.setProperty("hadoop.home.dir","E:\\dw\\hadoop-2.6.0-cdh5.13.0");
//        putdata("testinfo", "5", "time","test","etest");
        String string = getdata("testinfo", "5", "time","test");
        System.out.println(string);
    }


}

```



```java
//mongodb 工具类
import com.alibaba.fastjson.JSONObject;
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;
import org.bson.types.ObjectId;

/**
 * Created by li on 2019/1/5.
 */
public class MongoUtils {

    private static MongoClient mongoClient = new MongoClient("192.168.37.141",27017);



    public static Document findoneby(String tablename, String database,String yearbasetype){
        MongoDatabase mongoDatabase = mongoClient.getDatabase(database);
        MongoCollection mongoCollection = mongoDatabase.getCollection(tablename);
        Document  doc = new Document();
        doc.put("info", yearbasetype);
        FindIterable<Document> itrer = mongoCollection.find(doc);
        MongoCursor<Document> mongocursor = itrer.iterator();
        if(mongocursor.hasNext()){
            return mongocursor.next();
        }else{
            return null;
        }
    }


    public static void saveorupdatemongo(String tablename,String database,Document doc) {
        MongoDatabase mongoDatabase = mongoClient.getDatabase(database);
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
                System.out.println("come into saveorupdatemongo ---- update---"+ JSONObject.toJSONString(doc));
            } catch (Exception e) {
// TODO Auto-generated catch block
                e.printStackTrace();
            }
        }else{
            mongocollection.insertOne(doc);
            try {
                System.out.println("come into saveorupdatemongo ---- insert---"+JSONObject.toJSONString(doc));
            }catch (Exception e) {
// TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
    }
```

map数据处理代码:

```java
import com.youfan.entity.YearBase;
import com.youfan.util.DateUtils;
import com.youfan.util.HbaseUtil;
import org.apache.commons.lang3.StringUtils;
import org.apache.flink.api.common.functions.MapFunction;

public class YearBaseMap implements MapFunction<String,YearBase> {

    @Override
    public YearBase map(String s) throws Exception {
        if(StringUtils.isBlank(s)){
            return null;
        }
        //因为是存储在hdfs以","分隔
        String [] userinfos = s.split(",");
        String userid = userinfos[0];
        String username = userinfos[1];
        String sex = userinfos[2];
        String telphone = userinfos[3];
        String email = userinfos[4];
        String age = userinfos[5];
        String register = userinfos[6];
        String usetype = userinfos[7];
        String yearbasetype = DateUtils.getYearbasebyAge(age);
        String tablename = "userflahinfo";
        String rowkey = userid;
        String familyname = "baseinfo";
        String column = "yearbase";
        //把计算以前的结果插入数据到hbase
        HbaseUtil.putdata(tablename,rowkey,familyname,column,yearbasetype);
        YearBase  yearbase = new YearBase();
        String groupfield = "yearbase=="+yearbasetype;
        yearbase.setYeartype(yearbasetype);
        yearbase.setCount(1L);
        yearbase.setGroupfield(groupfield);

        return yearbase;
    }
}
```

reduce统计代码

```java
package com.youfan.reduce;

import com.youfan.entity.YearBase;
import org.apache.flink.api.common.functions.ReduceFunction;

public class YearBaseReduce implements ReduceFunction<YearBase> {


    @Override
    public YearBase reduce(YearBase yearBase, YearBase t1) throws Exception {
        String yeartype = yearBase.getYeartype();
        Long count1 = yearBase.getCount();
        Long count2 =t1.getCount();
        YearBase finalyearBase = new YearBase();
        finalyearBase.setYeartype(yeartype);
        finalyearBase.setCount(count1+count2);
        return finalyearBase;
    }
}
```

实体类

```java
package com.youfan.entity;

public class YearBase {
    private String yeartype; //
    private Long count;
    private String groupfield; //分组字段

    public String getGroupfield() {
        return groupfield;
    }

    public void setGroupfield(String groupfield) {
        this.groupfield = groupfield;
    }

    public String getYeartype() {
        return yeartype;
    }

    public void setYeartype(String yeartype) {
        this.yeartype = yeartype;
    }

    public Long getCount() {
        return count;
    }

    public void setCount(Long count) {
        this.count = count;
    }
}
```

Flink 数据批处理代码:

```java
package com.youfan.task;

import com.youfan.entity.YearBase;
import com.youfan.map.YearBaseMap;
import com.youfan.reduce.YearBaseReduce;
import com.youfan.util.MongoUtils;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.utils.ParameterTool;
import org.bson.Document;

import java.util.List;

public class YearBaseTask {public static void main(String[] args) {
    final ParameterTool params = ParameterTool.fromArgs(args);
    //set up the execution environment
    final ExecutionEnvironment env= ExecutionEnvironment.getExecutionEnvironment();
    //make parameters available in the web interface
    env.getConfig().setGlobalJobParameters(params);

    //get input data from hdfs path(mysql ->hdfs)
    DataSet<String> text=env.readTextFile(params.get("input"));
    DataSet<YearBase> mapresult = text.map(new YearBaseMap());
    //DataSet map = text.flatMap(null);
    DataSet<YearBase> reduce = mapresult.groupBy("groupbyField").reduce(new YearBaseReduce());
    try {
        List<YearBase> reduceresult = reduce.collect();
        //保存计算以后的结果到mongodb
        for(YearBase yearbase: reduceresult){
            String yeartype = yearbase.getYeartype();
            Long count = yearbase.getCount();
            Document doc = MongoUtils.findoneby("yearbasestatics","youfanPortrait",yeartype);
            if(doc == null){
             doc= new Document();
             doc.put("yearbasetype",yeartype);
             doc.put("count",count);
            }else{
                Long countpre =doc.getLong("count");
                Long total = countpre+count;
                doc.put("count",total);
            }
            MongoUtils.saveorupdatemongo("yearbasestatics","youfanPortrait",doc);
        }
        env.execute("test");
    } catch (Exception e) {
        e.printStackTrace();
    }
 }
}
```

