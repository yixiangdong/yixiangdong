---
layout: clickhouse4
title: Flink+ClickHouse[精华]电商实时数据分析系统之客户端数据上报
date: 2020-04-05 21:01:29
tags: ClickHouse
password: 2020-04-05!!
abstract: 该文章已加密, 请加博主微信号:books_111索要密码查看。
message: 该文章已加密, 请加博主微信号:books_111索要密码查看。
wrong_pass_message: 密码不正确，请重新输入！
wrong_hash_message: 文章不能被校验, 不过您还是能看看解密后的内容！
comments: true
---

### 客户端数据上报

#### 在common模块test目录下新建SendLogData类

<!--more-->

```java
package com.jiangzi.test;

import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.serializer.SerializerFeature;
import com.jiangzi.input.AppInfo;
import com.jiangzi.input.PcInfo;
import com.jiangzi.input.ScanPageLog;

import java.io.InputStream;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class SendLogData {
    private static void postHttpMethod(String urlpath,String data){
        try {
            URL url = new URL(urlpath);
            HttpURLConnection urlConnection = (HttpURLConnection)url.openConnection();
            urlConnection.setRequestMethod("POST");
            urlConnection.setDoInput(true);
            urlConnection.setDoOutput(true);
            urlConnection.setInstanceFollowRedirects(true);
            urlConnection.setUseCaches(true);
            urlConnection.setRequestProperty("User-Agent", "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:26.0) Gecko/20100101 Firefox/26.0");
            urlConnection.setRequestProperty("Content-Type", "application/json");
            urlConnection.setConnectTimeout(1000 * 5);
            urlConnection.connect();
            OutputStream outputStream = urlConnection.getOutputStream();
            outputStream.write(data.getBytes("utf-8"));
            outputStream.flush();
            outputStream.close();
            InputStream inputStream = urlConnection.getInputStream();
            int httpCode = urlConnection.getResponseCode();
            byte[] inputdata = new byte[1024];
            StringBuffer stringBuffer = new StringBuffer();
            while(inputStream.read(inputdata,0,1024) != -1){
                stringBuffer.append(new String (inputdata));
            }
            System.out.println(httpCode);
            System.out.println(stringBuffer.toString());
            inputStream.close();
        } catch (Exception e) {

        }

    }

    public static void main(String[] args) {
        AppInfo appInfo = new AppInfo();
        String appinfoString = JSONObject.toJSONString(appInfo, SerializerFeature.WriteMapNullValue);


        String deviceId = appInfo.getDeviceId();
        String openTime = appInfo.getOpenTime();

        try {
        PcInfo pcInfo = new PcInfo();
        pcInfo.setDeviceId("24564545645");
        DateFormat dateFormat = new SimpleDateFormat("yyyyMMddHH");
            Date time = dateFormat.parse("2019101106");
            String timeString = time.getTime()+"";
            pcInfo.setOpenTime(timeString);
            ScanPageLog scanPageLog = new ScanPageLog();
            scanPageLog.setDeviceType("1");
            time = dateFormat.parse("2019101107");
            scanPageLog.setVisitTime(time.getTime()+"");
            scanPageLog.setDeviceComomInfo(pcInfo);
            String scanPageLogString = JSONObject.toJSONString(scanPageLog, SerializerFeature.WriteMapNullValue);
            System.out.println(scanPageLogString);
            postHttpMethod("http://127.0.0.1:9081/dataCollect",scanPageLogString);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}

```

为common模块添加alibaba fastjson,kafka,Hbase依赖

```java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.58</version>
</dependency>
<dependency>
	<groupId>org.apache.hbase</groupId>
	<artifactId>hbase-client</artifactId>
	<version>1.0.0</version>
</dependency>
<dependency>
	<groupId>org.apache.kafka</groupId>
	<artifactId>kafka-clients</artifactId>
	<version>2.1.0</version>
</dependency>

```

#### 在数据收集dataCollect模块完善数据收集接口类DataCollection

