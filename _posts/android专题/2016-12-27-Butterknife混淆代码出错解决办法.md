---
layout: post
title: Android Studio使用Butterknife混淆代码出错解决办法
category: 混淆专题
tags: proguard
keywords: Butterknife
description: 
---

### 文章引用出处：[www.cnblogs.com/bestcolin/p/5551958.html](www.cnblogs.com/bestcolin/p/5551958.html) 

###  在proguard-rules.pro混淆规则文件中添加：

	-keep class butterknife.** { *; }
	-dontwarn butterknife.internal.**
	-keep class **$$ViewBinder { *; }
	
	-keepclasseswithmembernames class * {
	    @butterknife.* <fields>;
	}
	
	-keepclasseswithmembernames class * {
	    @butterknife.* <methods>;
	}