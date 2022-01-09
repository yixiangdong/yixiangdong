---
title: 阿里云Dataworks+Maxcompute实时计算数据仓库方案之用户行为数仓搭建(一)
date: 2020-03-19 11:12:23
tags: 数据仓库
comments: true
---


![业务数仓系统](1584646112295.png)

## 数据仓库概念

### 数据仓库定义(Data Warehouse)

是为企业所有决策制定过程，提供所有系统数据支持的战略集合。
数据仓库好处：可以帮助企业， 改进业务流程、 控制成本、 提高产品质量等。
数据仓库做什么： 清洗，转义，分类，重组，合并，拆分，统计等。
数据仓库输出到哪： 报表系统、用户画像、推荐系统、机器学习、风控系统等
<!-- more -->
## 项目需求分析 

1. 采集埋点日志数据 
2. 采集业务数据库中数据 
3. 数据仓库的搭建（用户行为数仓、业务数仓） 
4. 分析统计业务指标 
5. 对结果进行可视化展示 

## 项目框架 

阿里云产品 简介 类比

![阿里云技术框架](阿里云技术框架.png)

## 系统数据流程设计 

![架构图](architecher.png)

## 技术选型 

![技术选型](技术选型.png)

## 数据生成模块 

### 埋点数据基本格式 

1）公共字段：基本所有安卓手机都包含的字段 

2） 业务字段：埋点上报的字段，有具体的业务类型 

下面就是一个示例，表示业务字段的上传。 

{
"ap":"xxxxx",//项目数据来源 app pc 

"cm": { //公共字段
"mid": "", // (String) 设备唯一标识
"uid": "", // (String) 用户标识
"vc": "1", // (String) versionCode，程序版本号
"vn": "1.0", // (String) versionName，程序版本名
"l": "zh", // (String) language 系统语言
"sr": "", // (String) 渠道号，应用从哪个渠道来的。
"os": "7.1.1", // (String) Android 系统版本
"ar": "CN", // (String) area 区域
"md": "BBB100-1", // (String) model 手机型号
"ba": "blackberry", // (String) brand 手机品牌
"sv": "V2.2.1", // (String) sdkVersion
"g": "", // (String) gmail
"hw": "1620x1080", // (String) heightXwidth，屏幕宽高
"t": "1506047606608", // (String) 客户端日志产生时的时间
"nw": "WIFI", // (String) 网络模式
"ln": 0, // (double) lng 经度
"la": 0 // (double) lat 纬度
},
"et": [ //事件
{
"ett": "1506047605364", //客户端事件产生时间
"en": "display", //事件名称
"kv": { //事件结果，以 key-value 形式自行定义
"goodsid": "236",
"action": "1",
"extend1": "1",
"place": "2",
"category": "75"
}
}
]
} 

示例日志（服务器时间戳 | 日志） ： 

1540934156385|{
"ap": "gmall",
"cm": {
"uid": "1234",
"vc": "2",
"vn": "1.0",
"la": "EN",
"sr": "",
"os": "7.1.1",
"ar": "CN",
"md": "BBB100-1",
"ba": "blackberry",
"sv": "V2.2.1",
"g": "abc@gmail.com",
"hw": "1620x1080",
"t": "1506047606608",
"nw": "WIFI",
"ln": 0
},
"et":  [ 

{
"ett": "1506047605364", //客户端事件产生时间
"en": "display", //事件名称
"kv": { //事件结果，以 key-value 形式自行定义
"goodsid": "236",
"action": "1",
"extend1": "1",
"place": "2",
"category": "75"
}
},{
"ett": "1552352626835",
"en": "error",
"kv": {
"errorBrief": "错误摘要",
"errorDetail": "错误详情"
}
}
]
}

}

下面是各个埋点日志格式。 其中商品点击属于信息流的范畴
3.2 事件日志数据
3.2.1 商品列表页（loading）
事件名称： loading 

| 标签         | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| action       | 动作：开始加载=1，加载成功=2，加载失败=3                     |
| loading_time | 加载时长：计算下拉开始到接口返回数据的时间，（开始加载报 0，加载成 功或加载失败才上报时间） |
| loading_way  | 加载类型： 1-读取缓存， 2-从接口拉新数据 （加载成功才上报加载类型） |
| extend1      | 扩展字段 Extend1                                             |
| extend2      | 扩展字段 Extend2                                             |
| type         | 加载类型：自动加载=1，用户下拽加载=2，底部加载=3（底部条触发点击 底部提示条/点击返回顶部加载） |
| type1        | 加载失败码：把加载失败状态码报回来（报空为加载成功，没有失败） |

商品曝光（display） 

| 标签     | 含义                                                 |
| -------- | ---------------------------------------------------- |
| action   | 动作：曝光商品=1，点击商品=2，                       |
| goodsid  | 商品 ID（服务端下发的 ID）                           |
| place    | 顺序（第几条商品，第一条为 0，第二条为 1，如此类推） |
| extend1  | 曝光类型： 1 - 首次曝光 2-重复曝光                   |
| category | 分类 ID（服务端定义的分类 ID）                       |

商品详情页 

| action        | 动作：开始加载=1，加载成功=2（ pv），加载失败=3, 退出页面=4  |
| ------------- | ------------------------------------------------------------ |
| goodsid       | 商品 ID（服务端下发的 ID）                                   |
| show_style    | 商品样式： 0、无图、 1、一张大图、 2、两张图、 3、三张小图、 4、一张小图、 5、 一张大图两张小图 |
| news_staytime | 页面停留时长：从商品开始加载时开始计算，到用户关闭页面所用的时间。若中途 用跳转到其它页面了，则暂停计时，待回到详情页时恢复计时。或中途划出的时间 超过 10 分钟，则本次计时作废，不上报本次数据。如未加载成功退出，则报空。 |
| loading_time  | 加载时长：计算页面开始加载到接口返回数据的时间 （开始加载报 0，加载成功或 加载失败才上报时间） |
| type1         | 加载失败码：把加载失败状态码报回来（报空为加载成功，没有失败） |
| category      | 分类 ID（服务端定义的分类 ID）                               |

购物车（cart） 

| 标签       | 含义                                                  |
| ---------- | ----------------------------------------------------- |
| itemid     | 商品                                                  |
| action     | 操作类型： 1 添加购物车； 2 改变商品数量； 3 移除商品 |
| change_num | 加减数量                                              |
| before_num | 更改前数量                                            |
| after_num  | 更改后数量                                            |
| price      | 商品单价                                              |

广告（ad） 

| 标签       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| entry      | 入口：商品列表页=1 应用首页=2 商品详情页=3                   |
| action     | 动作：请求广告=1 取缓存广告=2 广告位展示=3 广告展示=4 广告点击=5 |
| content    | 状态：成功=1 失败=2                                          |
| detail     | 失败码（没有则上报空）                                       |
| source     | 广告来源:admob=1 facebook=2 ADX（百度） =3 VK（头条） =4     |
| behavior   | 用户行为： 主动获取广告=1 被动获取广告=2                     |
| newstype   | Type: 1- 图文 2-图集 3-段子 4-GIF 5-视频 6-调查 7-纯文 8-视频+图文 9-GI F+图文 0-其他 |
| show_style | 内容样式：无图(纯文字)=6 一张大图=1 三站小图+文=4 一张小图=2 一张大图 两张小图+文=3 图集+文 = 5 一张大图+文=11 GIF 大图+文=12 视频(大图)+文 = 13 来源于详情页相关推荐的商品，上报样式都为 0（因为都是左文右图） |

消息通知（notification） 

| 标签    | 含义                                                         |
| ------- | ------------------------------------------------------------ |
| action  | 动作：通知产生=1，通知弹出=2，通知点击=3，常驻通知展示（不重复上报， 一天之内只报一次） =4 |
| type    | 通知 id：预警通知=1，天气预报（早=2，晚=3），常驻=4          |
| ap_time | 客户端弹出时间                                               |
| content | 备用字段                                                     |

评论（comment） 

| 标签         | 含义                                            | 字段类型 | 长度 | 允许空 | 缺省值 |
| ------------ | ----------------------------------------------- | -------- | ---- | ------ | ------ |
| comment_id   | 评论表                                          | int      | 10,0 |        |        |
| userid       | 用户 id                                         | int      | 10,0 | √      | 0      |
| p_comment_id | 父级评论 id(为 0 则是一 级评论,不为 0 则是回复) | int      | 10,0 | √      |        |
| content      | 评论内容                                        | string   | 1000 | √      |        |
| addtime      | 创建时间                                        | string   | √    |        |        |
| other_id     | 评论的相关 id                                   | int      | 10,0 | √      |        |
| praise_count | 点赞数量                                        | int      | 10,0 | √      | 0      |
| reply_count  | 回复数量                                        | int      | 10,0 | √      | 0      |

收藏（favorites） 

| 标签 | 含义 | 字段类型 | 长度 | 允许空 | 缺省值 |
| ---- | ---- | -------- | ---- | ------ | ------ |
| id   | 主键 | int      | 10,0 |        |        |

| course_id | 商品 id  | int    | 10,0 | √    | 0    |
| --------- | -------- | ------ | ---- | ---- | ---- |
| userid    | 用户 ID  | int    | 10,0 | √    | 0    |
| add_time  | 创建时间 | string | √    |      |      |

点赞（praise） 

| 标签      | 含义                                                        | 字段类型 | 长度 | 允许空 | 缺省值 |
| --------- | ----------------------------------------------------------- | -------- | ---- | ------ | ------ |
| id        | 主键 id                                                     | int      | 10,0 |        |        |
| userid    | 用户 id                                                     | int      | 10,0 | √      |        |
| target_id | 点赞的对象 id                                               | int      | 10,0 | √      |        |
| type      | 点赞类型 1 问答点赞 2 问 答评论点赞 3 文章点赞数 4 评论点赞 | int      | 10,0 | √      |        |
| add_time  | 添加时间                                                    | string   | √    |        |        |

错误日志（error） 

| 标签        | 含义     |
| ----------- | -------- |
| errorBrief  | 错误摘要 |
| errorDetail | 错误详情 |

启动日志数据（start） 