添加common模块依赖到dataCollect模块

```
<dependency>
    <groupId>com.jiangzi</groupId>
    <artifactId>jiangziCommon</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

完善接口类

```java
package com.jiangzi.control;

import com.jiangzi.dataCollectUtils.UserStatus;
import com.jiangzi.input.AppInfo;
import com.jiangzi.input.PcInfo;
import com.jiangzi.input.ScanPageLog;
import com.jiangzi.input.XiaochengxuInfo;
import org.apache.commons.lang.StringUtils;
import com.alibaba.fastjson.JSONObject;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.kafka.core.KafkaTemplate;

/**
 * 数据收集服务
 */
@RestController
public class DataCollection {
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @RequestMapping(method = RequestMethod.POST,value ="helloWorld" )
    public String toHelloWorld(String name) {
        return name;
    }

    @RequestMapping(method = RequestMethod.POST,value = "dataCollect")
    public void dataCollect(@RequestBody String data){
        if(StringUtils.isNotBlank(data)){
            JSONObject jsonObject = JSONObject.parseObject(data);
            String deviceType = jsonObject.getString("deviceType");
            ScanPageLog scanPageLog = JSONObject.parseObject(data,ScanPageLog.class);
            String deviceComomInfo = jsonObject.getString("deviceComomInfo");
            //0、app端 1、pc端 2、小程序端
            if("0".equals(deviceType)){
                System.out.println("进入app端");
                AppInfo appinfo = JSONObject.parseObject(deviceComomInfo, AppInfo.class);
                UserStatus.filterNewStatus(appinfo);
                UserStatus.filterActiveStatus(appinfo);
                scanPageLog.setDeviceComomInfo(appinfo);
            }else if("1".equals(deviceType)){
                System.out.println("进入pc端");
                PcInfo pcInfo = JSONObject.parseObject(deviceComomInfo, PcInfo.class);
                UserStatus.filterNewStatus(pcInfo);
                UserStatus.filterActiveStatus(pcInfo);
                scanPageLog.setDeviceComomInfo(pcInfo);
            }else if("2".equals(deviceType)){
                System.out.println("进入小程序端");
                XiaochengxuInfo xiaochengxuInfo = JSONObject.parseObject(deviceComomInfo, XiaochengxuInfo.class);
                UserStatus.filterNewStatus(xiaochengxuInfo);
                UserStatus.filterActiveStatus(xiaochengxuInfo);
                scanPageLog.setDeviceComomInfo(xiaochengxuInfo);
            }
            String scanPageLogString = JSONObject.toJSONString(scanPageLog);
            System.out.println(scanPageLogString);
            kafkaTemplate.send("datainfo", scanPageLogString);
        }

    }

}

```

#### 添加子类用户状态类

```java
package com.jiangzi.dataCollectUtils;


import com.jiangzi.input.AppInfo;
import com.jiangzi.input.DeviceComomInfo;
import com.jiangzi.input.PcInfo;
import com.jiangzi.input.XiaochengxuInfo;
import com.jiangzi.utils.DateUtils;
import com.jiangzi.utils.HbaseUtils2;
import org.apache.commons.lang.StringUtils;

/**
 * Created by Administrator on 2020/2/18.
 */
