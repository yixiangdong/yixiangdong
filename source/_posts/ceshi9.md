---
layout: ceshi9
title: 实时用户画像系统(四) 败家指数
date: 2020-03-26 14:51:29
tags: 机器学习
comments: true
---

### 数据库建表

```sql
CREATE TABLE `youfanportrait`.`productinfo` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `producttypeid` INT NULL,
  `productname` VARCHAR(45) NULL,
  `productdesc` VARCHAR(45) NULL,
  `price` INT NULL,
  `num` INT NULL,
  `createtime` TIMESTAMP NULL,
  `updatetime` TIMESTAMP NULL,
  `mecharid` INT NULL,
  `producturl` VARCHAR(45) NULL,
  PRIMARY KEY (`id`));
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
  
CREATE TABLE `orderinfo` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `productid` int(11) DEFAULT NULL,
  `producttypeid` int(11) DEFAULT NULL,
  `createtime` timestamp NULL DEFAULT NULL,
  `amount` double DEFAULT NULL,
  `paytype` int(11) DEFAULT NULL,
  `paytime` timestamp NULL DEFAULT NULL,
  `paystatus` int(11) DEFAULT NULL,
  `couponamount` double DEFAULT NULL,
  `totalamount` double DEFAULT NULL,
  `refundamount` double DEFAULT NULL,
  `num` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin

```

### 败家指数定义

败家指数 = 支付金额平均值x0.3、最大支付金额x0.3、下单频率x0.4

<!-- more -->

### 实体定义

```java
import java.util.List;

/**
 * Created by li on 2019/1/5.
 */
public class BaiJiaInfo {
    private String baijiatype;//败家指数区段：0-20 、20-50 、50-70、70-80、80-90、90-100
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
    private String groupfield;//分组

    private List<BaiJiaInfo> list;

    public List<BaiJiaInfo> getList() {
        return list;
    }

    public void setList(List<BaiJiaInfo> list) {
        this.list = list;
    }

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

    public String getBaijiatype() {
        return baijiatype;
    }

    public void setBaijiatype(String baijiatype) {
        this.baijiatype = baijiatype;
    }

    public Long getCount() {
        return count;
    }

    public void setCount(Long count) {
        this.count = count;
    }

    public String getGroupfield() {
        return groupfield;
    }

    public void setGroupfield(String groupfield) {
        this.groupfield = groupfield;
    }
}

```

### 添加map类

```java

import com.youfan.entity.BaiJiaInfo;
import com.youfan.entity.CarrierInfo;
import com.youfan.util.CarrierUtils;
import com.youfan.util.HbaseUtil;
import org.apache.commons.lang3.StringUtils;
import org.apache.flink.api.common.functions.MapFunction;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by li on 2019/1/5.
 */
public class BaijiaMap implements MapFunction<String, BaiJiaInfo>{
    @Override
    public BaiJiaInfo map(String s) throws Exception {
        if(StringUtils.isBlank(s)){
            return null;
        }

        String[] orderinfos = s.split(",");
        String id= orderinfos[0];
        String productid = orderinfos[1];
        String producttypeid = orderinfos[2];
        String createtime = orderinfos[3];
        String amount = orderinfos[4];
        String paytype = orderinfos[5];
        String paytime = orderinfos[6];
        String paystatus = orderinfos[7];
        String couponamount = orderinfos[8];
        String totalamount = orderinfos[9];
        String refundamount = orderinfos[10];
        String num = orderinfos[11];
        String userid = orderinfos[12];


        BaiJiaInfo baiJiaInfo = new BaiJiaInfo();
        baiJiaInfo.setUserid(userid);
        baiJiaInfo.setCreatetime(createtime);
        baiJiaInfo.setAmount(amount);
        baiJiaInfo.setPaytype(paytype);
        baiJiaInfo.setPaytime(paytime);
        baiJiaInfo.setPaystatus(paystatus);
        baiJiaInfo.setCouponamount(couponamount);
        baiJiaInfo.setTotalamount(totalamount);
        baiJiaInfo.setRefundamount(refundamount);
        String groupfield = "baijia=="+userid;
        baiJiaInfo.setGroupfield(groupfield);
        List<BaiJiaInfo> list = new ArrayList<BaiJiaInfo>();
        list.add(baiJiaInfo);
        return baiJiaInfo;
    }
}

```

添加reduce类

```java
import com.youfan.entity.BaiJiaInfo;
import com.youfan.entity.CarrierInfo;
import org.apache.flink.api.common.functions.GroupReduceFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.util.Collector;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by li on 2019/1/5.
 */
public class BaijiaReduce implements ReduceFunction<BaiJiaInfo>{


    @Override
    public BaiJiaInfo reduce(BaiJiaInfo baiJiaInfo, BaiJiaInfo t1) throws Exception {
        String userid = baiJiaInfo.getUserid();
        List<BaiJiaInfo> baijialist1 = baiJiaInfo.getList();
        List<BaiJiaInfo> baijialist2 = t1.getList();
        List<BaiJiaInfo> finallist = new ArrayList<BaiJiaInfo>();
        finallist.addAll(baijialist1);
        finallist.addAll(baijialist2);

        BaiJiaInfo baiJiaInfofinal = new BaiJiaInfo();
        baiJiaInfofinal.setUserid(userid);
        baiJiaInfofinal.setList(finallist);
        return baiJiaInfofinal;
    }
}
```

