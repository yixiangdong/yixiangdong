---
layout: ceshi24
title: 实时用户画像系统(十九)vue.js+highcharts构建图表1
date: 2020-03-30 20:45:56
tags: 机器学习
comments: true
---

### 安装highcharts

npm install highcharts --save

<!--more-->

### 访问highcharts官网以及演示图表

<https://www.highcharts.com.cn/demo/highcharts>

![highcharts官网首页](1585572638136.png)

#### 基础折线图

使用方法: 图中红圈区域是重点需要了解的地方，只需要把js和html代码复制到项目里面去就可以了。

![折线图](1585572699158.png)

#### 加入bootstrap css样式

下载bootstrap 3.3.6

![bootstrap官网下载页面](1585574110442.png)

![加载bootstrap 静态样式](1585574188815.png)

项目依赖package.json

```json
"dependencies": {
  "highcharts": "^7.0.0",
  "vue": "^2.0.1",
  "vue-highcharts": "0.0.11",
  "vue-resource": "^1.5.1"
},
"devDependencies": {
  "autoprefixer": "^6.4.0",
  "babel-core": "^6.0.0",
  "babel-loader": "^6.0.0",
  "babel-plugin-transform-runtime": "^6.0.0",
  "babel-preset-es2015": "^6.0.0",
  "babel-preset-stage-2": "^6.0.0",
  "babel-register": "^6.0.0",
  "chai": "^3.5.0",
  "connect-history-api-fallback": "^1.1.0",
  "css-loader": "^0.25.0",
  "eventsource-polyfill": "^0.9.6",
  "express": "^4.13.3",
  "extract-text-webpack-plugin": "^1.0.1",
  "file-loader": "^0.9.0",
  "function-bind": "^1.0.2",
  "html-webpack-plugin": "^2.8.1",
  "http-proxy-middleware": "^0.17.2",
  "inject-loader": "^2.0.1",
  "isparta-loader": "^2.0.0",
  "json-loader": "^0.5.4",
  "karma": "^1.3.0",
  "karma-coverage": "^1.1.1",
  "karma-mocha": "^1.2.0",
  "karma-phantomjs-launcher": "^1.0.0",
  "karma-sinon-chai": "^1.2.0",
  "karma-sourcemap-loader": "^0.3.7",
  "karma-spec-reporter": "0.0.26",
  "karma-webpack": "^1.7.0",
  "less-loader": "^2.2.3",
  "lolex": "^1.4.0",
  "mocha": "^3.1.0",
  "opn": "^4.0.2",
  "ora": "^0.3.0",
  "phantomjs-prebuilt": "^2.1.3",
  "shelljs": "^0.7.4",
  "sinon": "^1.17.3",
  "sinon-chai": "^2.8.0",
  "style-loader": "^0.13.1",
  "stylus": "^0.54.5",
  "stylus-loader": "^3.0.2",
  "url-loader": "^0.5.7",
  "vue-loader": "^9.4.0",
  "vue-router": "^2.0.1",
  "vue-style-loader": "^1.0.0",
  "vuex": "^2.0.0",
  "webpack": "^1.13.2",
  "webpack-dev-middleware": "^1.8.3",
  "webpack-hot-middleware": "^2.12.2",
  "webpack-merge": "^0.14.1"
}
```



### 运行项目并访问

![运行项目](1585574342773.png)

![运行dev](1585574377176.png)

也可以npm install之后 运行命令npm run dev

![运行日志](1585574469880.png)

http://localhost:8080

![项目页面](1585574429053.png)

