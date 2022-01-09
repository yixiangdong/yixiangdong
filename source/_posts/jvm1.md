---
layout: jvm1
title: jvm源码之jdk,jre,jvm以及内存溢出场景模拟
date: 2020-04-02 19:12:08
tags: 源码分析
comments: true
---

### jdk,jre,jvm的包含关系

![1585826092223](1585826092223.png)

详情可以参照:

<https://docs.oracle.com/javase/7/docs/>

![jvm架构](1585826305357.png)

### jvm初体验: 内存溢出场景模拟

<!--more-->

![内存溢出场景模拟](1585829912226.png)

```java
import java.util.ArrayList;
import java.util.List;
public class Main{
    public static void main(String[] args){
        List<Demo>  demolist = new ArrayList<>();
        while(true){
            demolist.add(new Demo());
        }
    }
}
```

![内存溢出以前](1585830534423.png)

![内存直线上升](1585830000745.png)

![堆内存溢出异常](1585830050838.png)

内存溢出检测：堆内存快照

RUN AS->RUN CONFIGURATION

![设置VM参数](1585831275215.png)

-XX:+HeapDumpOnOutOfMemoryError -Xms20m -Xmx20m

-Xms20m -Xmx20m 的意思是只分配给这个程序20mb的内存，这样程序就很快内存溢出了，只是为了方便模拟内存溢出这个场景，平时不能这么设置。

![内存溢出模拟](1585831573263.png)

错误提示:

java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid12920.hprof ...
Heap dump file created [34441735 bytes in 0.148 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:265)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
	at java.util.ArrayList.add(ArrayList.java:462)
	at com.test.Main.main(Main.java:12)

在项目根目录找到报错的日志文件

![java_pid12920.hprof](1585831746962.png)

java_pid12920.hprof该文件需要下载一个内存分析插件来查看

比如： Memory Analyzer 1.7.0 release

在这里下载：<http://www.eclipse.org/mat/downloads.php>

![内存分析插件](1585831976642.png)

插件下载链接：<https://www.eclipse.org/downloads/download.php?file=/mat/1.10.0/rcp/MemoryAnalyzer-1.10.0.20200225-win32.win32.x86_64.zip>

![解压](1585833829570.png)

![打开内存溢出的日志文件java_pid12920.hprof看到的界面](1585833861684.png)

![使用open dominator tree分析内存使用百分比](1585834325180.png)

![查看内存使用最高的类名](1585834428860.png)

列明解释：

Shallow Heap: 对象本身占用的内存大小

Retained Heap: 当前直接或者间接引用对象的大小总和（这里是GC要回收的内存大小）

![查看对象引用的次数最多的Value](1585834996009.png)

然后根据引用的对象定位到代码，找到解决方案

