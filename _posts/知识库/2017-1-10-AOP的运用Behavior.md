---
layout: post
title: AOP的运用-Behavior
category: 知识库
tags: Behavior
keywords: Behavior
description: Behavior
---


## 0 参考

[CoordinatorLayout 自定义Behavior并不难，由简到难手把手带你飞](http://www.cnblogs.com/hexihexi/p/6143716.html)

[自定义CoordinatorLayout的Behavior实现知乎和简书快速返回效果](http://blog.csdn.net/tiankong1206/article/details/48394393)

[android-[译]掌握CoordinatorLayout](http://www.jianshu.com/p/f418bf95db2d)

[关于 CoordinatorLayout 与 Behavior 的一点分析](http://android.jobbole.com/83640/)

## 1 理论知识

<li>CoordinatorLayout是一个非常强大的控件，它接管了child组件之间的交互
<li>AOP切面编程
<li>其实Behavior就是一个应用于View的观察者模式，一个View跟随者另一个View的变化而变化，或者说一个View监听另一个View。在Behavior中，被观察View 也就是事件源被称为denpendcy，而观察View，则被称为child。
<li> Behavior  可以监听所有子控件和事件处理。  Behavior监听事件来做一个拦截AOP分发
<li>Behavior只有是CoordinatorLayout的直接子View才有意义。可以为任何View添加一个Behavior。Behavior是一系列回调。让你有机会以非侵入的为View添加动态的依赖布局，和处理父布局(CoordinatorLayout)滑动手势的机会

## 2 控件知识


CollapsingToolbarLayout extends FrameLayout

CoordinatorLayout extends ViewGroup implements NestedScrollingParent  协调子控件的事件，子控件需要自己实现行为来控制其他子控件

AppBarLayout extends LinearLayout  注解有缺省行为，不需要在布局设置
FloatingActionButton extends ImageView 注解有缺省行为，不需要在布局设置


TextInputLayout extends LinearLayout
TabLayout extends HorizontalScrollView
NavigationView extends ScrimInsetsFrameLayout

Snackbar
