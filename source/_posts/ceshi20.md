---
layout: ceshi20
title: 实时用户画像系统(十五)flink分布式kmeans实现用户分群
date: 2020-03-29 22:49:56
tags: 机器学习
comments: true
---

### 添加实体类

<!--more-->

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Created by li on 2019/1/5.
 */
public class UserGroupInfo {
    private String userid;
    private String createtime;
    private String amount ;
    private String paytype ;
    private String paytime;
    private String paystatus;//0、未支付 1、已支付 2、已退款
    private String couponamount;
    private String totalamount;
    private String refundamount;
    private Long count;//数量
    private String producttypeid;//消费类目
    private String groupfield;//分组
    private List<UserGroupInfo> list;//一个用户所有的消费信息

    private double avramout;//平均消费金额
    private double maxamout;//消费最大金额
    private int days;//消费频次
    private Long buytype1;//消费类目1数量
    private Long buytype2;//消费类目2数量
    private Long buytype3;//消费类目3数量
    private Long buytime1;//消费时间点1数量
    private Long buytime2;//消费时间点2数量
    private Long buytime3;//消费时间点3数量
    private Long buytime4;//消费时间点4数量

    public String getUserid() {
        return userid;
    }

    public void setUserid(String userid) {
        this.userid = userid;
    }

    public String getCreatetime() {
        return createtime;
    }

    public void setCreatetime(String createtime) {
        this.createtime = createtime;
    }

    public String getAmount() {
        return amount;
    }

    public void setAmount(String amount) {
        this.amount = amount;
    }

    public String getPaytype() {
        return paytype;
    }

    public void setPaytype(String paytype) {
        this.paytype = paytype;
    }

    public String getPaytime() {
        return paytime;
    }

    public void setPaytime(String paytime) {
        this.paytime = paytime;
    }

    public String getPaystatus() {
        return paystatus;
    }

    public void setPaystatus(String paystatus) {
        this.paystatus = paystatus;
    }

    public String getCouponamount() {
        return couponamount;
    }

    public void setCouponamount(String couponamount) {
        this.couponamount = couponamount;
    }

    public String getTotalamount() {
        return totalamount;
    }

    public void setTotalamount(String totalamount) {
        this.totalamount = totalamount;
    }

    public String getRefundamount() {
        return refundamount;
    }

    public void setRefundamount(String refundamount) {
        this.refundamount = refundamount;
    }

    public Long getCount() {
        return count;
    }

    public void setCount(Long count) {
        this.count = count;
    }

    public String getProducttypeid() {
        return producttypeid;
    }

    public void setProducttypeid(String producttypeid) {
        this.producttypeid = producttypeid;
    }

    public String getGroupfield() {
        return groupfield;
    }

    public void setGroupfield(String groupfield) {
        this.groupfield = groupfield;
    }

    public List<UserGroupInfo> getList() {
        return list;
    }

    public void setList(List<UserGroupInfo> list) {
        this.list = list;
    }

    public double getAvramout() {
        return avramout;
    }

    public void setAvramout(double avramout) {
        this.avramout = avramout;
    }

    public double getMaxamout() {
        return maxamout;
    }

    public void setMaxamout(double maxamout) {
        this.maxamout = maxamout;
    }

    public int getDays() {
        return days;
    }

    public void setDays(int days) {
        this.days = days;
    }

    public Long getBuytype1() {
        return buytype1;
    }

    public void setBuytype1(Long buytype1) {
        this.buytype1 = buytype1;
    }

    public Long getBuytype2() {
        return buytype2;
    }

    public void setBuytype2(Long buytype2) {
        this.buytype2 = buytype2;
    }

    public Long getBuytype3() {
        return buytype3;
    }

    public void setBuytype3(Long buytype3) {
        this.buytype3 = buytype3;
    }

    public Long getBuytime1() {
        return buytime1;
    }

    public void setBuytime1(Long buytime1) {
        this.buytime1 = buytime1;
    }

    public Long getBuytime2() {
        return buytime2;
    }

    public void setBuytime2(Long buytime2) {
        this.buytime2 = buytime2;
    }

    public Long getBuytime3() {
        return buytime3;
    }

    public void setBuytime3(Long buytime3) {
        this.buytime3 = buytime3;
    }

    public Long getBuytime4() {
        return buytime4;
    }

    public void setBuytime4(Long buytime4) {
        this.buytime4 = buytime4;
    }
}

```

#### 添加map数据处理类

```java
import com.youfan.logic.LogicInfo;
import org.apache.commons.lang3.StringUtils;
import org.apache.flink.api.common.functions.MapFunction;

import java.util.Random;

/**
 * Created by li on 2019/1/5.
 */
public class KMeansMap implements MapFunction<String, KMeans>{
    @Override
    public KMeans map(String s) throws Exception {
        if(StringUtils.isBlank(s)){
            return null;
        }
        //2,3,4
        Random random = new Random();
        String [] temps = s.split(",");
        String variable1 = temps[0];
        String variable2 = temps[1];
        String variable3 = temps[2];
        KMeans kMeans = new KMeans();
        kMeans.setVariable1(variable1);
        kMeans.setVariable2(variable2);
        kMeans.setVariable3(variable3);
        kMeans.setGroupbyfield("logic=="+random.nextInt(10));
        return kMeans;
    }
}
```

#### 添加reduce统计计算类

```java
import com.youfan.logic.CreateDataSet;
import com.youfan.logic.LogicInfo;
import com.youfan.logic.Logistic;
import org.apache.flink.api.common.functions.GroupReduceFunction;
import org.apache.flink.util.Collector;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.Set;