| 标签         | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| entry        | 入 口 ： push=1 ， widget=2 ， icon=3 ， notification=4, lockscreen_widget =5 |
| open_ad_type | 开屏广告类型: 开屏原生广告=1, 开屏插屏广告=2                 |
| action       | 状态：成功=1 失败=2                                          |
| loading_time | 加载时长：计算下拉开始到接口返回数据的时间，（开始加载报 0，加载 成功或加载失败才上报时间） |
| detail       | 失败码（没有则上报空）                                       |
| extend1      | 失败的 message（没有则上报空）                               |
| en           | 日志类型 start                                               |

## 数据生成脚本 

### 创建 Maven 工程 

1） 创建 log-collector 

![Maven项目](idea.png)

2）创建一个包名： com.atguigu.appclient
3） 在 com.atguigu.appclient 包下创建一个类， AppMain。
4） 在 pom.xml 文件中添加如下内容 

```xml
<!--版本号统一-->
<properties>
<slf4j.version>1.7.20</slf4j.version>
<logback.version>1.0.7</logback.version>
</properties>
<dependencies>
<!--阿里巴巴开源 json 解析框架-->
<dependency>
<groupId>com.alibaba</groupId>
<artifactId>fastjson</artifactId>
<version>1.2.51</version>
</dependency>
<!--日志生成框架-->
<dependency>
<groupId>ch.qos.logback</groupId>
<artifactId>logback-core</artifactId>
<version>${logback.version}</version>
</dependency>
<dependency>
<groupId>ch.qos.logback</groupId>
<artifactId>logback-classic</artifactId>
<version>${logback.version}</version>
</dependency>
</dependencies> 

<!--编译打包插件-->
<build>
<plugins>
<plugin>
<artifactId>maven-compiler-plugin</artifactId>
<version>2.3.2</version>
<configuration>
<source>1.8</source>
<target>1.8</target>
</configuration>
</plugin>
<plugin>
<artifactId>maven-assembly-plugin </artifactId>
<configuration>
<descriptorRefs>
<descriptorRef>jar-with-dependencies</descriptorRef>
</descriptorRefs>
<archive>
<manifest>
<mainClass>com.atguigu.appclient.AppMain</mainClass>
</manifest>
</archive>
</configuration>
<executions>
<execution>
<id>make-assembly</id>
<phase>package</phase>
<goals>
<goal>single</goal>
</goals>
</execution>
</executions>
</plugin>
</plugins>
</build>
```



注意： com.atguigu.appclient.AppMain 要和自己建的全类名一致。 

### 公共字段 Bean 

1） 创建包名： com.atguigu.bean
2） 在 com.atguigu.bean 包下依次创建如下 bean 对象 

```java
package com.atguigu.bean;
/**

*公共日志
*/
public class AppBase{
private String mid; // (String) 设备唯一标识
private String uid; // (String) 用户 uid
private String vc; // (String) versionCode，程序版本号
private String vn; // (String) versionName，程序版本名
private String l; // (String) 系统语言
private String sr; // (String) 渠道号，应用从哪个渠道来的。
private String os; // (String) Android 系统版本
private String ar; // (String) 区域
private String md; // (String) 手机型号
private String ba; // (String) 手机品牌
private String sv; // (String) sdkVersion
private String g; // (String) gmail 

private String hw; // (String) heightXwidth，屏幕宽高
private String t; // (String) 客户端日志产生时的时间
private String nw; // (String) 网络模式
private String ln; // (double) lng 经度
private String la; // (double) lat 纬度
public String getMid() {
return mid;
}
public void setMid(String mid) {
this.mid = mid;
}
public String getUid() {
return uid;
}
public void setUid(String uid) {
this.uid = uid;
}
public String getVc() {
return vc;
}
public void setVc(String vc) {
this.vc = vc;
}
public String getVn() {
return vn;
}
public void setVn(String vn) {
this.vn = vn;
}
public String getL() {
return l;
}
public void setL(String l) {
this.l = l;
}
public String getSr() {
return sr;
}
public void setSr(String sr) {
this.sr = sr;
}
public String getOs() {
return os;
}
public void setOs(String os) {
this.os = os;
}
public String getAr() { 

return ar;
}
public void setAr(String ar) {
this.ar = ar;
}
public String getMd() {
return md;
}
public void setMd(String md) {
this.md = md;
}
public String getBa() {
return ba;
}
public void setBa(String ba) {
this.ba = ba;
}
public String getSv() {
return sv;
}
public void setSv(String sv) {
this.sv = sv;
}
public String getG() {
return g;
}
public void setG(String g) {
this.g = g;
}
public String getHw() {
return hw;
}
public void setHw(String hw) {
this.hw = hw;
}
public String getT() {
return t;
}
public void setT(String t) {
this.t = t;
}
public String getNw() {
return nw;
}
public void setNw(String nw) {
this.nw = nw;
} 
public String getLn() {
return ln;
}
public void setLn(String ln) {
this.ln = ln;
}
public String getLa() {
return la;
}
public void setLa(String la) {
this.la = la;
}
}

```

### 启动日志 Bean 

```java
package com.atguigu.bean;
/**
* 启动日志
*/
public class AppStart extends AppBase {
private String entry;//入口： push=1， widget=2， icon=3， notification=4,
lockscreen_widget =5
private String open_ad_type;//开屏广告类型: 开屏原生广告=1, 开屏插屏广告=2
private String action;//状态：成功=1 失败=2
private String loading_time;//加载时长：计算下拉开始到接口返回数据的时间，（开
始加载报 0，加载成功或加载失败才上报时间）
private String detail;//失败码（没有则上报空）
private String extend1;//失败的 message（没有则上报空）
private String en;//启动日志类型标记
public String getEntry() {
return entry;
}
public void setEntry(String entry) {
this.entry = entry;
}
public String getOpen_ad_type() {
return open_ad_type;
}
public void setOpen_ad_type(String open_ad_type) {
this.open_ad_type = open_ad_type;
}
public String getAction() {
return action;
}
public void setAction(String action) {
this.action = action;
}
public String getLoading_time() {
return loading_time;
}
public void setLoading_time(String loading_time) {
this.loading_time = loading_time;
}
public String getDetail() {
return detail;
}
public void setDetail(String detail) {
this.detail = detail;
}
public String getExtend1() {
return extend1;
}
public void setExtend1(String extend1) {
this.extend1 = extend1;
}
public String getEn() {
return en;
}
public void setEn(String en) {
this.en = en;
}
}
```

### 错误日志 Bean 

```java
package com.atguigu.bean;
/**
* 错误日志
*/
public class AppError {
private String errorBrief; //错误摘要
private String errorDetail; //错误详情
public String getErrorBrief() {
return errorBrief;
}
public void setErrorBrief(String errorBrief) {
this.errorBrief = errorBrief;
}
public String getErrorDetail() {
return errorDetail;
}
public void setErrorDetail(String errorDetail) {
this.errorDetail = errorDetail;
}
}
```

### 事件日志 Bean 之商品曝光 

```java
package com.atguigu.bean;
/**
*
* 商品点击日志 
 */
  public class AppDisplay {
  private String action;//动作：曝光商品=1，点击商品=2，
  private String goodsid;//商品 ID（服务端下发的 ID）
  private String place;//顺序（第几条商品，第一条为 0，第二条为 1，如此类推）
  private String extend1;//曝光类型： 1 - 首次曝光 2-重复曝光（没有使用）
  private String category;//分类 ID（服务端定义的分类 ID）
  public String getAction() {
  return action;
  }
  public void setAction(String action) {
  this.action = action;
  }
  public String getGoodsid() {
  return goodsid;
  }
  public void setGoodsid(String goodsid) {
  this.goodsid = goodsid;
  }
  public String getPlace() {
  return place;
  }
  public void setPlace(String place) {
  this.place = place;
  }
  public String getExtend1() {
  return extend1;
  }
  public void setExtend1(String extend1) {
  this.extend1 = extend1;
  }
  public String getCategory() {
  return category;
  }
  public void setCategory(String category) {
  this.category = category;
  }
  } 
```

### 事件日志 Bean 之商品详情页 

```java
package com.atguigu.bean;
/**
* 商品详情
*/
public class AppNewsDetail {
private String entry;//页面入口来源：应用首页=1、 push=2、详情页相关推荐=3
private String action;//动作：开始加载=1，加载成功=2（pv），加载失败=3, 退出页
面=4
private String goodsid;//商品 ID（服务端下发的 ID）
private String showtype;//商品样式： 0、无图 1、一张大图 2、两张图 3、三张小图 4、
一张小图 5、一张大图两张小图 来源于详情页相关推荐的商品，上报样式都为 0（因为都是左文
右图）
private String news_staytime;//页面停留时长：从商品开始加载时开始计算，到用户
关闭页面所用的时间。若中途用跳转到其它页面了，则暂停计时，待回到详情页时恢复计时。或中途
划出的时间超过 10 分钟，则本次计时作废，不上报本次数据。如未加载成功退出，则报空。
private String loading_time;//加载时长：计算页面开始加载到接口返回数据的时间
（开始加载报 0，加载成功或加载失败才上报时间）
private String type1;//加载失败码：把加载失败状态码报回来（报空为加载成功，没有
失败）
private String category;//分类 ID（服务端定义的分类 ID）
public String getEntry() {
return entry;
}
public void setEntry(String entry) {
this.entry = entry;
}
public String getAction() {
return action;
}
public void setAction(String action) {
this.action = action;
}
public String getGoodsid() {
return goodsid;
}
public void setGoodsid(String goodsid) {
this.goodsid = goodsid;
}
public String getShowtype() {
return showtype;
}
public void setShowtype(String showtype) {
this.showtype = showtype;
}
public String getNews_staytime() {
return news_staytime;
}
public void setNews_staytime(String news_staytime) {
this.news_staytime = news_staytime;
}
public String getLoading_time() {
return loading_time;
}
public void setLoading_time(String loading_time) {
this.loading_time = loading_time;
}
public String getType1() {
return type1;
}
public void setType1(String type1) {
this.type1 = type1;
}
public String getCategory() {
return category;
}
public void setCategory(String category) {
this.category = category;
}
}

```

### 事件日志 Bean 之商品列表页 

