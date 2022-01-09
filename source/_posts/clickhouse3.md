---
layout: clickhouse3
title: Flink+ClickHouse电商实时数据分析系统之日志数据结构
date: 2020-04-05 21:01:20
tags: ClickHouse
comments: true
---

### 浏览日志

#### PC端

macAdress mac地址
deviceId 设备号
userId用户id
cookie如果没有cookie，用户禁用了cookie，或者由于其他原因为获取到，该字段值为空
remoteIP客户端的ip
remoteName客户端名称
country国家

<!--more-->

province省
city市
deviceType设备类型，计算机、移动设备、其他等等
os操作系统
brower 浏览器信息
resolution分辨率
sourceInfo搜索引擎
sourceType跳转源类型
srcDomain跳转来源地址

#### 小程序端

userId//用户id
deviceId//设备id
weixinAccount//微信号
weixinName //微信昵称
weixinSex 微信性别信息
weixinArea 微信地区
openTime打开小程序时间
leaveTime退出时间

#### App端

userId
deviceId
openTimeStamp//打开时间
leaveTimeStamp;//退出时间
appPlatform;//平台，安卓，IOS
deviceStyle;//型号
brand;//品牌
screenSize;//分辨率
osType;//操作系统

### 页面浏览信息

页面id
访问时间
跳出时间
停留时间
商品id
广告id
商品类别id
活动id
秒杀活动id
团购活动id



### 新建模块jiangziCommon

作用：用于存放公共日志结构

#### 新建设备公共类DeviceComomInfo

```java
public class DeviceComomInfo {
    private String userId;//用户id,未登陆，用户id为-1
    private String deviceId;//设备id
    private String openTime;//打开终端时间
    private String leaveTime;//退出终端时间
    private String remoteIP;//客户端的ip
    private String country;//国家
    private String province;//省
    private String city;//城市
    private String channelinfo;//渠道信息
    private boolean isNew = false;//是否为新增用户
    private boolean hourActive = false;//是否为小时活跃
    private boolean dayActive = false;//是否为天活跃
    private boolean monthActive = false;//是否为月活跃
    private boolean weekActive = false;//是否为周活跃
    private boolean fiveMinuteActive = false;//是否为5分钟活跃


    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getDeviceId() {
        return deviceId;
    }

    public void setDeviceId(String deviceId) {
        this.deviceId = deviceId;
    }

    public String getOpenTime() {
        return openTime;
    }

    public void setOpenTime(String openTime) {
        this.openTime = openTime;
    }

    public String getLeaveTime() {
        return leaveTime;
    }

    public void setLeaveTime(String leaveTime) {
        this.leaveTime = leaveTime;
    }

    public String getRemoteIP() {
        return remoteIP;
    }

    public void setRemoteIP(String remoteIP) {
        this.remoteIP = remoteIP;
    }

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }

    public String getProvince() {
        return province;
    }

    public void setProvince(String province) {
        this.province = province;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public boolean isNew() {
        return isNew;
    }

    public void setNew(boolean aNew) {
        isNew = aNew;
    }

    public boolean isHourActive() {
        return hourActive;
    }

    public void setHourActive(boolean hourActive) {
        this.hourActive = hourActive;
    }

    public boolean isDayActive() {
        return dayActive;
    }

    public void setDayActive(boolean dayActive) {
        this.dayActive = dayActive;
    }

    public boolean isMonthActive() {
        return monthActive;
    }

    public void setMonthActive(boolean monthActive) {
        this.monthActive = monthActive;
    }

    public boolean isWeekActive() {
        return weekActive;
    }

    public void setWeekActive(boolean weekActive) {
        this.weekActive = weekActive;
    }

    public boolean isFiveMinuteActive() {
        return fiveMinuteActive;
    }

    public void setFiveMinuteActive(boolean fiveMinuteActive) {
        this.fiveMinuteActive = fiveMinuteActive;
    }

    public String getChannelinfo() {
        return channelinfo;
    }

    public void setChannelinfo(String channelinfo) {
        this.channelinfo = channelinfo;
    }
}
```

#### 新建app 公共类，继承与设备类

```java
package com.jiangzi.input;

public class AppInfo extends DeviceComomInfo{
    private String appPlatform;//平台，安卓，IOS
    private String deviceStyle;//型号
    private String brand;//品牌
    private String screenSize;//分辨率
    private String osType;//操作系统

    public String getAppPlatform() {
        return appPlatform;
    }

    public void setAppPlatform(String appPlatform) {
        this.appPlatform = appPlatform;
    }

    public String getDeviceStyle() {
        return deviceStyle;
    }

    public void setDeviceStyle(String deviceStyle) {
        this.deviceStyle = deviceStyle;
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public String getScreenSize() {
        return screenSize;
    }

    public void setScreenSize(String screenSize) {
        this.screenSize = screenSize;
    }

    public String getOsType() {
        return osType;
    }

    public void setOsType(String osType) {
        this.osType = osType;
    }
}
```

 #### 新建PC端公共类，继承设备类

