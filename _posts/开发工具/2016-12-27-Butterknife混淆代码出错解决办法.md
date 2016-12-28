---
layout: post
title: Butterknife混淆代码
category: 开发工具
tags: proguard
keywords: Butterknife
description: Android Studio使用Butterknife混淆代码出错解决办法
---

### 文章引用出处：[https://www.cnblogs.com/bestcolin/p/5551958.html](https://www.cnblogs.com/bestcolin/p/5551958.html) 

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