```java
package com.atguigu.bean;
/**
* 商品列表
*/
public class AppLoading {
private String action;//动作：开始加载=1，加载成功=2，加载失败=3
private String loading_time;//加载时长：计算下拉开始到接口返回数据的时间，（开
始加载报 0，加载成功或加载失败才上报时间）
private String loading_way;//加载类型： 1-读取缓存， 2-从接口拉新数据 （加载
成功才上报加载类型）
private String extend1;//扩展字段 Extend1
private String extend2;//扩展字段 Extend2
private String type;//加载类型：自动加载=1，用户下拽加载=2，底部加载=3（底部条
触发点击底部提示条/点击返回顶部加载）
private String type1;//加载失败码：把加载失败状态码报回来（报空为加载成功，没有
失败）
public String getAction() {
return action;
}
public void setAction(String action) {
this.action = action;
}
public String getLoading_time() {
return loading_time;
}
public void setLoading_time(String loading_time) {
this.loading_time = loading_time;
}
public String getLoading_way() {
return loading_way;
}
public void setLoading_way(String loading_way) {
this.loading_way = loading_way;
}
public String getExtend1() {
return extend1;
}
public void setExtend1(String extend1) {
this.extend1 = extend1;
}
public String getExtend2() {
return extend2;
}
public void setExtend2(String extend2) {
this.extend2 = extend2;
}
public String getType() {
return type;
}
public void setType(String type) {
this.type = type;
}
public String getType1() {
return type1;
}
public void setType1(String type1) {
this.type1 = type1;
}
}    
```

### 事件日志 Bean 之购物车 

```java
package com.atguigu.bean;
/**
* 购物车
*/
public class AppCart {
int itemid;
int action; // 1 添加产品进购物车 2 调整购物车数量
int changeNum; // 数量变化
int beforeNum; // 变化前数量
int afterNum; // 变化后数量
Double price; // 加入购物车时的单价
public int getItemid() {
return itemid;
}
public void setItemid(int itemid) {
this.itemid = itemid;
}
public int getAction() {
return action;
}
public void setAction(int action) {
this.action = action;
}
public int getChangeNum() {
return changeNum;
}
public void setChangeNum(int changeNum) {
this.changeNum = changeNum;
}
public int getBeforeNum() {
return beforeNum;
}
public void setBeforeNum(int beforeNum) {
this.beforeNum = beforeNum;
}
public int getAfterNum() {
return afterNum;
}
public void setAfterNum(int afterNum) {
this.afterNum = afterNum;
}
public Double getPrice() {
return price;
}
public void setPrice(Double price) {
this.price = price;
}
}
```

### 事件日志 Bean 之广告 

```
package com.atguigu.bean;
/**
* 广告
*/
public class AppAd {
private String entry;//入口：商品列表页=1 应用首页=2 商品详情页=3
private String action;//动作：请求广告=1 取缓存广告=2 广告位展示=3 广告展示
=4 广告点击=5
private String content;//状态：成功=1 失败=2
private String detail;//失败码（没有则上报空）
private String source;//广告来源:admob=1 facebook=2 ADX（百度） =3 VK（俄
罗斯） =4
private String behavior;//用户行为： 主动获取广告=1 被动获取广告=2
private String newstype;//Type: 1- 图文 2-图集 3-段子 4-GIF 5-视频 6-调查
7-纯文 8-视频+图文 9-GIF+图文 0-其他
private String show_style;//内容样式：无图(纯文字)=6 一张大图=1 三站小图+文
=4 一张小图=2 一张大图两张小图+文=3 图集+文 = 5
//一张大图+文=11 GIF 大图+文=12 视频(大图)+文 = 13
//来源于详情页相关推荐的商品，上报样式都为 0（因为都是左文右
图）
public String getEntry() {
return entry;
}
public void setEntry(String entry) {
this.entry = entry;
}
public String getAction() {
return action;
}
public void setAction(String action) {
this.action = action;
}
public String getContent() {
return content;
}
public void setContent(String content) {
this.content = content;
}
public String getDetail() {
return detail;
}
public void setDetail(String detail) {
this.detail = detail;
}
public String getSource() {
return source;
}
public void setSource(String source) {
this.source = source;
}
public String getBehavior() {
return behavior;
}
public void setBehavior(String behavior) {
this.behavior = behavior;
} p
ublic String getNewstype() {
return newstype;
}
public void setNewstype(String newstype) {
this.newstype = newstype;
}
public String getShow_style() {
return show_style;
}
public void setShow_style(String show_style) {
this.show_style = show_style;
}
}
```

### 事件日志 Bean 之消息通知 

```java
package com.atguigu.bean;
/**
* 消息通知日志
*/
public class AppNotification {
private String action;//动作：通知产生=1，通知弹出=2，通知点击=3，常驻通知展示
（不重复上报，一天之内只报一次） =4
private String type;//通知 id：预警通知=1，天气预报（早=2，晚=3），常驻=4
private String ap_time;//客户端弹出时间
private String content;//备用字段
public String getAction() {
return action;
}
public void setAction(String action) {
this.action = action;
}
public String getType() {
return type;
}
public void setType(String type) {
this.type = type;
}
public String getAp_time() {
return ap_time;
}
public void setAp_time(String ap_time) {
this.ap_time = ap_time;
}
public String getContent() {
return content;
}
public void setContent(String content) {
this.content = content;
}
}
```

### 事件日志 Bean 之用户评论 

```
package com.atguigu.bean;
/**
* 评论
*/
public class AppComment {
private int comment_id;//评论表
private int userid;//用户 id
private int p_comment_id;//父级评论 id(为 0 则是一级评论,不为 0 则是回复)
private String content;//评论内容
private String addtime;//创建时间
private int other_id;//评论的相关 id
private int praise_count;//点赞数量
private int reply_count;//回复数量
public int getComment_id() {
return comment_id;
}
public void setComment_id(int comment_id) {
this.comment_id = comment_id;
}
public int getUserid() {
return userid;
}
public void setUserid(int userid) {
this.userid = userid;
}
public int getP_comment_id() {
return p_comment_id;
}
public void setP_comment_id(int p_comment_id) {
this.p_comment_id = p_comment_id;
}
public String getContent() {
return content;
}
public void setContent(String content) {
this.content = content;
}
public String getAddtime() {
return addtime;
}
public void setAddtime(String addtime) {
this.addtime = addtime;
}
public int getOther_id() {
return other_id;
}
public void setOther_id(int other_id) {
this.other_id = other_id;
}
public int getPraise_count() {
return praise_count;
}
public void setPraise_count(int praise_count) {
this.praise_count = praise_count;
}
public int getReply_count() {
return reply_count;
}
public void setReply_count(int reply_count) {
this.reply_count = reply_count;
}
}

```

### 事件日志 Bean 之用户收藏 

```java
package com.atguigu.bean;
/**
* 收藏
*/
public class AppFavorites {
private int id;//主键
private int course_id;//商品 id
private int userid;//用户 ID
private String add_time;//创建时间
public int getId() {
return id;
}
public void setId(int id) {
this.id = id;
}
public int getCourse_id() {
return course_id;
}
public void setCourse_id(int course_id) {
this.course_id = course_id;
}
public int getUserid() {
return userid;
}
public void setUserid(int userid) {
this.userid = userid;
}
public String getAdd_time() {
return add_time;
}
public void setAdd_time(String add_time) {
this.add_time = add_time;
}
}
```

### 事件日志 Bean 之用户点赞 

```java
package com.atguigu.bean;
/**
* 点赞
*/
public class AppPraise {
private int id; //主键 id
private int userid;//用户 id
private int target_id;//点赞的对象 id
private int type;//点赞类型 1 问答点赞 2 问答评论点赞 3 文章点赞数 4 评论点赞
private String add_time;//添加时间
public int getId() {
return id;
}
public void setId(int id) {
this.id = id;
}
public int getUserid() {
return userid;
}
public void setUserid(int userid) {
this.userid = userid;
}
public int getTarget_id() {
return target_id;
}
public void setTarget_id(int target_id) {
this.target_id = target_id;
}
public int getType() {
return type;
}
public void setType(int type) {
this.type = type;
}
public String getAdd_time() {
return add_time;
}
public void setAdd_time(String add_time) {
this.add_time = add_time;
}
}
```

### 主函数 

![主函数](主函数.png)

在 AppMain 类中添加如下内容： 