### 添加task类

```java
import com.youfan.entity.BaiJiaInfo;
import com.youfan.entity.CarrierInfo;
import com.youfan.map.BaijiaMap;
import com.youfan.map.CarrierMap;
import com.youfan.reduce.BaijiaReduce;
import com.youfan.reduce.CarrierReduce;
import com.youfan.util.DateUtils;
import com.youfan.util.HbaseUtil;
import com.youfan.util.MongoUtils;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.utils.ParameterTool;
import org.bson.Document;

import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.*;

/**
 * Created by li on 2019/1/5.
 */
public class BaiJiaTask {
    public static void main(String[] args) {
        final ParameterTool params = ParameterTool.fromArgs(args);

        // set up the execution environment
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        // make parameters available in the web interface
        env.getConfig().setGlobalJobParameters(params);

        // get input data
        DataSet<String> text = env.readTextFile(params.get("input"));

        DataSet<BaiJiaInfo> mapresult = text.map(new BaijiaMap());
        DataSet<BaiJiaInfo> reduceresutl = mapresult.groupBy("groupfield").reduce(new BaijiaReduce());
        try {
            List<BaiJiaInfo> reusltlist = reduceresutl.collect();
            for(BaiJiaInfo baiJiaInfo:reusltlist){
                    String userid = baiJiaInfo.getUserid();
                    List<BaiJiaInfo> list = baiJiaInfo.getList();
                    Collections.sort(list, new Comparator<BaiJiaInfo>() {
                        @Override
                        public int compare(BaiJiaInfo o1, BaiJiaInfo o2) {
                            String timeo1 = o1.getCreatetime();
                            String timeo2 = o2.getCreatetime();
                            DateFormat dateFormat = new SimpleDateFormat("yyyyMMdd hhmmss");
                            Date datenow = new Date();
                            Date time1 = datenow;
                                    Date time2 = datenow;
                            try {
                                time1 = dateFormat.parse(timeo1);
                                time2 = dateFormat.parse(timeo2);
                            } catch (ParseException e) {
                                e.printStackTrace();
                            }
                            return time1.compareTo(time2);
                        }
                    });
                BaiJiaInfo before = null;
                Map<Integer,Integer> frequencymap = new HashMap<Integer,Integer>();
                double maxamount = 0d;
                double sum = 0d;
                for(BaiJiaInfo baiJiaInfoinner:list){
                    if(before==null){
                        before = baiJiaInfoinner;
                        continue;
                    }
                    //计算购买的频率
                    String beforetime = before.getCreatetime();
                    String endstime = baiJiaInfoinner.getCreatetime();
                    int days = DateUtils.getDaysBetweenbyStartAndend(beforetime,endstime,"yyyyMMdd hhmmss");
                    int brefore = frequencymap.get(days)==null?0:frequencymap.get(days);
                    frequencymap.put(days,brefore+1);

                    //计算最大金额
                    String totalamountstring = baiJiaInfoinner.getTotalamount();
                    Double totalamout = Double.valueOf(totalamountstring);
                    if(totalamout>maxamount){
                        maxamount = totalamout;
                    }

                    //计算平均值
                    sum += totalamout;

                    before = baiJiaInfoinner;
                }
                double avramount = sum/list.size();
                int totaldays = 0;
                Set<Map.Entry<Integer,Integer>> set = frequencymap.entrySet();
                for(Map.Entry<Integer,Integer> entry :set){
                    Integer frequencydays = entry.getKey();
                    Integer count = entry.getValue();
                    totaldays += frequencydays*count;
                }
                int avrdays = totaldays/list.size();//平均天数

                //败家指数 = 支付金额平均值*0.3、最大支付金额*0.3、下单频率*0.4
                //支付金额平均值30分（0-20 5 20-60 10 60-100 20 100-150 30 150-200 40 200-250 60 250-350 70 350-450 80 450-600 90 600以上 100  ）
                // 最大支付金额30分（0-20 5 20-60 10 60-200 30 200-500 60 500-700 80 700 100）
                // 下单平率30分 （0-5 100 5-10 90 10-30 70 30-60 60 60-80 40 80-100 20 100以上的 10）
                int avraoumtsoce = 0;
                if(avramount>=0 && avramount < 20){
                    avraoumtsoce = 5;
                }else if (avramount>=20 && avramount < 60){
                    avraoumtsoce = 10;
                }else if (avramount>=60 && avramount < 100){
                    avraoumtsoce = 20;
                }else if (avramount>=100 && avramount < 150){
                    avraoumtsoce = 30;
                }else if (avramount>=150 && avramount < 200){
                    avraoumtsoce = 40;
                }else if (avramount>=200 && avramount < 250){
                    avraoumtsoce = 60;
                }else if (avramount>=250 && avramount < 350){
                    avraoumtsoce = 70;
                }else if (avramount>=350 && avramount < 450){
                    avraoumtsoce = 80;
                }else if (avramount>=450 && avramount < 600){
                    avraoumtsoce = 90;
                }else if (avramount>=600){
                    avraoumtsoce = 100;
                }

                int maxaoumtscore = 0;
                if(maxamount>=0 && maxamount < 20){
                    maxaoumtscore = 5;
                }else if (maxamount>=20 && maxamount < 60){
                    maxaoumtscore = 10;
                }else if (maxamount>=60 && maxamount < 200){
                    maxaoumtscore = 30;
                }else if (maxamount>=200 &&maxamount < 500){
                    maxaoumtscore = 60;
                }else if (maxamount>=500 && maxamount < 700){
                    maxaoumtscore = 80;
                }else if (maxamount>=700){
                    maxaoumtscore = 100;
                }

                // 下单平率30分 （0-5 100 5-10 90 10-30 70 30-60 60 60-80 40 80-100 20 100以上的 10）
                int avrdaysscore = 0;
                if(avrdays>=0 && avrdays < 5){
                    avrdaysscore = 100;
                }else if (avramount>=5 && avramount < 10){
                    avrdaysscore = 90;
                }else if (avramount>=10 && avramount < 30){
                    avrdaysscore = 70;
                }else if (avramount>=30 && avramount < 60){
                    avrdaysscore = 60;
                }else if (avramount>=60 && avramount < 80){
                    avrdaysscore = 40;
                }else if (avramount>=80 && avramount < 100){
                    avrdaysscore = 20;
                }else if (avramount>=100){
                    avrdaysscore = 10;
                }
                double totalscore = (avraoumtsoce/100)*30+(maxaoumtscore/100)*30+(avrdaysscore/100)*40;

                String tablename = "userflaginfo";
                String rowkey = userid;
                String famliyname = "baseinfo";
                String colum = "baijiasoce";
                HbaseUtil.putdata(tablename,rowkey,famliyname,colum,totalscore+"");
            }
            env.execute("baijiascore analy");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

### 添加工具类DateUtils

```java
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

