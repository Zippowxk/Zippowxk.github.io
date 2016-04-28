---
layout:     post
title:      "PeriodicAdStands"
subtitle:   " \"创造完美的广告轮播页\""
date:       2016-4-28 12:00:00
author:     "Wxk"
header-img: "http://bizhi.33lc.com/uploadfile/2015/0611/20150611012625286.jpg"
tags:
    - UI设计
---

> “一般人：感觉还不错，能用就行啦    
> 优秀的人：我要的是完美的解决方案”


## 前言

最近忙于工作，少有更新，项目中最近遇到了很常见的轮播图，以前使用timer的方式自动播放不够完美，有以下几点：

1. timer 不如CADisplayLink精准，其实主要是 Runloop的原因
2. timer的运行模式更改为CommonModes以后会造成用户体验的下降，就是说当我们滚动下面的TableVIew之类的控件时，轮播图依旧会动， 传统的解决方案是使用滚动判断，或者把timer放在DefaultMode中，但是会有另一个问题。两者不可兼得。
3. 最终我们需求就是解决这个悖论，既要实现在触发界面上其他任何事件时停止滚动，又要在触发结束后立马开启自动滚动

最终诞生了本文，使用稍微复杂一点的RunLoop解决。 先上几个对比的案例：

#### 1. 优酷iOS V5.4.2 首页
Hades君不要介意啦

![Image1](http://imgchr.com/images/bug37a23f.gif)

仔细的同学会发现，在特定时间去滚动下面的视图，会造成轮播图快速滚动一次，也就是说之前有一个事件没有被取消或者时间算法搞错了。正常状况下间隔为3秒。

#### 2. BiliBili iOS 4.12.1 首页

![Image1](http://imgchr.com/images/bug32.gif)

这个案例更加明显，正常情况下5秒的时间间隔，bug测试后变成了不到1秒。

#### 3. 优酷iOS V5.4.2 首页

这个测试的是另外一个问题，就是用ScrollView作为容器时，Youku的工程师为了减少内存的浪费，使用了三张重用图，但是问题是没有考虑到渲染问题，测试方法如下：快速连续滑动两次就会无法继续拖拽。这个问题 B站iOS端已经渲染好，但是仍旧无法继续拖动，这个问题留着待解决。

![Image1](http://imgchr.com/images/bug33.gif)


好了，掰扯完了这么多，就是给出大家最终解决方案。 使用CollectionView+RunLoop方案

---

## 正文

### 1. 构造CollectionView 

这一步着实很简单，任何有过UI搭建基础的同学，简单看一下CollectionView的API就很简单的可以实现水平方线的图片滑动，设置一下Cell的宽度和CollectionView的PagingEnabled即可。

### 2. 循环滚动

循环滚动大家应该很熟悉了，思路是：在左右两端各加上一个Cell，如果想要展示 1 2 3 4 5 个数字，那么我们的数据应该是这么保存 5 1 2 3 4 5 1 ，相信大家仔细一个就懂了。如果不懂就不妨下载源码一看便知。

用CollectionView的好处我也跟大家说咯，既可以重用Cell达到节约内存的效果，又可以防止快速滑动时渲染失败的尴尬。

### 3. 自动播放 （CFRunLoop）

这一步是最为复杂的。当我们看到这样的命题，很自然脑海中闪现出一下几个问题：

1. 用户在操作界面时（包括手动滚动轮播窗口，操作其他UI空间），不应该继续自动播放。
2. 用户停止交互时继续播放。
3. 轮播页不可见时，应该停止播放 

好了接下来是解决方案：

1. 最直观的NSTimer
2. 更加精准的CADisplayLink
3. 可以炫技装*的CFRunLoop 

在使用NSTimer时 很多人会遇到一个问题就是当我们使用最简单的方式创建NSTimer并且使用时，经常会因为UI的交互造成计时器停止。其实是因为Runloop机制把CPU资源优先分配给了更加需要处理的UI事件。解决的办法就是 把NSTimer添加到当前RunLoop时，选用CommonModes。那么问题是这样做了以后Timer会更准了，我们如何确定有UI事件，并且停止呢。更多的做法是判断一下多有的ScollView是否开始被拖拽，一旦开始就停止NSTimer，当拖拽结束再开始NSTimer。但是除了这些呢，比如突然弹出的警告框，比如其他一些UI事件呢，我们无法挨个添加监听。 所以NSTimer和CADisplayLink都可以放弃了。只能用CFRunLoop了。


--- 
以下高能

使用Runloop方式很多，NSRunLoop可自定义性能较少，CFRunLoop可以直接监听Loop中的状态回执。所以CFRunLoop必须要用了。关于Runloop的介绍，接下来我会专门写一篇文章。不去管why，我们暂时关心一下how。

#### 3.1 使用下面的方法来添加监听：


	-(void)addRunLoopObserver{
	    
	    // The application uses garbage collection, so noautorelease pool is needed.
	    NSRunLoop*myRunLoop = [NSRunLoop currentRunLoop];
	    _goToSleep = NO;
	    // Create a run loop observer and attach it to the runloop.
	    CFRunLoopObserverContext  context = {0, (__bridge void *)(self), NULL, NULL, NULL};
	    observer =CFRunLoopObserverCreate(kCFAllocatorDefault,
	                                      kCFRunLoopAllActivities, YES, 0, &myRunLoopObserver, &context);
	    
	    if (observer)
	    {
	        CFRunLoopRef    cfLoop = [myRunLoop getCFRunLoop];
	        CFRunLoopAddObserver(cfLoop, observer, kCFRunLoopCommonModes);
	    }
	    
	    
	}

#### 3.2 实现监听：

	void  myRunLoopObserver(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info){
	    
	    //    进入休眠
	    if (activity == 1UL << 5) {
	
	    }else if (activity == 1UL << 6){
	        //    退出休眠
	    }
	    
	    
	}

这里需要使用C的函数来监听，监听的状态可以分为7种：



	/* Run Loop Observer Activities */
	typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
	    kCFRunLoopEntry = (1UL << 0),  // 进入Loop
	    kCFRunLoopBeforeTimers = (1UL << 1),  //Timers的事件注入前
	    kCFRunLoopBeforeSources = (1UL << 2),  // soures的事件注入前
	    kCFRunLoopBeforeWaiting = (1UL << 5), // 马上进入休眠
	    kCFRunLoopAfterWaiting = (1UL << 6), // 休眠结束
	    kCFRunLoopExit = (1UL << 7), //  退出Loop
	    kCFRunLoopAllActivities = 0x0FFFFFFFU  //Any Activities
	};

所以我们需要的就是退出休眠和进入休眠前的两种状态，来查看是否有需要执行的代码，一般情况下就是UI事件，当然如果有其他NSTimer的话最好不用该方法。当然也可以再进行判断。


需要做的事情无非就是：

1. 进入休眠前开始播放
2. 退出休眠时停止播放
3. 每次播放到下一帧时，先停止之前的监听，然后再重新开始监听。
4. 开始监听时使用<br>`- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay inModes:(NSArray<NSString *> *)modes;` <br>在5秒以后播放下一帧，当不足5秒就退出休眠时使用 <br>`+ (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget selector:(SEL)aSelector object:(nullable id)anArgument;`取消之前的操作。


更多的源代码请查看Github 或者 Code4App