```java
package com.atguigu.appclient;
import java.io.UnsupportedEncodingException;
import java.util.Random;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.atguigu.bean.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
/**
* 日志行为数据模拟
*/
public class AppMain {
private final static Logger logger =
LoggerFactory.getLogger(AppMain.class);
private static Random rand = new Random();
// 设备 id
private static int s_mid = 0;
// 用户 id
private static int s_uid = 0;
// 商品 id
private static int s_goodsid = 0;
public static void main(String[] args) {
// 参数一：控制发送每条的延时时间，默认是 0
Long delay = args.length > 0 ? Long.parseLong(args[0]) : 0L;
// 参数二：循环遍历次数
int loop_len = args.length > 1 ? Integer.parseInt(args[1]) : 1000;
// 生成数据
generateLog(delay, loop_len);
}
private static void generateLog(Long delay, int loop_len) {
for (int i = 0; i < loop_len; i++) {
JSONObject json = new JSONObject();
json.put("ap", "app");
json.put("cm", generateComFields());
JSONArray eventsArray = new JSONArray();
// 启动日志
eventsArray.add(generateStart());
// 事件日志
// 商品曝光
if (rand.nextBoolean()) {
eventsArray.add(generateDisplay());
}
// 商品详情页
if (rand.nextBoolean()) {
eventsArray.add(generateNewsDetail());
}
// 商品列表页
if (rand.nextBoolean()) {
eventsArray.add(generateNewList());
}
// 购物车
if (rand.nextBoolean()) {
eventsArray.add(generateCart());
}
// 广告
if (rand.nextBoolean()) {
eventsArray.add(generateAd());
}
// 消息通知
if (rand.nextBoolean()) {
eventsArray.add(generateNotification());
}
//故障日志
if (rand.nextBoolean()) {
eventsArray.add(generateError());
}
// 用户评论
if (rand.nextBoolean()) {
eventsArray.add(generateComment());
}
// 用户收藏
if (rand.nextBoolean()) {
eventsArray.add(generateFavorites());
}
// 用户点赞
if (rand.nextBoolean()) {
eventsArray.add(generatePraise());
}
json.put("et", eventsArray);
//时间
long millis2 = System.currentTimeMillis();
//控制台打印
logger.info(millis2 + "|" + json.toJSONString());
// 延迟
try {
Thread.sleep(delay);
} catch (InterruptedException e) {
e.printStackTrace();
}
}
}
/**
* 公共字段设置
*/
private static JSONObject generateComFields() {
AppBase appBase = new AppBase();
//设备 id
appBase.setMid(s_mid + "");
s_mid++;
// 用户 id
appBase.setUid(s_uid + "");
s_uid++;
// 程序版本号 5,6 等
appBase.setVc("" + rand.nextInt(20));
//程序版本名 v1.1.1
appBase.setVn("1." + rand.nextInt(4) + "." + rand.nextInt(10));
// 安卓系统版本
appBase.setOs("8." + rand.nextInt(3) + "." + rand.nextInt(10));
// 语言 es,en,pt
int flag = rand.nextInt(3);
switch (flag) {
case (0):
appBase.setL("es");
break;
case (1):
appBase.setL("en");
break;
case (2):
appBase.setL("pt");
break;
}
// 渠道号 从哪个渠道来的
appBase.setSr(getRandomChar(1));
// 区域
flag = rand.nextInt(2);
switch (flag) {
case 0:
appBase.setAr("BR");
case 1:
appBase.setAr("MX");
}
// 手机品牌 ba ,手机型号 md，就取 2 位数字了
flag = rand.nextInt(3);
switch (flag) {
case 0:
appBase.setBa("Sumsung");
appBase.setMd("sumsung-" + rand.nextInt(20));
break;
case 1:
appBase.setBa("Huawei");
appBase.setMd("Huawei-" + rand.nextInt(20));
break;
case 2:
appBase.setBa("HTC");
appBase.setMd("HTC-" + rand.nextInt(20));
break;
}
// 嵌入 sdk 的版本
appBase.setSv("V2." + rand.nextInt(10) + "." + rand.nextInt(10));
// gmail
appBase.setG(getRandomCharAndNumr(8) + "@gmail.com");
// 屏幕宽高 hw
flag = rand.nextInt(4);
switch (flag) {
case 0:
appBase.setHw("640*960");
break;
case 1:
appBase.setHw("640*1136");
break;
case 2:
appBase.setHw("750*1134");
break;
case 3:
appBase.setHw("1080*1920");
break;
}
// 客户端产生日志时间
long millis = System.currentTimeMillis();
appBase.setT("" + (millis - rand.nextInt(99999999)));
// 手机网络模式 3G,4G,WIFI
flag = rand.nextInt(3);
switch (flag) {
case 0:
appBase.setNw("3G");
break;
case 1:
appBase.setNw("4G");
break;
case 2:
appBase.setNw("WIFI");
break;
}
// 拉丁美洲 西经 34°46′至西经 117°09；北纬 32°42′至南纬 53°54′
// 经度
appBase.setLn((-34 - rand.nextInt(83) - rand.nextInt(60) / 10.0) +
"");
// 纬度
appBase.setLa((32 - rand.nextInt(85) - rand.nextInt(60) / 10.0) +
"");
return (JSONObject) JSON.toJSON(appBase);
}
/**
* 商品展示事件
*/
private static JSONObject generateDisplay() {
AppDisplay appDisplay = new AppDisplay();
boolean boolFlag = rand.nextInt(10) < 7;
// 动作：曝光商品=1，点击商品=2，
if (boolFlag) {
appDisplay.setAction("1");
} else {
appDisplay.setAction("2");
}
// 商品 id
String goodsId = s_goodsid + "";
s_goodsid++;
appDisplay.setGoodsid(goodsId);
// 顺序 设置成 6 条吧
int flag = rand.nextInt(6);
appDisplay.setPlace("" + flag);
// 曝光类型
flag = 1 + rand.nextInt(2);
appDisplay.setExtend1("" + flag);
// 分类
flag = 1 + rand.nextInt(100);
appDisplay.setCategory("" + flag);
JSONObject jsonObject = (JSONObject) JSON.toJSON(appDisplay);
return packEventJson("display", jsonObject);
}
/**
* 商品详情页
*/
private static JSONObject generateNewsDetail() {
AppNewsDetail appNewsDetail = new AppNewsDetail();
// 页面入口来源
int flag = 1 + rand.nextInt(3);
appNewsDetail.setEntry(flag + "");
// 动作
appNewsDetail.setAction("" + (rand.nextInt(4) + 1));
// 商品 id
appNewsDetail.setGoodsid(s_goodsid + "");
// 商品来源类型
flag = 1 + rand.nextInt(3);
appNewsDetail.setShowtype(flag + "");
// 商品样式
flag = rand.nextInt(6);
appNewsDetail.setShowtype("" + flag);
// 页面停留时长
flag = rand.nextInt(10) * rand.nextInt(7);
appNewsDetail.setNews_staytime(flag + "");
// 加载时长
flag = rand.nextInt(10) * rand.nextInt(7);
appNewsDetail.setLoading_time(flag + "");
// 加载失败码
flag = rand.nextInt(10);
switch (flag) {
case 1:
appNewsDetail.setType1("102");
break;
case 2:
appNewsDetail.setType1("201");
break;
case 3:
appNewsDetail.setType1("325");
break;
case 4:
appNewsDetail.setType1("433");
break;
case 5:
appNewsDetail.setType1("542");
break;
default:
appNewsDetail.setType1("");
break;
}
// 分类
flag = 1 + rand.nextInt(100);
appNewsDetail.setCategory("" + flag);
JSONObject eventJson = (JSONObject) JSON.toJSON(appNewsDetail);
return packEventJson("newsdetail", eventJson);
}
/**
* 商品列表
*/
private static JSONObject generateNewList() {
AppLoading appLoading = new AppLoading();
// 动作
int flag = rand.nextInt(3) + 1;
appLoading.setAction(flag + "");
// 加载时长
flag = rand.nextInt(10) * rand.nextInt(7);
appLoading.setLoading_time(flag + "");
// 失败码
flag = rand.nextInt(10);
switch (flag) {
case 1:
appLoading.setType1("102");
break;
case 2:
appLoading.setType1("201");
break;
case 3:
appLoading.setType1("325");
break;
case 4:
appLoading.setType1("433");
break;
case 5:
appLoading.setType1("542");
break;
default:
appLoading.setType1("");
break;
}
// 页面 加载类型
flag = 1 + rand.nextInt(2);
appLoading.setLoading_way("" + flag);
// 扩展字段 1
appLoading.setExtend1("");
// 扩展字段 2
appLoading.setExtend2("");
// 用户加载类型
flag = 1 + rand.nextInt(3);
appLoading.setType("" + flag);
JSONObject jsonObject = (JSONObject) JSON.toJSON(appLoading);
return packEventJson("loading", jsonObject);
}
/**
* 购物车
*/
static public JSONObject generateCart() {
AppCart appItemCart = new AppCart();
appItemCart.setItemid(s_mid);
appItemCart.setBeforeNum(1 + rand.nextInt(3));
appItemCart.setAction(1 + rand.nextInt(2));
if (appItemCart.getAction() == 2) {
int changNum = (-1) + rand.nextInt(3);
changNum = (changNum == 0 ? 1 : changNum);
appItemCart.setChangeNum(changNum);
appItemCart.setAfterNum(appItemCart.getBeforeNum() +
appItemCart.getChangeNum());
}
JSONObject jsonObject = (JSONObject) JSON.toJSON(appItemCart);
return packEventJson("favorites", jsonObject);
}
/**
* 广告相关字段
*/
private static JSONObject generateAd() {
AppAd appAd = new AppAd();
// 入口
int flag = rand.nextInt(3) + 1;
appAd.setEntry(flag + "");
// 动作
flag = rand.nextInt(5) + 1;
appAd.setAction(flag + "");
// 状态
flag = rand.nextInt(10) > 6 ? 2 : 1;
appAd.setContent(flag + "");
// 失败码
flag = rand.nextInt(10);
switch (flag) {
case 1:
appAd.setDetail("102");
break;
case 2:
appAd.setDetail("201");
break;
case 3:
appAd.setDetail("325");
break;
case 4:
appAd.setDetail("433");
break;
case 5:
appAd.setDetail("542");
break;
default:
appAd.setDetail("");
break;
}
// 广告来源
flag = rand.nextInt(4) + 1;
appAd.setSource(flag + "");
// 用户行为
flag = rand.nextInt(2) + 1;
appAd.setBehavior(flag + "");
// 商品类型
flag = rand.nextInt(10);
appAd.setNewstype("" + flag);
// 展示样式
flag = rand.nextInt(6);
appAd.setShow_style("" + flag);
JSONObject jsonObject = (JSONObject) JSON.toJSON(appAd);
return packEventJson("ad", jsonObject);
}
/**
* 启动日志
*/
static JSONObject generateStart() {
AppStart appStart = new AppStart();
// 入口
int flag = rand.nextInt(5) + 1;
appStart.setEntry(flag + "");
// 开屏广告类型
flag = rand.nextInt(2) + 1;
appStart.setOpen_ad_type(flag + "");
// 状态
flag = rand.nextInt(10) > 8 ? 2 : 1;
appStart.setAction(flag + "");
// 加载时长
appStart.setLoading_time(rand.nextInt(20) + "");
// 失败码
flag = rand.nextInt(10);
switch (flag) {
case 1:
appStart.setDetail("102");
break;
case 2:
appStart.setDetail("201");
break;
case 3:
appStart.setDetail("325");
break;
case 4:
appStart.setDetail("433");
break;
case 5:
appStart.setDetail("542");
break;
default:
appStart.setDetail("");
break;
}
JSONObject jsonObject = (JSONObject) JSON.toJSON(appStart);
return packEventJson("start", jsonObject);
}
/**
* 消息通知
*/
private static JSONObject generateNotification() {
AppNotification appNotification = new AppNotification();
int flag = rand.nextInt(4) + 1;
// 动作
appNotification.setAction(flag + "");
// 通知 id
flag = rand.nextInt(4) + 1;
appNotification.setType(flag + "");
// 客户端弹时间
appNotification.setAp_time((System.currentTimeMillis() -
rand.nextInt(99999999)) + "");
// 备用字段
appNotification.setContent("");
JSONObject jsonObject = (JSONObject) JSON.toJSON(appNotification);
return packEventJson("notification", jsonObject);
}
/**
* 错误日志数据
*/
private static JSONObject generateError() {
AppError appErrorLog = new AppError();
String[] errorBriefs = {"at
cn.lift.dfdf.web.AbstractBaseController.validInbound(AbstractBaseControll
er.java:72)", "at
cn.lift.appIn.control.CommandUtil.getInfo(CommandUtil.java:67)"};
//错误摘要
String[] errorDetails = {"java.lang.NullPointerException\\n " +
"at
cn.lift.appIn.web.AbstractBaseController.validInbound(AbstractBaseControl
ler.java:72)\\n " + "at
cn.lift.dfdf.web.AbstractBaseController.validInbound", "at
cn.lift.dfdfdf.control.CommandUtil.getInfo(CommandUtil.java:67)\\n " +
"at
sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorI
mpl.java:43)\\n" + " at
java.lang.reflect.Method.invoke(Method.java:606)\\n"}; //错误详情
//错误摘要
appErrorLog.setErrorBrief(errorBriefs[rand.nextInt(errorBriefs.length)]);
//错误详情
appErrorLog.setErrorDetail(errorDetails[rand.nextInt(errorDetails.length)
]);
JSONObject jsonObject = (JSONObject) JSON.toJSON(appErrorLog);
return packEventJson("error", jsonObject);
}
/**
* 为各个事件类型的公共字段（时间、事件类型、 Json 数据）拼接
*/
private static JSONObject packEventJson(String eventName, JSONObject
jsonObject) {
JSONObject eventJson = new JSONObject();
eventJson.put("ett", (System.currentTimeMillis() -
rand.nextInt(99999999)) + "");
eventJson.put("en", eventName);
eventJson.put("kv", jsonObject);
return eventJson;
}
/**
* 获取随机字母组合
*
* @param length 字符串长度
*/
private static String getRandomChar(Integer length) {
StringBuilder str = new StringBuilder();
Random random = new Random();
for (int i = 0; i < length; i++) {
// 字符串
str.append((char) (65 + random.nextInt(26)));// 取得大写字母
}
return str.toString();
}
/**
* 获取随机字母数字组合
*
* @param length 字符串长度
*/
private static String getRandomCharAndNumr(Integer length) {
StringBuilder str = new StringBuilder();
Random random = new Random();
for (int i = 0; i < length; i++) {
boolean b = random.nextBoolean();
if (b) { // 字符串
// int choice = random.nextBoolean() ? 65 : 97; 取得 65 大写字
母还是 97 小写字母
str.append((char) (65 + random.nextInt(26)));// 取得大写字母
} else { // 数字
str.append(String.valueOf(random.nextInt(10)));
}
}
return str.toString();
}
/**
* 收藏
*/
private static JSONObject generateFavorites() {
AppFavorites favorites = new AppFavorites();
favorites.setCourse_id(rand.nextInt(10));
favorites.setUserid(rand.nextInt(10));
favorites.setAdd_time((System.currentTimeMillis() -
rand.nextInt(99999999)) + "");
JSONObject jsonObject = (JSONObject) JSON.toJSON(favorites);
return packEventJson("favorites", jsonObject);
}
/**
* 点赞
*/
private static JSONObject generatePraise() {
AppPraise praise = new AppPraise();
praise.setId(rand.nextInt(10));
praise.setUserid(rand.nextInt(10));
praise.setTarget_id(rand.nextInt(10));
praise.setType(rand.nextInt(4) + 1);
praise.setAdd_time((System.currentTimeMillis() -
rand.nextInt(99999999)) + "");
JSONObject jsonObject = (JSONObject) JSON.toJSON(praise);
return packEventJson("praise", jsonObject);
}
/**
* 评论
*/
private static JSONObject generateComment() {
AppComment comment = new AppComment();
comment.setComment_id(rand.nextInt(10));
comment.setUserid(rand.nextInt(10));
comment.setP_comment_id(rand.nextInt(5));
comment.setContent(getCONTENT());
comment.setAddtime((System.currentTimeMillis() -
rand.nextInt(99999999)) + "");
comment.setOther_id(rand.nextInt(10));
comment.setPraise_count(rand.nextInt(1000));
comment.setReply_count(rand.nextInt(200));
JSONObject jsonObject = (JSONObject) JSON.toJSON(comment);
return packEventJson("comment", jsonObject);
}
/**
* 生成单个汉字
*/
private static char getRandomChar() {
String str = "";
int hightPos; //
int lowPos;
Random random = new Random();
//随机生成汉子的两个字节
hightPos = (176 + Math.abs(random.nextInt(39)));
lowPos = (161 + Math.abs(random.nextInt(93)));
byte[] b = new byte[2];
b[0] = (Integer.valueOf(hightPos)).byteValue();
b[1] = (Integer.valueOf(lowPos)).byteValue();
try {
str = new String(b, "GBK");
} catch (UnsupportedEncodingException e) {
e.printStackTrace();
System.out.println("错误");
}
return str.charAt(0);
}
/**
* 拼接成多个汉字
*/
private static String getCONTENT() {
StringBuilder str = new StringBuilder();
for (int i = 0; i < rand.nextInt(100); i++) {
str.append(getRandomChar());
}
return str.toString();
}
}
```

