---
layout: post
title: Android异常之Service启动Activity
category: 源码专题
tags: Android异常之Service启动Activity
---

# [Android异常之Service启动Activity](http://bbs.51cto.com/thread-1133875-1.html)

	Android异常之Service启动Activity
	
	在Activity中其中startActivity这个大家应该是非常熟悉的；那么从service里面调用startActivity话，会怎么样呢？
	会出现下面的异常：
	android.util.AndroidRuntimeException: Calling startActivity() from outside of an Activity  context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?
	
	也就是在service里面启动Activity的话，必须添加FLAG_ACTIVITY_NEW_TASK flag。
	那么下面的话，我们将从下面几个方面分析这个问题。
	1．    这个异常怎么产生的？
	2．    解决这个异常后会出现问题？
	3．    为什么Activity.startActivity()不会出现这个问题？
	4．    Android 为什么要这么设计？
	下面，一一分析
	
	一.    Context的继承关系图
	首先来看一张图， 这张图表示了Context里面的基本继承关系。
	
	1.    最上面的是Context.java，它其实是一个抽象类，它有两个重要的子类ContextImpl和ContextWrapper
	2.    ContextImpl，是Context功能实现的主要类，
	3.    ContextWrapper，顾名思义，它只是一个包装而已。主要功能实现都是通过调用ContextImpl去实现的。
	4.    ContextThemeWrapper，包括一些主题的包装，由于Service没有主题，所以直接继承ContextWrapper；但是Activity就需要继承ContextThemeWrapper
	
	
	
	二.    异常如何产生
	1．    找到报错的代码
	文件：
	frameworks\base\core\java\android\app\ContextImpl.java
	代码：
	01
		public void startActivity(Intent intent, Bundle options) {
	02
		        warnIfCallingFromSystemProcess();
	03
		        if ((intent.getFlags()&Intent.FLAG_ACTIVITY_NEW_TASK) == 0) {
	04
		            throw new AndroidRuntimeException(
	05
		                    "Calling startActivity() from outside of an Activity "
	06
		                    + " context requires the FLAG_ACTIVITY_NEW_TASK flag."
	07
		                    + " Is this really what you want?");
	08
		        }
	09
		        mMainThread.getInstrumentation().execStartActivity(
	10
		            getOuterContext(), mMainThread.getApplicationThread(), null,
	11
		            (Activity)null, intent, -1, options);
	12
		}
	在下面的if条件判断，如果不包含FLAG_ACTIVITY_NEW_TASK就会报这个错误
	1
		if ((intent.getFlags()&Intent.FLAG_ACTIVITY_NEW_TASK) == 0) {
	2
		    ...
	3
		}   
	那么service.startActivity(Intent intent)怎么会调用这里来的呢？
	要回答这个问题，我们分析下service.startActivity()做了什么，其实，service.startActivity调用的是ContextWrapper.startActivity()，因为service继承自ContextWrapper
	2.  代码文件
	frameworks\base\core\java\android\content\ContextWrapper.java
	代码：
	1
		public void startActivity(Intent intent, Bundle options) {
	2
		    mBase.startActivity(intent, options);
	3
		}
	ContextWrapper.startActivity的话，是直接调用的
	mBase.startActivity(intent, options);
	那么这个mBase是什么呢？又是什么时候赋值的呢？其实mBase是在ContextWrapper的attachBaseContext的时候初始化的。如下：
	1
		protected void attachBaseContext(Context base) {
	2
		        if (mBase != null) {
	3
		            throw new IllegalStateException("Base context already set");
	4
		        }
	5
		        mBase = base;
	6
		    }
	那又是谁调用attachBaseContext的呢？
	是在service创建的时候，在ActivityThread里面调用，如下：
	
	3. 代码文件
	frameworks\base\core\java\android\app\ActivityThread.java
	代码：
	01
		private void handleCreateService(CreateServiceData data) {
	02
		        LoadedApk packageInfo = getPackageInfoNoCheck(
	03
		                data.info.applicationInfo, data.compatInfo);
	04
		        Service service = null;
	05
		        try {
	06
		            java.lang.ClassLoader cl = packageInfo.getClassLoader();
	07
		            service = (Service) cl.loadClass(data.info.name).newInstance();
	08
		        } catch (Exception e) {
	09
		            ....
	10
		        }
	11
		        try {
	12
		            if (localLOGV) Slog.v(TAG, "Creating service " + data.info.name);
	13
		            ContextImpl context = ContextImpl.createAppContext(this, packageInfo);
	14
		            context.setOuterContext(service);
	15
		            Application app = packageInfo.makeApplication(false, mInstrumentation);
	16
		            service.attach(context, this, data.info.name, data.token, app,
	17
		                    ActivityManagerNative.getDefault());
	18
		            service.onCreate();
	19
		            mServices.put(data.token, service);
	20
		            ....
	21
		        } catch (Exception e) {
	22
		            ...
	23
		        }
	24
		    }
	抽出主要代码分析ActivityThread. handleCreateService()方法里面主要做这几件事
	3.1 通过pms找到要启动的Service配置信息，然后通过反射生成Service对象
	3.2 创建ContextImpl对象，然后调用service.attach方法设置到ContextWrapper.java的mBaseContext变量里面。
	
	那现在就明白了，service.startActivity()->ContextWrapper.startActivity()->ContextImpl.startActivity()
	然后再ContextImpl.startActivity里面会检查Intent的参数是否包含FLAG_ACTIVITY_NEW_TASK，从而出现这个异常。
	
	三.    解决这个异常后会出现问题？
	有些同学就会说了，在Service里面启动Activity必须要有FLAG_ACTIVITY_NEW_TASK参数，那么我们添加上不就可以了？如下：
	intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
	那么这样会带来什么问题呢？
	这样带来的问题就是在最近任务列表里面会出现两个相同的应用程序，比如你是在电话本里面启动的，那么最近任务列表就会出现两个电话本；因为有两个Task嘛！
	那怎么解决呢？其实也非常好解决，只要在新的Task里面的Activity里面配置android:excludeFromRecents="true"就可以了。表示这个Activity不会显示在最近列表里面。
	
	四.    Activity.startActivity()为什么不出现这个异常呢？
	要回答这个问题，需要看下Activity.startActivity()调用到哪里去了
	代码文件：
	frameworks\base\core\java\android\app\Activity.java
	代码：
	1
		public void startActivity(Intent intent) {
	2
		   this.startActivity(intent, null);
	3
		}
	接下来会调用startActivityForResult()->然后一路调用到Ams去启动Activity;
	原来如此，Activity重写了startActivity()方法...
	
	
	五.    Android 为什么要这么设计?
	那现在来回答这个问题，为什么Android在Service 里面启动Activity要强制规定使用参数FLAG_ACTIVITY_NEW_TASK呢？
	我们可以来做这样一个假设，我们有这样一个需求：
	我们在电话本里面启动一个Service，然后它执行5分钟后，启动一个Activity
	那么很有可能用户在5分钟后已经不在电话本程序里面操作了，有可能去上网，打开浏览器程序了。
	5分钟后，此时当前的Task是浏览器的task，那么弹出Activity，如果这个Activity在当前Task的话，也就是浏览器的Task；那么用户就会觉得莫名其妙；因为弹出的Activity和浏览器在一个Task，本来这个Activity应该属于电话本的。
	
	所以，对于Service而言，干脆强制定义启动的Activity要创建一个新的Task.
	这种设计，我觉得还是比较合理的。

