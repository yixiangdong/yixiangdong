---
layout: ceshi8
title: 实时用户画像系统(三)手机运营商标签代码编写
date: 2020-03-25 21:43:17
tags: 机器学习
comments: true
---

### 设计流程类似于用户年代标签

<!-- more -->

```java
//手机运营商工具类
package com.youfan.util;

import java.util.regex.Pattern;

/**
 * Created by li on 2019/1/5.
 */
public class CarrierUtils {

    /**
     * 中国电信号码格式验证 手机段： 133,153,180,181,189,177,1700,173,199
     **/
    private static final String CHINA_TELECOM_PATTERN = "(^1(33|53|77|73|99|8[019])\\d{8}$)|(^1700\\d{7}$)";

    /**
     * 中国联通号码格式验证 手机段：130,131,132,155,156,185,186,145,176,1709
     **/
    private static final String CHINA_UNICOM_PATTERN = "(^1(3[0-2]|4[5]|5[56]|7[6]|8[56])\\d{8}$)|(^1709\\d{7}$)";

    /**
     * 中国移动号码格式验证
     * 手机段：134,135,136,137,138,139,150,151,152,157,158,159,182,183,184,187,188,147,178,1705
     **/
    private static final String CHINA_MOBILE_PATTERN = "(^1(3[4-9]|4[7]|5[0-27-9]|7[8]|8[2-478])\\d{8}$)|(^1705\\d{7}$)";


    /**
     * 0、未知 1、移动 2、联通 3、电信
     * @param telphone
     * @return
     */
    public static int getCarrierByTel(String telphone){
        boolean b1 = telphone == null || telphone.trim().equals("") ? false : match(CHINA_MOBILE_PATTERN, telphone);
        if (b1) {
            return 1;
        }
        b1 = telphone == null || telphone.trim().equals("") ? false : match(CHINA_UNICOM_PATTERN, telphone);
        if (b1) {
            return 2;
        }
        b1 = telphone == null || telphone.trim().equals("") ? false : match(CHINA_TELECOM_PATTERN, telphone);
        if (b1) {
            return 3;
        }
        return 0;
    }

    /**
     * 匹配函数
     * @param regex
     * @param tel
     * @return
     */
    private static boolean match(String regex, String tel) {
        return Pattern.matches(regex, tel);
    }


}
```

实体类

```java
public class CarrierInfo {
    private String carrier;//运营商
    private Long count;//数量
    private String groupfield;//分组

    public String getCarrier() {
        return carrier;
    }

    public void setCarrier(String carrier) {
        this.carrier = carrier;
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

map数据处理类

```java
import com.youfan.entity.CarrierInfo;
import com.youfan.entity.YearBase;
import com.youfan.util.CarrierUtils;
import com.youfan.util.DateUtils;
import com.youfan.util.HbaseUtil;
import org.apache.commons.lang3.StringUtils;
import org.apache.flink.api.common.functions.MapFunction;

/**
 * Created by li on 2019/1/5.
 */
public class CarrierMap implements MapFunction<String, CarrierInfo>{
    @Override
    public CarrierInfo map(String s) throws Exception {
        if(StringUtils.isBlank(s)){
            return null;
        }
        String[] userinfos = s.split(",");
        String userid = userinfos[0];
        String username = userinfos[1];
        String sex = userinfos[2];
        String telphone = userinfos[3];
        String email = userinfos[4];
        String age = userinfos[5];
        String registerTime = userinfos[6];
        String usetype = userinfos[7];//'终端类型：0、pc端；1、移动端；2、小程序端'

        int carriertype = CarrierUtils.getCarrierByTel(telphone);
        String carriertypestring = carriertype==0?"未知运营商":carriertype==1?"移动用户":carriertype==2?"联通用户":"电信用户";

        String tablename = "userflaginfo";
        String rowkey = userid;
        String famliyname = "baseinfo";
        String colum = "carrierinfo";//运营商
        HbaseUtil.putdata(tablename,rowkey,famliyname,colum,carriertypestring);
        CarrierInfo carrierInfo = new CarrierInfo();
        String groupfield = "carrierInfo=="+carriertype;
        carrierInfo.setCount(1l);
        carrierInfo.setCarrier(carriertypestring);
        carrierInfo.setGroupfield(groupfield);
        return carrierInfo;
    }
}
```

reduce 端统计运营商个数

```java
package com.youfan.reduce;

import com.youfan.entity.CarrierInfo;
import com.youfan.entity.YearBase;
import org.apache.flink.api.common.functions.ReduceFunction;

/**
 * Created by li on 2019/1/5.
 */
public class CarrierReduce implements ReduceFunction<CarrierInfo>{

    @Override
    public CarrierInfo reduce(CarrierInfo carrierInfo, CarrierInfo t1) throws Exception {
        String carrier = carrierInfo.getCarrier();
        Long count1 = carrierInfo.getCount();
        Long count2 = t1.getCount();

        CarrierInfo carrierInfofinal = new CarrierInfo();
        carrierInfofinal.setCarrier(carrier);
        carrierInfofinal.setCount(count1+count2);
        return carrierInfofinal;
    }
}

```

Flink 类：

```java
package com.youfan.task;

import com.youfan.entity.CarrierInfo;
import com.youfan.entity.YearBase;
import com.youfan.map.CarrierMap;
import com.youfan.map.YearBaseMap;
import com.youfan.reduce.CarrierReduce;
import com.youfan.reduce.YearBaseReduce;
import com.youfan.util.MongoUtils;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.api.java.utils.ParameterTool;
import org.bson.Document;

import java.util.List;

/**
 * Created by li on 2019/1/5.
 */
public class CarrierTask {
    public static void main(String[] args) {
        final ParameterTool params = ParameterTool.fromArgs(args);

        // set up the execution environment
        final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

        // make parameters available in the web interface
        env.getConfig().setGlobalJobParameters(params);

        // get input data
        DataSet<String> text = env.readTextFile(params.get("input"));

        DataSet<CarrierInfo> mapresult = text.map(new CarrierMap());
        DataSet<CarrierInfo> reduceresutl = mapresult.groupBy("groupfield").reduce(new CarrierReduce());
        try {
            List<CarrierInfo> reusltlist = reduceresutl.collect();
            for(CarrierInfo carrierInfo:reusltlist){
                    String carrier = carrierInfo.getCarrier();
                    Long count = carrierInfo.getCount();

                Document doc = MongoUtils.findoneby("carrierstatics","youfanPortrait",carrier);
                if(doc == null){
                    doc = new Document();
                    doc.put("info",carrier);
                    doc.put("count",count);
                }else{
                    Long countpre = doc.getLong("count");
                    Long total = countpre+count;
                    doc.put("count",total);
                }
                MongoUtils.saveorupdatemongo("carrierstatics","youfanPortrait",doc);
            }
            env.execute("carrier analy");
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}

```