### 配置日志打印（Logback） 

Logback 主要用于在磁盘和控制台打印日志。 Logback 具体使用如下：
1）在 resources 文件夹下创建 logback.xml 文件。
2）在 logback.xml 文件中填写如下配配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
<!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径 -->
<property name="LOG_HOME" value="/opt/module/logs/" />
<!-- 控制台输出 -->
<appender name="STDOUT"
class="ch.qos.logback.core.ConsoleAppender">
<encoder
class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
<!--格式化输出： %d 表示日期， %thread 表示线程名， %-5level：级别从左显示 5 个
字符宽度%msg：日志消息， %n 是换行符 -->
<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}
- %msg%n</pattern>
</encoder>
</appender>
<!-- 按照每天生成日志文件。存储事件日志 -->
<appender name="FILE"
class="ch.qos.logback.core.rolling.RollingFileAppender">
<!-- <File>${LOG_HOME}/app.log</File>设置日志不超过${log.max.size}时的
保存路径，注意，如果是 web 项目会保存到 Tomcat 的 bin 目录 下 -->
<rollingPolicy
class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
<!--日志文件输出的文件名 -->
<FileNamePattern>${LOG_HOME}/app-%d{yyyy-MMdd}.log</FileNamePattern>
<!--日志文件保留天数 -->
<MaxHistory>30</MaxHistory>
</rollingPolicy>
    <encoder