```java
package com.jiangzi.input;

public class PcInfo extends DeviceComomInfo{
    private String macAdress;//mac地址
    private String cookie;//如果没有cookie，用户禁用了cookie，或者由于其他原因为获取到，该字段值为空
    private String remoteName;//客户端名称
    private String deviceType;//设备类型，计算机、移动设备、其他等等
    private String os;//操作系统
    private String brower;//浏览器信息
    private String resolution;//分辨率
    private String sourceInfo;//搜索引擎
    private String sourceType;//跳转源类型
    private String srcDomain;//跳转来源地址

    public String getMacAdress() {
        return macAdress;
    }

    public void setMacAdress(String macAdress) {
        this.macAdress = macAdress;
    }

    public String getCookie() {
        return cookie;
    }

    public void setCookie(String cookie) {
        this.cookie = cookie;
    }

    public String getRemoteName() {
        return remoteName;
    }

    public void setRemoteName(String remoteName) {
        this.remoteName = remoteName;
    }

    public String getDeviceType() {
        return deviceType;
    }

    public void setDeviceType(String deviceType) {
        this.deviceType = deviceType;
    }

    public String getOs() {
        return os;
    }

    public void setOs(String os) {
        this.os = os;
    }

    public String getBrower() {
        return brower;
    }

    public void setBrower(String brower) {
        this.brower = brower;
    }

    public String getResolution() {
        return resolution;
    }

    public void setResolution(String resolution) {
        this.resolution = resolution;
    }

    public String getSourceInfo() {
        return sourceInfo;
    }

    public void setSourceInfo(String sourceInfo) {
        this.sourceInfo = sourceInfo;
    }

    public String getSourceType() {
        return sourceType;
    }

    public void setSourceType(String sourceType) {
        this.sourceType = sourceType;
    }

    public String getSrcDomain() {
        return srcDomain;
    }

    public void setSrcDomain(String srcDomain) {
        this.srcDomain = srcDomain;
    }
}
```

#### 新建小程序公共类，继承设备类

```java
package com.jiangzi.input;

public class XiaochengxuInfo extends DeviceComomInfo{
    private String weixinAccount;//微信号
    private String weixinName;//微信昵称
    private String weixinSex;//微信性别信息
    private String weixinArea;//微信地区

    public String getWeixinAccount() {
        return weixinAccount;
    }

    public void setWeixinAccount(String weixinAccount) {
        this.weixinAccount = weixinAccount;
    }

    public String getWeixinName() {
        return weixinName;
    }

    public void setWeixinName(String weixinName) {
        this.weixinName = weixinName;
    }

    public String getWeixinSex() {
        return weixinSex;
    }

    public void setWeixinSex(String weixinSex) {
        this.weixinSex = weixinSex;
    }

    public String getWeixinArea() {
        return weixinArea;
    }

    public void setWeixinArea(String weixinArea) {
        this.weixinArea = weixinArea;
    }
}
```

#### 新建浏览页面日志

```java
package com.jiangzi.input;

public class ScanPageLog {
    private String pageId;//页面id
    private String visitTime;//访问时间
    private String jumpTime;//跳出时间
    private String remainTime;//停留时间
    private String productId;//商品id
    private String adId;//广告id
    private String productTypeId;//商品类别id
    private String huodongId;// 活动id
    private String miaoshaId;//秒杀活动id
    private String tuangouId;//团购活动id
    private String deviceType;//0、app端 1、pc端 2、小程序端
    private DeviceComomInfo deviceComomInfo;
    public String getPageId() {
        return pageId;
    }

    public void setPageId(String pageId) {
        this.pageId = pageId;
    }

    public String getVisitTime() {
        return visitTime;
    }

    public void setVisitTime(String visitTime) {
        this.visitTime = visitTime;
    }

    public String getJumpTime() {
        return jumpTime;
    }

    public void setJumpTime(String jumpTime) {
        this.jumpTime = jumpTime;
    }

    public String getRemainTime() {
        return remainTime;
    }

    public void setRemainTime(String remainTime) {
        this.remainTime = remainTime;
    }

    public String getProductId() {
        return productId;
    }

    public void setProductId(String productId) {
        this.productId = productId;
    }

    public String getAdId() {
        return adId;
    }

    public void setAdId(String adId) {
        this.adId = adId;
    }

    public String getProductTypeId() {
        return productTypeId;
    }

    public void setProductTypeId(String productTypeId) {
        this.productTypeId = productTypeId;
    }

    public String getHuodongId() {
        return huodongId;
    }

    public void setHuodongId(String huodongId) {
        this.huodongId = huodongId;
    }

    public String getMiaoshaId() {
        return miaoshaId;
    }

    public void setMiaoshaId(String miaoshaId) {
        this.miaoshaId = miaoshaId;
    }

    public String getTuangouId() {
        return tuangouId;
    }

    public void setTuangouId(String tuangouId) {
        this.tuangouId = tuangouId;
    }

    public String getDeviceType() {
        return deviceType;
    }

    public void setDeviceType(String deviceType) {
        this.deviceType = deviceType;
    }

    public DeviceComomInfo getDeviceComomInfo() {
        return deviceComomInfo;
    }

    public void setDeviceComomInfo(DeviceComomInfo deviceComomInfo) {
        this.deviceComomInfo = deviceComomInfo;
    }
}
```

到此日志结构新建完毕！