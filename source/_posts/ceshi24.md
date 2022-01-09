---
layout: ceshi24
title: 实时用户画像系统(二十)vue.js+highcharts构建图表2
date: 2020-03-30 20:46:06
tags: 机器学习
comments: true
---

### 在项目中引入highcharts图表

该文章适合有一定基础的vue语法基础的童鞋学习，语法在这里不再讲述。

<!--more-->

#### 项目components结构

![项目components结构](1585621334382.png)

各个部分的代码如下:

##### charts.vue

作用: charts页面调用HighCharts显示图像

```javascript
<template>
  <div class="x-bar">
    <div :id="id"  :option="option"></div>
  </div>
</template>
<script>
  import HighCharts from 'highcharts'
  export default {
    // 验证类型
    props: {
      id: {
        type: String
      },
      option: {
        type: Object
      }
    },
    watch: {
      option () {
        HighCharts.chart(this.id,this.option);
      }
    },
    mounted() {
      HighCharts.chart(this.id,this.option)
    }
  }
</script>
```

##### highcharts.vue

作用: 定义图像各个维度（标题，副标题，列，x轴,y轴）

```javascript
<template>
<div>
  <x-chart id="high" class="high" :option="option1"></x-chart>
 </div>
</template>
<script>
  // 导入chart组件
  var myvue = {};
  import XChart from './charts'
  export default {
    data() {
      return {
        option1:{
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
          tooltip: {
            // head + 每个 point + footer 拼接成完整的 table
            headerFormat: '<span style="font-size:10px">{point.key}</span><table>',
            pointFormat: '<tr><td style="color:{series.color};padding:0">{series.name}: </td>' +
              '<td style="padding:0"><b>{point.y:.1f} mm</b></td></tr>',
            footerFormat: '</table>',
            shared: true,
            useHTML: true
          },
          plotOptions: {
            column: {
              borderWidth: 0
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
    mounted:function(){
      myvue.other.title.text = '2010 ~ 2016 年太阳能行业就业人员发展情况';
      myvue.other.subtitle.text = '数据来源：thesolarfoundation.com';
      myvue.other.series = myvue.data;//数据
      myvue.other.yAxis.title.text = '就业人数'; //数据
      myvue.option = myvue.other;
    },
    components: {
      XChart
    }
  }
</script>
```

#### 路由router设置

```vue
import Vue from 'vue'
import VueRouter from 'vue-router'
import home from '../components/home.vue'
import index from '../index.vue'

import store from '../store/index.js'
import VueResource from 'vue-resource'
import highcharts from '../components/highcharts.vue'
//highcharts的引入
Vue.use(VueResource)
Vue.use(VueRouter)

/* eslint-disable no-new */
// new Vue({
//   el: '#app',
//   render: h => h(App)
// })


// 0. 如果使用模块化机制编程， 要调用 Vue.use(VueRouter)

// 1. 定义（路由）组件。
// 可以从其他文件 import 进来
// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
// 我们晚点在讨论嵌套路由。
const routes = [
  { path: '/',name:"home",component: home},
  { path: '/highcharts',name:"highcharts",component: highcharts}
  ]
// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数, 不过先这么简单着吧。
const router = new VueRouter({
  mode: 'history',
  routes // （缩写）相当于 routes: routes
})

// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  store,
  router,
  render: h => h(index)
}).$mount('#app')

// 现在，应用已经启动了！
```

#### 前端显示结果

npm run dev

http://localhost:8081/highcharts

![前端显示结果](1585622687778.png)

