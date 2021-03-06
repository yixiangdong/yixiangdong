---
layout: ceshi16
title: 实时用户画像系统(十一)java实现逻辑回归算法理论学习
date: 2020-03-27 21:55:53
tags: 机器学习
comments: true
---

### 微分

看待微分的意义，可以有不同的角度，最常用的两种是：
函数图像中，某点的切线的斜率

<!-- more -->

函数的变化率几个微分的例子：

![微分公式](1585317803821.png)

### 多微分

当一个函数有多个变量的时候，就有了多变量的微分，即分别对每个变量进行求微分。

![多微分例子](1585318120547.png)

### 梯度

梯度实际上就是多变量微分的一般化

![梯度例子](1585318167979.png)

![梯度例子](1585318195373.png)

在单变量的函数中，梯度其实就是函数的微分，代表着函数在某个给定点的切线的斜率
在多变量函数中，梯度是一个向量，向量有方向，梯度的方向就指出了函数在给定点的上升最快的方向

### 梯度下降算法的数学解释

J是关于Θ的一个函数，我们当前所处的位置为Θ0点，要从这个点走到J的最小值点，也就是山底。首先我们先确定前进的方向，也就是梯度的方向，然后走一段距离的步长，也就是α，走完这个段步长，就到达了Θ1这个点！

![梯度下降算法](1585318284122.png)

α在梯度下降算法中被称作为学习率或者步长

### 梯度下降算法的实现

首先，我们需要定义一个代价函数，在此我们选用均方误差代价函数

![梯度下降算法](1585318331948.png)

m是数据集中点的个数
½是一个常量，这样是为了在求梯度的时候，二次方乘下来就和这里的½抵消了，自然就没有多余的常数系数，方便后续的计算，同时对结果不会有影响
y 是数据集中每个点的真实y坐标的值
h 是我们的预测函数，根据每一个输入x，根据Θ 计算得到预测的y值，即

![预测函数](1585318355141.png)

我们可以根据代价函数看到，代价函数中的变量有两个，所以是一个多变量的梯度下降问题，求解出代价函数的梯度，也就是分别对两个变量进行微分

![误差计算方法](1585318397966.png)