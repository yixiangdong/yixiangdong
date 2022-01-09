---
layout: ceshi28
title: 实时用户画像系统(二十四)vuejs整合前端查询接口
date: 2020-03-31 13:03:05
tags: 机器学习
comments: true
---

### 整合前端查询接口

#### 在components添加baseyear.vue页面以及逻辑

这里的 :option="option"的代码和option: 代码以及xAxis和series是我们需要在图表中显示的， created()方法会请求数据，图表的数据就会显示出来。

<!--more-->

```vue
<template>
    <div>
      <x-chart id="high" class="high" :option="option"></x-chart>
    </div>
</template>

<script>
  // 导入chart组件
  var myvue = {};
  import XChart from './charts'
  export default {
    data() {
      return {
        option:{
          chart: {
            type: 'column'
          },
          title: {
            text: '月平均降雨量'
          },
          subtitle: {
            text: '数据来源: WorldClimate.com'
          },
          xAxis: {
            categories: [
              '一月','二月','三月','四月','五月','六月','七月','八月','九月','十月','十一月','十二月'
            ],
            crosshair: true
          },
          yAxis: {
            min: 0,
            title: {
              text: '降雨量 (mm)'
            }
          },
          series: [{
            name: '东京',
            data: [49.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5,500, 194.1, 95.6, 54.4]
          }]
        },
      }
    },
    beforeCreate:function(){
      myvue = this;
    },
    created() {
      //新增用户
      this.$http.post('http://127.0.0.1:8764/mongoData/resultinfoView',{
          "type": "yearBase"
        }).then((response) => {
          this.option = {
                        chart: {
                          type: 'column'
                        },
                        title: {
                          text: '年代趋势'
                        },
                        xAxis: {
                          categories: response.body.infolist,
                          crosshair: true
                        },
                        yAxis: {
                          min: 0,
                          title: {
                            text: '数量'
                          }
                        },
                        series: [{
                          name: '年代',
                          data: response.body.countlist
                        }]
          };
      });
    },
    components: {
      XChart
    }
  }
</script>

<style scoped>

</style>
```

#### router路由设置

components 模块中的vue页面在router中都对应一个路由

打开router->index.js

```vue
import baseyear from '../components/baseyear.vue'
...
const routes = [
  { path: '/',name:"home",component: home},
  { path: '/highcharts',name:"highcharts",component: highcharts},
  { path: '/baseyear',name:"baseyear",component: baseyear},
  ...
  ]
```

在后端接口添加跨域注解，以免发生跨域问题

把注解@CrossOrigin添加到MongoDataViewControl接口类上即可~！

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

