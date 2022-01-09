---
layout: ceshi14
title: 实时用户画像系统(九) 实时品牌偏好
date: 2020-03-27 17:07:13
tags: 机器学习
password: 2020-03-27!!
abstract: 该文章已加密, 请加博主微信号:books_111索要密码查看。
message: 该文章已加密, 请加博主微信号:books_111索要密码查看。
wrong_pass_message: 密码不正确，请重新输入！
wrong_hash_message: 文章不能被校验, 不过您还是能看看解密后的内容！
comments: true
---

#### 添加品牌偏好实体类

```java
public class BrandLike {
    private String brand;
    private long count;
    private String groupbyfield;

    public String getGroupbyfield() {
        return groupbyfield;
    }

    public void setGroupbyfield(String groupbyfield) {
        this.groupbyfield = groupbyfield;
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public long getCount() {
        return count;
    }

    public void setCount(long count) {
        this.count = count;
    }
}

```

<!-- more -->

#### 添加品牌日志结构类

之前的文章已经提到并已经添加,详细请看历史文章:  https://etop.work/2020/03/27/ceshi11/

#### 添加map类处理读取的数据

```java
import com.alibaba.fastjson.JSONObject;
import com.youfan.entity.BrandLike;
import com.youfan.kafka.KafkaEvent;
import com.youfan.log.ScanProductLog;
import com.youfan.util.HbaseUtil;
import com.youfan.utils.MapUtils;
import org.apache.commons.lang.StringUtils;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.util.Collector;

import java.util.HashMap;
import java.util.Map;

/**
 * Created by li on 2019/1/6.
 */
public class BrandLikeMap implements FlatMapFunction<KafkaEvent, BrandLike>  {

    @Override
    public void flatMap(KafkaEvent kafkaEvent, Collector<BrandLike> collector) throws Exception {
            String data = kafkaEvent.getWord();
            ScanProductLog scanProductLog = JSONObject.parseObject(data,ScanProductLog.class);
            int userid = scanProductLog.getUserid();
            String brand = scanProductLog.getBrand();
            String tablename = "userflaginfo";
            String rowkey = userid+"";
            String famliyname = "userbehavior";
            String colum = "brandlist";//运营
            String mapdata = HbaseUtil.getdata(tablename,rowkey,famliyname,colum);
            Map<String,Long> map = new HashMap<String,Long>();
            if(StringUtils.isNotBlank(mapdata)){
                map = JSONObject.parseObject(mapdata,Map.class);
            }
            //获取之前的品牌偏好
            String maxprebrand = MapUtils.getmaxbyMap(map);

            long prebarnd = map.get(brand)==null?0l:map.get(brand);
            map.put(brand,prebarnd+1);
            String finalstring = JSONObject.toJSONString(map);
            HbaseUtil.putdata(tablename,rowkey,famliyname,colum,finalstring);

            String maxbrand = MapUtils.getmaxbyMap(map);
            if(StringUtils.isNotBlank(maxbrand)&&!maxprebrand.equals(maxbrand)){
                BrandLike brandLike = new BrandLike();
                brandLike.setBrand(maxprebrand);
                brandLike.setCount(-1l);
                brandLike.setGroupbyfield("==brandlik=="+maxprebrand);
                collector.collect(brandLike);
            }

            BrandLike brandLike = new BrandLike();
            brandLike.setBrand(maxbrand);
            brandLike.setCount(1l);
            collector.collect(brandLike);
            brandLike.setGroupbyfield("==brandlik=="+maxbrand);
            colum = "brandlike";
            HbaseUtil.putdata(tablename,rowkey,famliyname,colum,maxbrand);
    }

}
```

#### 添加reduce类统计品牌偏好数量

```java
import com.youfan.entity.BrandLike;
import com.youfan.entity.CarrierInfo;
import org.apache.flink.api.common.functions.ReduceFunction;

/**
 * Created by li on 2019/1/6.
 */
public class BrandLikeReduce implements ReduceFunction<BrandLike> {
    @Override
    public BrandLike reduce(BrandLike brandLike, BrandLike t1) throws Exception {
        String brand = brandLike.getBrand();
        long count1 = brandLike.getCount();
        long count2 = t1.getCount();
        BrandLike brandLikefinal = new BrandLike();
        brandLikefinal.setBrand(brand);
        brandLikefinal.setCount(count1+count2);
        return brandLikefinal;
    }
}
```

#### 添加sink类保存结果到mongodb

```java
import com.youfan.entity.BrandLike;
import com.youfan.util.MongoUtils;
import org.apache.flink.streaming.api.functions.sink.SinkFunction;
import org.bson.Document;

/**
 * Created by li on 2019/1/6.
 */
public class BrandLikeSink implements SinkFunction<BrandLike> {
    @Override
    public void invoke(BrandLike value, Context context) throws Exception {
        String brand = value.getBrand();
        long count = value.getCount();
        Document doc = MongoUtils.findoneby("brandlikestatics","youfanPortrait",brand);
        if(doc == null){
            doc = new Document();
            doc.put("info",brand);
            doc.put("count",count);
        }else{
            Long countpre = doc.getLong("count");
            Long total = countpre+count;
            doc.put("count",total);
        }
        MongoUtils.saveorupdatemongo("brandlikestatics","youfanPortrait",doc);
    }
}

```