/**
 * Created by li on 2019/1/6.
 */
public class KMeansReduce implements GroupReduceFunction<KMeans,ArrayList<Point>> {
    @Override
    public void reduce(Iterable<KMeans> iterable, Collector<ArrayList<Point>> collector) throws Exception {
        Iterator<KMeans> iterator = iterable.iterator();
        ArrayList<float[]> dataSet = new ArrayList<float[]>();
        while(iterator.hasNext()){
            KMeans kMeans = iterator.next();
            float[] f = new float[]{Float.valueOf(kMeans.getVariable1()),Float.valueOf(kMeans.getVariable2()),Float.valueOf(kMeans.getVariable3())};
            dataSet.add(f);
        }
        KMeansRun kRun =new KMeansRun(6, dataSet);

        Set<Cluster> clusterSet = kRun.run();
        ArrayList<Point> arrayList = new ArrayList<Point>();
        for(Cluster cluster:clusterSet){
            arrayList.add(cluster.getCenter());
        }
        collector.collect(arrayList);
    }
}
```

#### 添加Flink 计算类

```java
import com.youfan.logic.LogicInfo;
import com.youfan.logic.LogicMap;
import com.youfan.logic.LogicReduce;
import org.apache.flink.api.common.io.FileOutputFormat;
import org.apache.flink.api.common.io.OutputFormat;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.utils.ParameterTool;
import org.apache.flink.configuration.Configuration;

import java.io.IOException;
import java.util.*;

/**
 * Created by li on 2019/1/6.
 */
public class KMeansTask {
    public static void main(String[] args) {
        final ParameterTool params = ParameterTool.fromArgs(args);

        // set up the execution environment
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        // make parameters available in the web interface
        env.getConfig().setGlobalJobParameters(params);

        // get input data
        DataSet<String> text = env.readTextFile(params.get("input"));

        DataSet<KMeans> mapresult = text.map(new KMeansMap());
        DataSet<ArrayList<Point>> reduceresutl = mapresult.groupBy("groupbyfield").reduceGroup(new KMeansReduce());
        try {
            List<ArrayList<Point>> reusltlist = reduceresutl.collect();
            ArrayList<float[]> dataSet = new ArrayList<float[]>();
            for(ArrayList<Point> array:reusltlist){
                for(Point point:array){
                    dataSet.add(point.getlocalArray());
                }
            }
            KMeansRun kRun =new KMeansRun(6, dataSet);

            Set<Cluster> clusterSet = kRun.run();
            List<Point> finalClutercenter = new ArrayList<Point>();
            int count= 100;
            for(Cluster cluster:clusterSet){
                Point point = cluster.getCenter();
                point.setId(count++);
                finalClutercenter.add(point);
            }
            DataSet<Point> flinalMap = text.map(new KMeansFinalMap(finalClutercenter));
            flinalMap.writeAsText(params.get("out"));
            env.execute("KmeansTask analy");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 添加最终map类

```java
import com.youfan.logic.LogicInfo;
import com.youfan.logic.LogicMap;
import com.youfan.logic.LogicReduce;
import org.apache.flink.api.common.io.FileOutputFormat;
import org.apache.flink.api.common.io.OutputFormat;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.utils.ParameterTool;
import org.apache.flink.configuration.Configuration;

import java.io.IOException;
import java.util.*;

/**
 * Created by li on 2019/1/6.
 */
public class KMeansTask {
    public static void main(String[] args) {
        final ParameterTool params = ParameterTool.fromArgs(args);

        // set up the execution environment
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        // make parameters available in the web interface
        env.getConfig().setGlobalJobParameters(params);

        // get input data
        DataSet<String> text = env.readTextFile(params.get("input"));

        DataSet<KMeans> mapresult = text.map(new KMeansMap());
        DataSet<ArrayList<Point>> reduceresutl = mapresult.groupBy("groupbyfield").reduceGroup(new KMeansReduce());
        try {
            List<ArrayList<Point>> reusltlist = reduceresutl.collect();
            ArrayList<float[]> dataSet = new ArrayList<float[]>();
            for(ArrayList<Point> array:reusltlist){
                for(Point point:array){
                    dataSet.add(point.getlocalArray());
                }
            }
            KMeansRun kRun =new KMeansRun(6, dataSet);

            Set<Cluster> clusterSet = kRun.run();
            List<Point> finalClutercenter = new ArrayList<Point>();
            int count= 100;
            for(Cluster cluster:clusterSet){
                Point point = cluster.getCenter();
                point.setId(count++);
                finalClutercenter.add(point);
            }
            DataSet<Point> flinalMap = text.map(new KMeansFinalMap(finalClutercenter));
            flinalMap.writeAsText(params.get("out"));
            env.execute("KmeansTask analy");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

