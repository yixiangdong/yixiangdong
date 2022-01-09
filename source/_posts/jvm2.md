---
layout: jvm2
title: jvm源码之Jvm监控工具使用
date: 2020-04-02 21:54:07
tags: 源码分析
comments: true
---

### Jvm监控工具使用

定位到jdk的安装目录C:\Program Files\Java\jdk1.8.0_211\bin\jconsole.exe

启动监控工具:

run->CMD->jconsole  

![CMD](1585836001333.png)

![Java监视和管理控制台](1585836025499.png)

该功能类似于jps

<!--more-->

![jps](1585836170677.png)

![jconsole界面](1585836340384.png)

### 监控新生代Eden区域新建对象

![内存Eden区域](1585837649144.png)

```java
package com.test.jconsole;

import java.util.ArrayList;
import java.util.List;

public class JConsoleTest {
	public byte []b1= new byte[128*1024];
    public static void main(String[] args) throws InterruptedException {
    	Thread.sleep(5000);
    	System.out.println("start ...");
    	fill(1000);
    	
    			
    }
    private static void fill(int n) throws InterruptedException {
    	List<JConsoleTest> jlist = new ArrayList<>();
    	for(int i = 0; i<n; i++) {
    		Thread.sleep(3000);
    		jlist.add(new JConsoleTest());
    	}
    }
}


```

### 运行程序并进入Jconsole界面看内存变化趋势

![运行](1585837840386.png)

![点击进程](1585837882801.png)

查看内存变化

堆内不进行垃圾回收，所以是直线上升的。

![查看堆内存变化](1585840014117.png)

对象在Eden区域是否是存活，根据对象年龄判断，如果长期存活放在老年代

![查看Eden内存变化](1585838102067.png)