/**
 * Created by li on 2019/1/5.
 */
public class DateUtils {
    public static String getYearbasebyAge(String age){
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(new Date());
        calendar.add(Calendar.YEAR,-Integer.valueOf(age));
        Date newdate = calendar.getTime();
        DateFormat dateFormat = new SimpleDateFormat("yyyy");
        String newdatestring = dateFormat.format(newdate);
        Integer newdateinteger = Integer.valueOf(newdatestring);
        String yearbasetype = "未知";
        if(newdateinteger >= 1940 && newdateinteger < 1950){
            yearbasetype = "40后";
        }else if (newdateinteger >= 1950 && newdateinteger < 1960){
            yearbasetype = "50后";
        }else if (newdateinteger >= 1960 && newdateinteger < 1970){
            yearbasetype = "60后";
        }else if (newdateinteger >= 1970 && newdateinteger < 1980){
            yearbasetype = "70后";
        }else if (newdateinteger >= 1980 && newdateinteger < 1990){
            yearbasetype = "80后";
        }else if (newdateinteger >= 1990 && newdateinteger < 2000){
            yearbasetype = "90后";
        }else if (newdateinteger >= 2000 && newdateinteger < 2010){
            yearbasetype = "00后";
        }else if (newdateinteger >= 2010 ){
            yearbasetype = "10后";
        }
        return yearbasetype;
    }


    public static int getDaysBetweenbyStartAndend(String starttime,String endTime,String dateFormatstring) throws ParseException {
        DateFormat dateFormat = new SimpleDateFormat(dateFormatstring);
        Date start = dateFormat.parse(starttime);
        Date end = dateFormat.parse(endTime);
        Calendar startcalendar = Calendar.getInstance();
        Calendar endcalendar = Calendar.getInstance();
        startcalendar.setTime(start);
        endcalendar.setTime(end);
        int days = 0;
        while(startcalendar.before(endcalendar)){
                startcalendar.add(Calendar.DAY_OF_YEAR,1);
                days += 1;
        }
        return days;
    }

    public static String gethoursbydate(String timevalue) throws ParseException {
        DateFormat dateFormat = new SimpleDateFormat("yyyyMMdd hhmmss");
        Date time = dateFormat.parse(timevalue);
        dateFormat = new SimpleDateFormat("hh");
        String resulthour = dateFormat.format(time);
        return resulthour;
    }
}
```