public class UserStatus {
    /**
     * 过滤是否是新增用户状态
     * create devicecomominfoapp,info
     * create devicecomominfopc,info
     * create devicecomominfoxiaochengxu,info
     *deviceComomInfo 设备信息
     * @return
     */
    public static void filterNewStatus(DeviceComomInfo deviceComomInfo){
        if(deviceComomInfo instanceof AppInfo){
            AppInfo appInfo = (AppInfo)deviceComomInfo;
            String deviceId = appInfo.getDeviceId();
            String openTime = appInfo.getOpenTime();
            try {
                String result = HbaseUtils2.getdata("devicecomominfoapp",deviceId,"info","uniqueId");
                if(StringUtils.isBlank(result)){
                    String result2 = HbaseUtils2.getdata("devicecomominfopc",deviceId,"info","uniqueId");
                    if(StringUtils.isBlank(result2)){
                        appInfo.setNew(true);
                        HbaseUtils2.putdata("devicecomominfoapp",deviceId,"info","uniqueId",deviceId);
                    }
                }
                deviceComomInfo = appInfo;
            } catch (Exception e) {
                e.printStackTrace();
            }
        }else if (deviceComomInfo instanceof PcInfo){
            PcInfo pcInfo = (PcInfo)deviceComomInfo;
            String macAdress = pcInfo.getMacAdress();
            String deviceId = pcInfo.getDeviceId();
            if(StringUtils.isNotBlank(macAdress)){
                try {
                    String result = HbaseUtils2.getdata("devicecomominfopc",macAdress,"info","uniqueId");
                    if(StringUtils.isBlank(result)){
                        HbaseUtils2.putdata("devicecomominfopc",macAdress,"info","uniqueId",macAdress);
                        pcInfo.setNew(true);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }else if (StringUtils.isNotBlank(deviceId)){
                try {
                    String result = HbaseUtils2.getdata("devicecomominfopc",deviceId,"info","uniqueId");
                    if(StringUtils.isBlank(result)){
                        String result2 = HbaseUtils2.getdata("devicecomominfoapp",deviceId,"info","uniqueId");
                        if(StringUtils.isBlank(result2)){
                            HbaseUtils2.putdata("devicecomominfopc",deviceId,"info","uniqueId",deviceId);
                            pcInfo.setNew(true);
                        }
                    }

                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            deviceComomInfo = pcInfo;
        }else if (deviceComomInfo instanceof XiaochengxuInfo){
            XiaochengxuInfo xiaochengxuInfo = (XiaochengxuInfo)deviceComomInfo;
            String weixinAccount = xiaochengxuInfo.getWeixinAccount();
            try {
                String result = HbaseUtils2.getdata("devicecomominfoxiaochengxu",weixinAccount,"info","uniqueId");
                if(StringUtils.isBlank(result)){
                    xiaochengxuInfo.setNew(true);
                    HbaseUtils2.putdata("devicecomominfoxiaochengxu",weixinAccount,"info","uniqueId",weixinAccount);
                }
                deviceComomInfo = xiaochengxuInfo;
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public static void filterActiveStatus(DeviceComomInfo deviceComomInfo){
        if(deviceComomInfo instanceof AppInfo){
            AppInfo appInfo = (AppInfo)deviceComomInfo;
            String deviceId = appInfo.getDeviceId();
            String openTime = appInfo.getOpenTime();
            Long openTimeMillons = Long.valueOf(openTime);
            try {
                String result = HbaseUtils2.getdata("devicecomominfoapp",deviceId,"info","lastvisittime");
                String lastVisitTime = "";
                if(StringUtils.isBlank(result)){
                    String result2 = HbaseUtils2.getdata("devicecomominfopc",deviceId,"info","lastvisittime");
                    if(StringUtils.isBlank(result2)){
                        appInfo.setHourActive(true);
                        appInfo.setDayActive(true);
                        appInfo.setMonthActive(true);
                        appInfo.setWeekActive(true);
                        HbaseUtils2.putdata("devicecomominfoapp",deviceId,"info","lastvisittime",openTime);
                    }else {
                        lastVisitTime = result2;
                    }
                }else{
                    lastVisitTime = result;
                }

                if(StringUtils.isNotBlank(lastVisitTime)){
                    long lastVisitTimeMillons = Long.valueOf(lastVisitTime);
                    //小时
                    long hourStart = DateUtils.getCurrentHourStart(openTimeMillons);
                    if(lastVisitTimeMillons < hourStart){
                        appInfo.setHourActive(true);
                    }

                    //天
                    long dayStart = DateUtils.getCurrentDayStart(openTimeMillons);
                    if(lastVisitTimeMillons < dayStart){
                        appInfo.setDayActive(true);
                    }

                    //周
                    long weekStart = DateUtils.getCurrentWeekStart(openTimeMillons);
                    if(lastVisitTimeMillons < weekStart){
                        appInfo.setDayActive(true);
                    }

                    //月
                    long monthStart = DateUtils.getCurrentMonthStart(openTimeMillons);
                    if(lastVisitTimeMillons < monthStart){
                        appInfo.setMonthActive(true);
                    }

                    //分钟
                    long fiveMinuteIn = DateUtils.getCurrentFiveMinuteInterStart(openTimeMillons);
                    if(lastVisitTimeMillons < fiveMinuteIn){
                        appInfo.setFiveMinuteActive(true);
                    }
                }
                HbaseUtils2.putdata("devicecomominfoapp",deviceId,"info","lastvisittime",openTime);

            } catch (Exception e) {
                e.printStackTrace();
            }
        }else if (deviceComomInfo instanceof PcInfo){
            PcInfo pcInfo = (PcInfo)deviceComomInfo;
            String macAdress = pcInfo.getMacAdress();
            String deviceId = pcInfo.getDeviceId();
            String openTime = pcInfo.getOpenTime();
            Long openTimeMillons = Long.valueOf(openTime);
            String lastVisitTime = "";
            try {
                    if(StringUtils.isNotBlank(macAdress)) {
                        String result = HbaseUtils2.getdata("devicecomominfopc", macAdress, "info", "lastvisittime");
                        if (StringUtils.isNotBlank(result)) {
                            lastVisitTime = result;
                        }else{
                            pcInfo.setHourActive(true);
                            pcInfo.setDayActive(true);
                            pcInfo.setMonthActive(true);
                            pcInfo.setWeekActive(true);
                            HbaseUtils2.putdata("devicecomominfopc", macAdress, "info", "lastvisittime",openTime);
                        }
                        HbaseUtils2.putdata("devicecomominfopc",macAdress,"info","lastvisittime",openTime);
                    }else if(StringUtils.isNotBlank(deviceId)) {
                        String result = HbaseUtils2.getdata("devicecomominfopc", deviceId, "info", "lastvisittime");
                        if (StringUtils.isBlank(result)) {
                            String result2 = HbaseUtils2.getdata("devicecomominfoapp", deviceId, "info", "lastvisittime");
                            if (StringUtils.isBlank(result2)) {
                                pcInfo.setHourActive(true);
                                pcInfo.setDayActive(true);
                                pcInfo.setMonthActive(true);
                                pcInfo.setWeekActive(true);
                                HbaseUtils2.putdata("devicecomominfopc", deviceId, "info", "lastvisittime", openTime);
                            } else {
                                lastVisitTime = result2;
                            }
                        }else{
                            lastVisitTime = result;
                        }
                        HbaseUtils2.putdata("devicecomominfopc",deviceId,"info","lastvisittime",openTime);
                    }
                if(StringUtils.isNotBlank(lastVisitTime)){
                    long lastVisitTimeMillons = Long.valueOf(lastVisitTime);
                    //小时
                    long hourStart = DateUtils.getCurrentHourStart(openTimeMillons);
                    if(lastVisitTimeMillons < hourStart){
                        pcInfo.setHourActive(true);
                    }

                    //天
                    long dayStart = DateUtils.getCurrentDayStart(openTimeMillons);
                    if(lastVisitTimeMillons < dayStart){
                        pcInfo.setDayActive(true);
                    }

                    //周
                    long weekStart = DateUtils.getCurrentWeekStart(openTimeMillons);
                    if(lastVisitTimeMillons < weekStart){
                        pcInfo.setDayActive(true);
                    }

                    //月
                    long monthStart = DateUtils.getCurrentMonthStart(openTimeMillons);
                    if(lastVisitTimeMillons < monthStart){
                        pcInfo.setMonthActive(true);
                    }

                    //分钟
                    long fiveMinuteIn = DateUtils.getCurrentFiveMinuteInterStart(openTimeMillons);
                    if(lastVisitTimeMillons < fiveMinuteIn){
                        pcInfo.setFiveMinuteActive(true);
                    }
                }


            } catch (Exception e) {
                    e.printStackTrace();
            }

        }else if (deviceComomInfo instanceof XiaochengxuInfo){
            XiaochengxuInfo xiaochengxuInfo = (XiaochengxuInfo)deviceComomInfo;
            String weixinAccount = xiaochengxuInfo.getWeixinAccount();
            String openTime = xiaochengxuInfo.getOpenTime();
            Long openTimeMillons = Long.valueOf(openTime);
            String lastVisitTime = "";
            try {
                String result = HbaseUtils2.getdata("devicecomominfoxiaochengxu", weixinAccount, "info", "lastvisittime");
                if(StringUtils.isBlank(result)){
                    xiaochengxuInfo.setHourActive(true);
                    xiaochengxuInfo.setDayActive(true);
                    xiaochengxuInfo.setMonthActive(true);
                    xiaochengxuInfo.setWeekActive(true);
                    HbaseUtils2.putdata("devicecomominfoxiaochengxu",weixinAccount, "info", "lastvisittime", openTime);
                }else{
                    lastVisitTime = result;
                }

                if(StringUtils.isNotBlank(lastVisitTime)){
                    long lastVisitTimeMillons = Long.valueOf(lastVisitTime);
                    //小时
                    long hourStart = DateUtils.getCurrentHourStart(openTimeMillons);
                    if(lastVisitTimeMillons < hourStart){
                        xiaochengxuInfo.setHourActive(true);
                    }

                    //天
                    long dayStart = DateUtils.getCurrentDayStart(openTimeMillons);
                    if(lastVisitTimeMillons < dayStart){
                        xiaochengxuInfo.setDayActive(true);
                    }

                    //周
                    long weekStart = DateUtils.getCurrentWeekStart(openTimeMillons);
                    if(lastVisitTimeMillons < weekStart){
                        xiaochengxuInfo.setDayActive(true);
                    }

                    //月
                    long monthStart = DateUtils.getCurrentMonthStart(openTimeMillons);
                    if(lastVisitTimeMillons < monthStart){
                        xiaochengxuInfo.setMonthActive(true);
                    }

                    //分钟

                    long fiveMinuteIn = DateUtils.getCurrentFiveMinuteInterStart(openTimeMillons);
                    if(lastVisitTimeMillons < fiveMinuteIn){
                        xiaochengxuInfo.setFiveMinuteActive(true);
                    }
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}

```

添加common模块工具类

工具类:

DateUtils.java

```java
package com.jiangzi.utils;

import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

/**
 * Created by Administrator on 2020/2/18.
 */
public class DateUtils {

    public static Long getCurrentHourStart(Long visitTime){
        Date date = new Date(visitTime);
        DateFormat dateFormat = new SimpleDateFormat("yyyyMMdd HH");
        try {
            Date filterTime = dateFormat.parse(dateFormat.format(date));
            return filterTime.getTime();
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }

    public static Long getCurrentDayStart(Long visitTime){
        Date date = new Date(visitTime);
        DateFormat dateFormat = new SimpleDateFormat("yyyyMMdd");
        try {
            Date filterTime = dateFormat.parse(dateFormat.format(date));
            return filterTime.getTime();
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }

    public static Long getCurrentWeekStart(Long visitTime){
        Calendar cal =Calendar.getInstance();
        if (null != visitTime) {
            cal.setTimeInMillis(visitTime);
        }
        cal.set(Calendar.DAY_OF_WEEK, Calendar.MONDAY);
        cal.set(Calendar.HOUR_OF_DAY, 0);
        cal.set(Calendar.MINUTE, 0);
        cal.set(Calendar.SECOND, 0);
        cal.set(Calendar.MILLISECOND, 0);
        return cal.getTimeInMillis();
    }

    public static Long getCurrentMonthStart(Long visitTime){
        Calendar cal =Calendar.getInstance();

        if (null != visitTime) {
            cal.setTimeInMillis(visitTime);
        }
        cal.set(Calendar.DAY_OF_MONTH, 1);
        cal.set(Calendar.HOUR_OF_DAY, 0);
        cal.set(Calendar.MINUTE, 0);
        cal.set(Calendar.SECOND, 0);
        cal.set(Calendar.MILLISECOND, 0);
        return cal.getTimeInMillis();

    }


    public static Long getCurrentFiveMinuteInterStart(Long visitTime){
        String timeString = getByinterMinute(visitTime+"");
        DateFormat dateFormat = new SimpleDateFormat("yyyyMMddHHmm");
        try {
            Date date = dateFormat.parse(timeString);
            return date.getTime();
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }
    public static String getByinterMinute(String timeinfo){
        Long timeMillons = Long.valueOf(timeinfo);
        Date date = new Date(timeMillons);
        DateFormat dateFormatMinute = new SimpleDateFormat("mm");
        DateFormat dateFormatHour = new SimpleDateFormat("yyyyMMddHH");
        String minute = dateFormatMinute.format(date);
        String hour = dateFormatHour.format(date);
        Long minuteLong = Long.valueOf(minute);
        String replaceMinute = "";
        if(minuteLong >= 0 && minuteLong <5){//0-5
            replaceMinute = "05";
        }else if (minuteLong >= 5 && minuteLong <10){
            replaceMinute = "10";
        }else if (minuteLong >= 10 && minuteLong <15){
            replaceMinute = "15";
        }else if (minuteLong >= 15 && minuteLong <20){
            replaceMinute = "20";
        }else if (minuteLong >= 20 && minuteLong <25){
            replaceMinute = "25";
        }else if (minuteLong >= 25 && minuteLong <30){
            replaceMinute = "30";
        }else if (minuteLong >= 30 && minuteLong <35){
            replaceMinute = "35";
        }else if (minuteLong >= 35 && minuteLong <40){
            replaceMinute = "40";
        }else if (minuteLong >= 40 && minuteLong <45){
            replaceMinute = "45";
        }else if (minuteLong >= 45 && minuteLong <50){
            replaceMinute = "50";
        }else if (minuteLong >= 50 && minuteLong <55){
            replaceMinute = "55";
        }else if (minuteLong >= 55 && minuteLong <60){
            replaceMinute = "60";
        }
        String fullTime = hour+replaceMinute;
        return fullTime;
    }

    public static String getByinterHour(String timeinfo){
        Long timeMillons = Long.valueOf(timeinfo);
        Date date = new Date(timeMillons);
        DateFormat dateFormatHour = new SimpleDateFormat("yyyyMMddHH");
        String result = dateFormatHour.format(date);
        return result;
    }

    public static String getByMillons(String timeinfo,String dateFormatString){
        Long timeMillons = Long.valueOf(timeinfo);
        Date date = new Date(timeMillons);
        DateFormat dateFormatHour = new SimpleDateFormat(dateFormatString);
        String result = dateFormatHour.format(date);
        return result;
    }

    public static String transferFormat(String timeinfoString,String dateFromat,String transferFormat){
        DateFormat dateFormatOld = new SimpleDateFormat(dateFromat);
        DateFormat dateFormatnew = new SimpleDateFormat(transferFormat);
        String finalString = "";
        try {
            Date date = dateFormatOld.parse(timeinfoString);
            finalString = dateFormatnew.format(date);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return finalString;
    }

    public static int compareDate(String o1, String o2,String dateFormatpara) throws ParseException {
        DateFormat dateFormat = new SimpleDateFormat(dateFormatpara);
        int result = (dateFormat.parse(o1)).compareTo(dateFormat.parse(o2));
        return result;

    }

    public static void main(String[] args) {
        DateFormat dateFormat = new SimpleDateFormat("yyyyMMdd HHmmss");
        try {
//            Date date = dateFormat.parse("20180907 040752");
//            String result = getByinterMinute(date.getTime()+"");
//            System.out.println(result);
//            result = getByinterHour(date.getTime()+"");
//            System.out.println(result);
            Date date = dateFormat.parse("20180907 040756");
            long timesss  = date.getTime();
            System.out.println(timesss);
            date = dateFormat.parse("20180907 050746");
            timesss  = date.getTime();
            System.out.println(timesss);
        } catch (Exception e) {
            e.printStackTrace();
        }

        try {
            int result = compareDate("20190807 02","20190807 06","yyyyMMdd HH");
            System.out.println(result);
        } catch (ParseException e) {
            e.printStackTrace();
        }


    }

}
```

HbaseUtils2.java

```java
package com.jiangzi.utils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;
import java.io.IOException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;


public class HbaseUtils2 {
    private static Admin admin = null;
    private static Connection conn = null;
    static{
        // 创建hbase配置对象
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.rootdir","hdfs://192.168.246.152:9000/hbase");
        //使用eclipse时必须添加这个，否则无法定位
        conf.set("hbase.zookeeper.quorum","192.168.246.152");
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
     * 创建表
     */
    public  static void createTable(String tabName,String famliyname) throws Exception {
        HTableDescriptor tab = new HTableDescriptor(tabName);
        // 添加列族,每个表至少有一个列族
        HColumnDescriptor colDesc = new HColumnDescriptor(famliyname);
        tab.addFamily(colDesc);
        // 创建表
        admin.createTable(tab);
        System.out.println("over");
    }

    /**
     * 插入数据，create "testinfo","time"
     * create "product","info"
     * create "userinfo","info"
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
     * ܱ获取数据，create "testinfo","time"
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
//        System.setProperty("hadoop.home.dir","E:\\soft\\hadoop-2.6.0-cdh5.5.1\\hadoop-2.6.0-cdh5.5.1");
//        createTable("testinfo","time");
//        putdata("testinfo", "1", "time","info","ty");
//        Map<String,String> datamap = new HashMap<String,String>();
//        datamap.put("info1","ty1");
//        datamap.put("info2","ty2");
//        put("testinfo", "2", "time",datamap);

        String result = getdata("testinfo","2", "time","info1");
        System.out.println(result);

    }


}
```

KafkaUtils.java

```java
package com.jiangzi.utils;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Properties;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

/**
 * Created by Administrator on 2020/2/28.
 */
public class KafkaUtils {
    private static Properties getProps(){
        Properties props = new Properties();
        props.put("bootstrap.servers", "192.168.246.152:9092");
        props.put("acks", "all"); // 发送所有ISR
        props.put("retries", 2); // 重试次数
//       props.put("batch.size", 16384); // 批量发送大小
//        props.put("buffer.memory", 33554432); // 缓存大小，根据本机内存大小配置
        props.put("linger.ms", 1000); // 发送频率，满足任务一个条件发送
        props.put("client.id", "producer-syn-1"); // 发送端id,便于统计
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        return props;
    }

    public static void sendData(String topicName,String data){
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(getProps());
        ProducerRecord<String, String> record = new ProducerRecord<String, String>(topicName,data);
        Future<RecordMetadata> metadataFuture = producer.send(record);
        try {
            RecordMetadata recordMetadata = metadataFuture.get();
            System.out.println("topic:"+recordMetadata.topic());
            System.out.println("partition:"+recordMetadata.partition());
            System.out.println("offset:"+recordMetadata.offset());

        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

    }

    public static void main(String[] args) {
        sendData("test","{\"id\":\"1\"}");
    }
}

```

#### 在数据收集模块添加kafka依赖

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

#### 完善配置文件application.properties

```reStructuredText
server.port=9081
spring.application.name=jiangziDataCollect

eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/

spring.kafka.bootstrap-servers=192.168.37.141:9092
spring.kafka.consumer.group-id=jiangzi

```

#### 手动创建kafka topic

topic 名字: : datainfo

kafka 创建的topic的命令比较简单不在这里介绍了