class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
<pattern>%msg%n</pattern>
</encoder>
<!--日志文件最大的大小 -->
<triggeringPolicy
class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
<MaxFileSize>10MB</MaxFileSize>
</triggeringPolicy>
</appender>
<!--异步打印日志-->
<appender name ="ASYNC_FILE" class=
"ch.qos.logback.classic.AsyncAppender">
<!-- 不丢失日志.默认的,如果队列的 80%已满,则会丢弃 TRACT、 DEBUG、 INFO 级别的
日志 -->
<discardingThreshold >0</discardingThreshold>
<!-- 更改默认的队列的深度,该值会影响性能.默认值为 256 -->
<queueSize>512</queueSize>
<!-- 添加附加的 appender,最多只能添加一个 -->
<appender-ref ref = "FILE"/>
</appender>
<!-- 日志输出级别 -->
<root level="INFO">
<appender-ref ref="STDOUT" />
<appender-ref ref="ASYNC_FILE" />
<appender-ref ref="error" />
</root>
</configuration>
```

### 打包 

1）采用 Maven 对程序打包 

![打包](打包.png)

## 数据采集模块 

### 日志生成 

1）集群规划 

![集群规划](集群规划.png)

2） 打包好的 log-collector-1.0-SNAPSHOT-jar-with-dependencies.jar 程序到hadoop102 的/opt/module/目录下，并执行如下命令 

java -jar log-collector-1.0-SNAPSHOT-jar-with-dependencies.jar > /dev/null 2>&1 & 

3）然后进入到/opt/module/logs 目录，观察日志是否写入成功 

## Flume 安装及使用 

1） Flume 官网地址： http://flume.apache.org/
2）文档查看地址： http://flume.apache.org/FlumeUserGuide.html
3）下载地址： http://archive.apache.org/dist/flume/ 

### Flume 简介 

Flume 是 Cloudera 提供的一个高可用的，高可靠的， 分布式的海量日志采集、聚合和传
输的系统。 Flume 基于流式架构，灵活简单。 

![flume](flume.png)

1） Source
主要负责采集工作，采用 TailDir 组件用于监控文件或文件夹的变化。
2） Channel
扮演数据管道的角色，对数据进行缓冲。采用非持久化的 Memory 类型。
3） Sink
把 Channel 中的数据输出到外部环境中，支持多种数据接口（ HDFS、 Kafka 等），此次
案例中我们的最终目标是数据到阿里云的数据总线中（ DataHub），调试阶段可以先输出到
控制台中。 

### Flume 安装 

#### 集群规划 

![flume集群规划](flume集群规划.png)

1）将 apache-flume-1.7.0-bin.tar.gz 上传到 hadoop102 的/opt/software 目录下
2）解压 apache-flume-1.7.0-bin.tar.gz 到/opt/module/目录下
[atguigu@hadoop102 software]$ tar -zxf apache-flume-1.7.0-
bin.tar.gz -C /opt/module/
3） 修改 apache-flume-1.7.0-bin 的名称为 flume
[atguigu@hadoop102 module]$ mv apache-flume-1.7.0-bin flume
4）将 flume/conf 下的 flume-env.sh.template 文件修改为 flume-env.sh，并配置 flume-env.sh 文
件 

[atguigu@hadoop102 conf]$ mv flume-env.sh.template flume-env.sh
[atguigu@hadoop102 conf]$ vi flume-env.sh
export JAVA_HOME=/opt/module/jdk1.8.0_144 

#### Flume 配置 

1）在/opt/module/flume/conf 中添加文件 file-flume-log.conf，该文件是一个 Flume 作业的核
心文件，咱们上述的 Source、 Channel、 Sink 都是通过这个配置文件来实现的。 

[atguigu@hadoop102 conf]$ vim file-flume-log.conf
添加如下内容 

# 定义组件名称
a1.sources = r1
a1.sinks = k1
a1.channels = c1
###source 部分
a1.sources.r1.type = TAILDIR
#记录偏移量实现断点续传
a1.sources.r1.positionFile =/opt/module/flume/test
/taildir_position.json
a1.sources.r1.channels = c1
a1.sources.r1.filegroups=f1
a1.sources.r1.filegroups.f1=/opt/module/logs/app.+
a1.sources.r1.fileHeader = true
###channel 部分
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 1000
###sink 部分 先输出到控制台中
a1.sinks.k1.type = logger

#把 source 和 sink 绑定到 channel 中

a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1 

#### 启动 Flume 进程 

[atguigu@hadoop102 flume]$ /opt/module/flume/bin/flume-ng agent -n a1 -c /opt/module/flume/conf/ -f
/opt/module/flume/conf/file-flume-log.conf -Dflume.root.logger=info,console 

说明

| 参数选项                         | 含义                                   |
| -------------------------------- | -------------------------------------- |
| flume-ng agent                   | 启动 Flume 程序                        |
| -n                               | 任务名称，必须和配置文件中的前缀一致。 |
| -c                               | Flume 基本配置文件位置                 |
| -f                               | Flume 任务配置文件位置                 |
| -Dflume.root.logger=info,console | 打印控制台                             |

验证，启动 Flume 的同时，运行日志生成程序，观察 Flume 控制台是否滚动打印日志。 

## DataHub 安装及使用 

### DataHub 简介 

Flume 部分已经可以输出后，咱们开始搭建真正需要输出的目的地---DataHub，即阿里云数据总线服务。
通俗来说这个 DataHub 类似于传统大数据解决方案中 Kafka 的角色，提供了一个数据 队列功能。 

对于离线计算， DataHub 除了供了一个缓冲的队列作用。 

同时由于 DataHub 提供了各种与其他阿里云上下游产品的对接功能，所以 DataHub 又扮演了一个数据的分发枢纽工作。 

![datahub](datahub.png)

1） DataHub 输入组件包括
Flume：主流的开源日志采集框架
DTS： 类似 Canal， 日志实时监控采集框架
Logstash： 也是日志采集框架，通常和 Elasticsearch、 Kibana 集合使用
Fluentd： Fluentd 是一个实时开源的数据收集器
OGG： 实时监控 Oracle 中数据变化
Java sdk：支持 JavaAPI 方式访问
2） DataHub 输出组件包括
RDS：类似与传统 MySQL 数据库
AnalyticDB： 面向分析型的分布式数据库
MaxCompute：离线分析框架
Elasticsearch：数据分析，倒排索引
StreamCompute：实时分析框架
TableSotre：类似于 Redis， KV 形式存储数据 

OSS：类似于 HDFS， 存储图片、视频 

### 创建 DataHub 与 Topic 

阿里云 DataHub 控制台入口： https://datahub.console.aliyun.com/datahub
1） 进入到 DataHub 控制台 

![DataHub 控制台](datahub_create.png)

2） 点击创建 Project 

![创建 Project ](create_project.png)

3）点击查看，准备创建主题 

![查看topic](topic.png)

4）点击创建 Topic 

![创建Topic](topic_create.png)

5）配置 Topic 详情 

![topic_details](topic_details.png)

说明：

| 选择参数   | 含义                                               |
| ---------- | -------------------------------------------------- |
| Topic 类型 | Tuple 为结构化数据， Blob 是二进制数据。           |
| Schema     | Tuple 类型的字段名                                 |
| Shard 数量 | 决定了队列吞吐量，每个 Shard 支持 1MB/s 的写入能力 |
| 生命周期   | 数据在队列中的最长存活时间                         |

## Flume 推送数据到 DataHub 

### Flume-DataHub 插件安装 

Flume 默认是不支持 DataHub 的，所以要给 Flume 安装 DataHub 的 Sink 插件 

插件名称: aliyun-flume-datahub-sink-2.0.2.tar.gz

1）首先在 Flume 安装目录建立插件文件夹 

mkdir plugins.d 

2 ） 利 用 SecureCRT 工 具 把 aliyun-flume-datahub-sink-2.0.2.tar.gz 拷 贝 到 该
/opt/module/flume/plugins.d 目录下，并原地解压缩 

ls

tar -zxvf aliyun-flume-datahubsink-2.0.2.tar.gz 

#### Flume 配置文件修改 

vim /opt/module/flume/conf/fileflume-datahub.conf 

#定义组件名称 

a1.sources = r1
a1.sinks = k1
a1.channels = c1
### source 部分
a1.sources.r1.type = TAILDIR
a1.sources.r1.positionFile =
/opt/module/flume/test/taildir_position.json
a1.sources.r1.channels = c1
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /opt/module/logs/app.+
a1.sources.r1.fileHeader = true
### channel 部分
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 1000

#sink 部分

a1.sinks.k1.type = com.aliyun.datahub.flume.sink.DatahubSink
a1.sinks.k1.datahub.accessID = LTAI4FiU71dZAL17SdLBa6Nt
a1.sinks.k1.datahub.accessKey = 63YzSmqMOSjDR5A2ZXEzFLM2tREY6m
a1.sinks.k1.datahub.endPoint = http://xxxxxinc.com
a1.sinks.k1.datahub.project = gmall_datahub
a1.sinks.k1.datahub.topic = base_log
a1.sinks.k1.batchSize = 100
a1.sinks.k1.serializer = DELIMITED
a1.sinks.k1.serializer.delimiter = "\\u007C"
a1.sinks.k1.serializer.fieldnames = event_time,log_string
a1.sinks.k1.serializer.charset = UTF-8
a1.sinks.k1.shard.number = 1
a1.sinks.k1.shard.maxTimeOut = 60

#把 source 和 sink 绑定到 channel 中

a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1 

说明：

| 选择参数              | 含义                                                         |
| --------------------- | ------------------------------------------------------------ |
| type                  | 必须是： com.aliyun.datahub.flume.sink.DatahubSink           |
| datahub.accessID      | accessID 下面小节会介绍                                      |
| datahub.accessKey     | accessKey                                                    |
| datahub.endPoint      | 不同的地区有不同的接入点，参考 https://help.aliyun.com/document_detail/47442.html |
| project               | datahub 中创建的 project 名                                  |
| topic                 | datahub 中 project 下的 topic                                |
| batchSize             | 每批次都少条记录                                             |
| serializer            | 序列化方式， DELIMITED 表示按分隔符切分                      |
| serializer.delimiter  | 具体分隔符是什么，这里“\\u007C” 表示 \| Unicode 转中文工具地址： http://www.msxindl.com/tools/unicode16.asp |
| serializer.fieldnames | 切分后对应的字段名                                           |

| serializer.charset | 字符集              |
| ------------------ | ------------------- |
| shard.number       | 分片数              |
| shard.maxTimeOut   | 超时时间， 单位是秒 |

### 获取 AccessID 和 AccessKey
DataHub 服务并不是靠 IP 来定位的，而是靠阿里云账号，每个阿里云账号只能有一个
DataHub，每个阿里云账号也会有唯一的 AccessID 和 AccessKey。所以通过 AccessId 和
AccessKey 就可以直接锁定某个阿里云账号的 DataHub。
1）悬浮鼠标到阿里云账号头像上->点击 accesskeys 

2）点击继续使用 AccessKey 

3）新用户需要点击创建 AccessKey 

4）获取到 AccessKeyID 和 AccessKeySecret 值 

### 查看接收数据

1）启动 Flume 进程 

/opt/module/flume/bin/flume-ng agent -n a1 -c /opt/module/flume/conf/ -f /opt/module/flume/conf/file-flume-datahub.conf -Dflume.root.logger=info,console 

2）启动日志生成程序 

java -jar log-collector-1.0-SNAPSHOT-jar-with-dependencies.jar > /dev/null 2>&1 & 

3） 观察 DataHub 中数据量 

![查看datahub](check_datahub.png)

## DataWorks 和 MaxCompute 

### 简介 

MaxCompute（大数据计算服务）是阿里巴巴自主研发的海量数据处理平台，主要提供数据上传和下载通道，提供 SQL 及 MapReduce 等多种计算分析服务，同时还提供完善的安全解决方案。
DataWorks（数据工场，原大数据开发套件） 是基于 MaxCompute 计算引擎的一站式大 数据工场，它能帮助您快速完成数据集成、开发、治理、服务、质量、安全等全套数据研发工作。 

![DataWorks 和 MaxCompute架构](dm_archi.png)

盘古： 相当于 Hadoop 中的 HDFS
伏羲： 相当于 Hadoop 中的 YARN
MaxCompute Engine： 相当于 MR、 Tez 等计算引擎
MaxCompute 和 DataWorks 一起向用户提供完善的 ETL 和数仓管理能力，以及 SQL、
MR、 Graph 等多种经典的分布式计算模型，能够更快速地解决用户海量数据计算问题，有
效降低企业成本，保障数据安全。 

## 用户行为数仓搭建 

### 数仓分层概念 

#### 数仓分层 

![数据仓库分层结构](dw.png)

1） ODS 层
原始数据层，存放原始数据，直接加载原始日志、数据，数据保持原貌不做处理。
2） DWD 层
对 ODS 层数据进行清洗（去除空值，脏数据，超过极限范围的数据）
3） DWS 层
以 DWD 为基础，进行轻度汇总。
4） ADS 层
为各种统计报表提供数据 

### 数仓分层优点 

1）把复杂问题简单化
将一个复杂的任务分解成多个步骤来完成，每一层只处理单一的步骤，比较简单、并且
方便定位问题。
2）减少重复开发
规范数据分层，通过的中间层数据，能够减少极大的重复计算，增加一次计算结果的复
用性。
3）隔离原始数据
不论是数据的异常还是数据的敏感性，使真实数据与统计数据解耦开。 

### 数仓命名规范 

ODS层命名为ods前缀
DWD层命名为dwd前缀 

DWS层命名为dws前缀 

ADS层命名为ads前缀 

维度表命名为dim前缀 

每日全量导入命名为df（day full） 后缀 

每日增量导入命名为di（day increase） 后缀 

### 数仓分层配置 

#### 建立业务流程 

1）点击数据开发->业务流程->新建业务流程->输入业务名称 

![新建业务流程](业务名称.png)

2）再业务流程下面就可以看到业务 1 

![查看业务](1584634741197.png)

### 配置表主题 

1）进入配置中心
点击最下方的齿轮，进入配置中心 

![配置表主题](1584634957752.png)

2）配置主题管理，这里主要是划分表的主题 

主题一般是表的对应的业务主题，比如：基础表、用户、商品、广告等对应的业务线。 

![配置主题管理](1584635032717.png)

#### 配置表层级 

![配置表层级](1584635099901.png)

在同一页面中， DataWorks 还提供了一个物理分类的划分维度，用户可以根据情况自行决定划分方式。
本案例中，对数据表按照数据来源划分为，日志、数据库和综合。 

### 原始数据层（ODS 层） 

用户行为数仓分层 

![用户行为数仓分层 ](1584635454970.png)

#### 建表语句 

1）回到数据开发的界面，按下图开始创建数据表 

![创建表](1584635640105.png)

2）输入表名->提交 

![提交](1584635719607.png)

开始建立的表要和原始数据最基本的结构一致 。

#### 配置基本属性 

配置新创建的表的主题。 

![配置表主题](1584635847424.png)

#### 配置物理模型 

注意选择层级和物理分类，由于建立的原始日志表，所以此处选择 ODS 层。分类是日志 

![配置物理模型](1584635939718.png)

从 DataHub 过来的数据必须选择分区表。如果手工文件导入的表可以选择非分区。
为了减少再购买存储服务器， 所以选择内部表。真实企业开发时，大部分情况都是创建外部表，需要在 OSS 服务中申请存储空间。

#### 配置字段 

发送过来的日志包含两部分： 服务器时间和日志详情。 这里面设计了两个字段对应接收。 

![配置建表字段](1584636608997.png)

注意：这里的主键，是指一个标志而已，本身不提供索引和唯一性约束。 

#### 配置分区 

表示数据按如下字段进行分区 

![配置分区](1584636730662.png)

由于目前 DataHub 至 MaxCompute 的接口，只支持按年月日+小时+分钟方式分区，所以这三个字段是必须有的，且字段类型必须是 String。 

#### 查看建表语句 

点击 DDL 模式，可以查看自动生成的建表语句。 和普通的创建表语句一样。 

![查看建表语句](1584637075694.png)

#### 提交到生产环境 

点击提交到生产环境，表就创建完成了 。

![提交到生产环境](1584636936852.png)

### 表的基本操作 

#### 查看表结构 

在最左侧菜单中选择【表管理】，可以在右侧查看表的结构信息。但是不可以修改。 

![查看表结构](1584637608000.png)

### 在业务流程中导入表 

在业务 1 中建 ods、 dwd、 dws、 ads， 4 个文件夹 

![导入表](1584637717559.png)

2）向 ods 文件夹中导入表 

![导入表到ods](1584637827902.png)

#### 临时查询 

1） 点击临时查询->新建->ODPS SQL 

![新建ODPS SQL](1584637916303.png)

2）创建临时查询节点 

![创建临时查询节点](1584637977047.png)

3）可以执行 SQL 命令 

![执行临时查询的SQL](1584638259522.png)

## DataHub 推送数据到 MaxCompute 

如下图，之前 Flume 中的数据利用 DataHub Sink 把数据写入到了 DataHub 中， DataHub中提供了很多的其他第三方的 DataConnector 可以连接各种例如： MaxCompute， ElasticSearch，ADB， RDS 等数据库。 

所以下面就要建立 DataConnector 把数据推送到 MaxCompute 中。 

![系统架构流程设计-DataConnector](1584638414320.png)

### 创建 DataConnector 

1）在 DataHub 中找到 Topic，在某个 Topic 下，点击右上角的 DataConnector 

![找到DataConnector ](1584638618996.png)

2） 点击同步到 MaxCompute 离线表 

![同步到 MaxCompute 离线表 ](1584638706167.png)

3） 创建 DataConnector 

![创建DataConnector ](1584638788411.png)

（1） MaxCompute Project：名称要和 MaxCompute 创建的工作空间名称一致
（ 2） MaxCompute Table： 数据导入到 MaxCompute 中的表名
（ 3） AccessKey ID： LTAI4FiU71dZAL17SdLBa6Nt
（ 4） AccessKey Secret： 63YzSmqMOSjDR5A2ZXEzFLM2tREY6m
注意： AccessKey ID 和 AccessKey Secret 要和自己的阿里云账号一一对应
（ 5）分区选项： system、 event_time
通常采用 system 时间分区。
如果是 event_time 方式分区，就要在 topic 中包含一个 event_time 的字段。不过这
个字段与以往的 timestamp 不同的是，必须精确到微秒级。而且这个字段一旦用于分区，
则不会再写入到实体表中。
（ 6）分区范围： 15 分钟起 

#### 发送数据 

建立好表后 DataConnector 就可以尝试发送数据了。
注意： 如果已经启动了 Flume，就不需要再次启动了。
1）启动 Flume 程序 

/opt/module/flume/bin/flume-ng agent-n a1 -c /opt/module/flume/conf/ -f /opt/module/flume/conf/file-flume-datahub.conf -Dflume.root.logger=info,console 

2）在服务器 hadoop102 上执行命令 

java -jar log-collector-1.0-SNAPSHOT-jar-with-dependencies.jar > /dev/null 2>&1 & 

#### 接收数据 

1） 观察 DataHub 中接收到数据（速度很快） 

![查看Datahub](1584639053388.png)

2）查看 MaxCompute 中接收到数据（1-5 分钟的延迟） 

![查看MaxCompute](1584639105289.png)

### 明细数据层（DWD 层） 

DWD 层主要是对 ODS 层数据进行清洗（去除空值，脏数据，超过极限范围的数据）。DWD 层处理后的表，能够成为非常明确可用的基础明细数据。
本次项目中需要将用户行为过来基础日志，根据表类型，一张一张的解析出来 11 张不同类型的表数据，方便后续的处理。 

#### 日志格式分析 

1）日志格式：服务器时间 | json 

2）其中 json 包括：
cm：公共字段的 key；
ap： app 的名称；
et：具体事件 

![日志格式](1584639317048.png)

### 自定义 UDTF（解析具体事件字段） 

开发 UDTF 有两个方法：

* 方法 1：在本地 IDEA 中创建工程，开发代码，打包，把 JAR 上传到 DataStudio 成为资源 JAR 包。然后基于资源 JAR 包，声明函数。

* 方法 2：直接在 FunctionStudio 中开发，然后在线打包发布程序，声明函数。

  相比而言，从发布流程上来说利用 FunctionStudio 更快捷方便。但是从 IDEA 开发角度
  来说，网页版本的 FunctionStudio，肯定不如客户端的功能强大、反应速度流畅。不过也可
  以两者配合起来使用。
  本次案例主要介绍通过 FunctionStudio 来编写 UDTF 函数。
  1） 按下图所示，打开 FunctionStudio 的界面 

![FunctionStudio 的界面](1584639625158.png)

2）创建工程 

![创建工程 ](1584639765968.png)

3）选择命名工程名，选择 udfjava 

![命名工程名](1584639815129.png)

4）添加代码文件 

可以看到工程创建好后，默认有很多参考的模板。 

![查看参考模板](1584639907800.png)

5）在 udtf 目录下创建一个 FlatEventUDTF 类 

![创建FlatEventUDTF类](1584640061023.png)

6）编写代码 

首先分析日志结构 

![日志结构](1584640172393.png)

1）在 pom.xml 中要加入 fastJson 依赖 

```xml
<dependency>
<groupId>com.alibaba</groupId>
<artifactId>fastjson</artifactId>
<version>1.2.28.odps</version>
</dependency> 
```

（ 2）编写自定义 UDTF 代码 

```java
import com.aliyun.odps.udf.ExecutionContext;
import com.aliyun.odps.udf.UDFException;
import com.aliyun.odps.udf.UDTF;
import com.aliyun.odps.udf.annotation.Resolve;
import com.alibaba.fastjson.*;
// TODO define input and output types, e.g.
"string,string->string,bigint".
@Resolve({"string->bigint,string,string"})
public class FlatEventUDTF extends UDTF {
@Override
public void setup(ExecutionContext ctx) throws UDFException
{ }
@Override
public void process(Object[] args) throws UDFException {
String event =(String) args[0];
JSONArray jsonArray = JSON.parseArray(event);
for (int i = 0; i < jsonArray.size(); i++) {
JSONObject jsonObject = jsonArray.getJSONObject(i);
String ett =(String) jsonObject.getString("ett");
String eventName =(String) jsonObject.getString("en");
String eventJson =(String) jsonObject.getString("kv");
forward(Long.parseLong(ett),eventName,eventJson);
}
}
@Override
public void close() throws UDFException {
}
}
```

其中， @Resolve({"string->bigint,string,string"} 表示该函数，传入参数是 string，返回
参数是三个，分别为长整型，和两个字符串，对应的返回值就是事件的时间、事件名称和事
件内容（json 格式）。 

7）打包部署 

（1）提交到 Data Studio 开发环境 

![提交到Data Studio 开发环境](1584640681181.png)

（2）提交函数详情 

![提交函数](1584640746505.png)

（ 3）提交函数过程，控制台打印 

![查看提交日志](1584640794096.png)

提交成功后 ，回到 DataWorks 中的资源菜单中可以看到增加了 1 个 JAR 包，函数菜单
中可以看到定义的函数。 

![查看提交后的jar包](1584640850276.png)

8）测试函数
在 DataStudio 中临时查询，执行如下语句 

```sql
select
FlatEventUDTF(GET_JSON_OBJECT(log_string,'$.et')) as
(event_time, event_name, event_json)
from ods_base_log
where ds='20191008'; 
```

能看到如下结果表示运行正确 

![查看udf函数运行结果](1584640962328.png)

### DWD 层建表（启动日志表） 

在流程中建立表，在 DWD 文件夹中增加一个 dwd_start_log 的表，可以直接用 ddl 模式建立，再进行微调。 

1） 在业务 1 的表上，右键新建表 

![创建DWD层表](1584641165935.png)

2）点击 DDL 模式 

![DDL模式](1584641222222.png)

3） 在 DDL 模式中添加建表语句 

![DDL 模式中添加建表语句 ](1584641312899.png)

详细建表语句如下 

```sql
CREATE TABLE `dwd_start_log` (
`mid` string,
`user_id` string,
`version_code` string,
`version_name` string,
`lang` string,
`source` string,
`os` string,
`area` string,
`model` string,
`brand` string,
`sdk_version` string,
`email` string,
`height_width` string,
`network` string,
`lng` string,
`lat` string,
`entry` string,
`open_ad_type` string,
`action` string,
`loading_time` string,
`detail` string,
`event_time` string COMMENT '事件时间'
)
PARTITIONED BY (ds string, hh string, mm string);
```

4）补充一些相关字段。 

![添加补充字段](1584641570680.png)

5）补充分区格式 

![补充分区格式](1584641611008.png)

6）提交到生产环境 

![提交到生产环境 ](1584642445160.png)

#### 手动将 ODS 层数据导入 DWD 层 

1）在临时查询页面， 把 ods 层 ods_base_log 里面的数据导入到 dwd_start_log 

```sql
INSERT OVERWRITE TABLE dwd_start_log PARTITION (ds,hh,mm)
select
GET_JSON_OBJECT(log_string,'$.cm.mid') mid,
GET_JSON_OBJECT(log_string,'$.cm.uid') user_id,
GET_JSON_OBJECT(log_string,'$.cm.vc') version_code,
GET_JSON_OBJECT(log_string,'$.cm.vn') version_name,
GET_JSON_OBJECT(log_string,'$.cm.l') lang,
GET_JSON_OBJECT(log_string,'$.cm.sr') source,
GET_JSON_OBJECT(log_string,'$.cm.os') os,
GET_JSON_OBJECT(log_string,'$.cm.ar') area,
GET_JSON_OBJECT(log_string,'$.cm.md') model,
GET_JSON_OBJECT(log_string,'$.cm.ba') brand,
GET_JSON_OBJECT(log_string,'$.cm.sv') sdk_version,
GET_JSON_OBJECT(log_string,'$.cm.hw') height_width,
GET_JSON_OBJECT(log_string,'$.cm.g') email,
GET_JSON_OBJECT(log_string,'$.cm.hw') sv,
GET_JSON_OBJECT(log_string,'$.cm.ln') ln,
GET_JSON_OBJECT(log_string,'$.cm.la') la,
GET_JSON_OBJECT( event_view.event_json,'$.entry') entry,
GET_JSON_OBJECT( event_view.event_json,'$.loading_time')
loading_time,
GET_JSON_OBJECT( event_view.event_json,'$.action') action,
GET_JSON_OBJECT( event_view.event_json,'$.open_ad_type')
open_ad_type,
GET_JSON_OBJECT( event_view.event_json,'$.detail') detail ,
event_view.event_time,
ds,
hh,
mm
from ods_base_log
LATERAL VIEW FlatEventUDTF(GET_JSON_OBJECT(log_string,'$.et' ))
event_view as event_time,event_name,event_json
where ds='20191008' and event_view.event_name = 'start';
```

注意: get_json_object函数的作用：用来解析json字符串的一个字段

2）在临时查询中查看导入结果 

```sql
SELECT * from dwd_start_log WHERE ds='20191008'; 
```

### 数据导入脚本 

1）在流程中加入一个ODPS数据开发SQL脚本 

![加入数据开发脚本](1584643098845.png)

![增加节点](1584643149148.png)

```sql
INSERT OVERWRITE TABLE dwd_start_log PARTITION (ds,hh,mm)
select
GET_JSON_OBJECT(log_string,'$.cm.mid') mid,
GET_JSON_OBJECT(log_string,'$.cm.uid') user_id,
GET_JSON_OBJECT(log_string,'$.cm.vc') version_code,
GET_JSON_OBJECT(log_string,'$.cm.vn') version_name,
GET_JSON_OBJECT(log_string,'$.cm.l') lang,
GET_JSON_OBJECT(log_string,'$.cm.sr') source,
GET_JSON_OBJECT(log_string,'$.cm.os') os,
GET_JSON_OBJECT(log_string,'$.cm.ar') area,
GET_JSON_OBJECT(log_string,'$.cm.md') model,
GET_JSON_OBJECT(log_string,'$.cm.ba') brand,
GET_JSON_OBJECT(log_string,'$.cm.sv') sdk_version,
GET_JSON_OBJECT(log_string,'$.cm.hw') height_width,
GET_JSON_OBJECT(log_string,'$.cm.g') email,
GET_JSON_OBJECT(log_string,'$.cm.hw') sv,
GET_JSON_OBJECT(log_string,'$.cm.ln') ln,
GET_JSON_OBJECT(log_string,'$.cm.la') la,
GET_JSON_OBJECT( event_view.event_json,'$.entry') entry,
GET_JSON_OBJECT( event_view.event_json,'$.loading_time')
loading_time,
GET_JSON_OBJECT( event_view.event_json,'$.action') action,
GET_JSON_OBJECT( event_view.event_json,'$.open_ad_type')
open_ad_type,
GET_JSON_OBJECT( event_view.event_json,'$.detail') detail,
event_view.event_time,
ds,
hh,
mm
from ods_base_log
LATERAL VIEW FlatEventUdtf (GET_JSON_OBJECT(log_string,'$.et' ))
event_view as event_time,event_name,event_json
where ds='${bizdate}' and event_view.event_name = 'start' ;
```

注意： 在上面的 SQL 中我们使用了一个${bizedate}作为外部传入的日期参数 。

2）配置参数
在执行或者调度该脚本的时候传入相应的参数。 

![配置参数](1584643401626.png)

（ 1）在右侧有一个调度配置，打开可以对参数进行设置。 

![添加日期参数](1584643504020.png)

注意：
这里参数设置用的是花括号， bizdate=${yyyymmdd}， 表示取前一日的日期；
如果采用方括号，如， bizdate= $[yyyymmdd]， 表示取当前日期。 

（ 2）配置脚本执行时间 

![配置脚本执行时间](1584643614288.png)

### 服务数据层（DWS 层） 

需求：日活统计 

#### 建表语句 

1） 创建表 dws_uv_detail_d 

![表dws_uv_detail_d](1584643770273.png)

2）点击 DDL 模式 

![DDL 模式](1584643818760.png)

3） 在 DDL 模式中添加建表语句 

```sql
CREATE TABLE `dws_uv_detail_d` (
`mid` string COMMENT '设备唯一标识',
`user_id` string COMMENT '用户标识',
`version_code` string COMMENT '程序版本号',
`version_name` string COMMENT '程序版本名',
`lang` string COMMENT '系统语言',
`source` string COMMENT '渠道号',
`os` string COMMENT '系统版本',
`area` string COMMENT '区域',
`model` string COMMENT '手机型号',
`brand` string COMMENT '手机品牌',
`sdk_version` string COMMENT 'sdkversion',
`email` string COMMENT 'email',
`height_width` string COMMENT '屏幕宽高',
`network` string COMMENT '网络模式',
`lng` string COMMENT '经度',
`lat` string COMMENT '纬度',
`event_time` bigint
)
COMMENT '活跃用户按天明细'
PARTITIONED BY (ds string,hh string,mm string);
```

4）补全建表信息描述 

![补全建表信息描述](1584643921603.png)

![添加分区信息](1584643948842.png)

5）提交到生产环境 

![提交到生产环境](1584643984926.png)

#### 手动将 DWD 层数据导入 DWS 层 

1）在临时查询页面， 把 DWD 层 dwd_start_log 里面的数据导入到 dws_uv_detail_d 

```sql
insert overwrite table dws_uv_detail_d partition(ds,hh,mm)
select
mid,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
email,
height_width,
network,
lng,
lat,
event_time,
ds,
hh,
mm
from
(
select
*,
ROW_NUMBER() OVER(PARTITION BY mid ORDER BY event_time
asc) rn
from dwd_start_log
where ds='20191008'
) st where rn = 1;
```

2）查看导入结果 

SELECT * from dws_uv_detail_d WHERE ds='20191008'; 

### 数据导入脚本 

DWS 层一般围绕某个主题进行聚合、拼接处理。针对统计日活的需求， DWS 主要的工作就进行以日为单位的去重操作。

1）在流程中加入一个数据开发脚本 

![dws层数据开发脚本](1584644314585.png)

![dws层数据开发脚本](1584644385290.png)

```sql
insert overwrite table dws_uv_detail_d partition(ds,hh,mm)
select
mid,
user_id,
version_code,
version_name,
lang,
source,
os,
area,
model,
brand,
sdk_version,
email,
height_width,
network,
lng,
lat,
event_time,
ds,
hh,
mm
from
(
select
*,
ROW_NUMBER() OVER(PARTITION BY mid ORDER BY event_time
asc) rn
from dwd_start_log
where ds = '${bizdate}'
) st where rn = 1;
```

2）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

## 应用数据层（ADS 层） 

统计各个渠道的 uv 个数 

### 建表语句 

1） 创建表 ads_uv_source_d 

![ads层建表](1584644541801.png)

2）点击 DDL 模式 

![DDL模式](1584644598884.png)

3） 在 DDL 模式中添加建表语句

```sql
CREATE TABLE `ads_uv_source_d` (
`source` string COMMENT '渠道',
`ct` bigint COMMENT '个数'
)
COMMENT '日活渠道统计'
PARTITIONED BY (ds string);
```

4）补全建表信息描述 

![ads层补全建表信息描述](1584644721022.png)

![补充分区信息](1584644784126.png)

5）提交到生产环境 

![提交到生产环境](1584644817997.png)

### ads层数据导入脚本 

1）在流程中加入一个数据开发脚本 

![在ADS层增加odps sql脚本](1584644889517.png)

![在ADS层增加odps sql脚本](1584644949976.png)

```sql
insert OVERWRITE table ads_uv_source_d PARTITION
(ds='${bizdate}')
SELECT
source,
COUNT(*) ct
from dws_uv_detail_d
where ds='${bizdate}'
group by source;
```

2）配置参数
点击调度配置-> bizdate=${yyyymmdd} 

### 日活需求： 全流程业务调度 

1）点击业务 1， 右侧就会出现之前写的脚本。默认三个脚本之间没有关系，可以根据业务需
求，手动连线。 

![点击业务1，手动连线](1584645057782.png)

2）点击执行 

![点击执行](1584645130260.png)

3）查询运行日志 

![查询运行日志](1584645175186.png)

4）临时查询， 检查结果 

```sql
SELECT * from ads_uv_source_d WHERE ds='20191008';
```



to be continued...
