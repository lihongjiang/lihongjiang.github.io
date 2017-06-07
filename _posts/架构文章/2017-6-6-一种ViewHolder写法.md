---
layout: post
title: 一种ViewHolder写法
category: 架构文章
tags: Behavior
---


## 背景
MVC 在android代码层面，V指的一般XML，对应的xml操作都在Activity，C一般也是Activity承担了，造成了VC同框。

MVP P解决了V/C分离,Activity不承担C功能，但还是没达到View和View逻辑分离，还是会出现在Activity，还是会造成代码混乱。
MVVM VM解决了view和数据model耦合。

在android开发中，页面基本都是一个滚动的页面，实现一般都是ScrollView或者ListView或者RecycleView。

在这些容器中装载的是一个个的复合控件。控件的官方操作写法是ViewHolder来绑定数据和事件处理。对于复杂的页面,holder还是比较庞大的。

结合MVC,MVP,MVVM，自己提出了一种写法来达到解耦View和View事件逻辑。

## 三层Holder模型

    1 ViewModel    XML属性映射层
    2 ViewAdapter  XML数据适配层
    3 ViewPresenter XML事件处理和其它模块解耦层

数据流方向：
  ViewPresenter---ViewAdapter---ViewModel
事件流方向
  ViewModel---ViewPpresenter--ViewAdapter---viewModel

## ViewModel层设计思路
    
 采用注解配置XML，反射初始化ROOTVIEW。
 采用接口来声明要操作的控件
 采用黑盒子ID方式操作控件
 
 基类提供类似标签属性方式操作常见控件方法，比如text(id,string),enable(id,boolean),click(id.oncllick);
 
 提供set/get控件方法
 
 提供获得Context上下文方法，统一维护。

 扩展组合标签属性方式。

## ViewAdapter层设计思路

  采用数据适配器方式，建立数据模型和UI界面的数据绑定桥梁

## ViewPresenter

  初始化View单击事件
  处理事件逻辑
  解耦网络·存储模块等层设计思路


