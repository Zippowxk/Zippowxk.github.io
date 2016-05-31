---
layout:     post
title:      "Using and Create Library in iOS"
subtitle:   " \"在iOS中创建、使用静态库(内含大量xib文件)\""
date:       2016-5-31 12:00:00
author:     "Wxk"
header-img: "img/post-bg-miui6.jpg"
tags:

    - 第三方拓展
    - 静态库    
---


## 前言

> “最近接手的项目中，有这样一个需求，把一部分UI和功能打包成静态库或者动态库给其他公司使用。但是这个需求一开始是没有的，所以需要把做完的工程需要重新整理一下。”

我想很多人都碰到过类似的问题，就是需要把你的某些功能共享给其他人，但是不开放源码，这很正常。比如涉及到加密算法的问题。这时候一般都会采用静态库，相对于静态库还有动态库，iOS8之前是不允许动态库上传App Store的，iOS 8 以后就有了动态库。
动态库和静态库的区别主要在于重复利用率，静态库在不同的应用中被使用，会重复被copy，比如你的大众点评客户端和美团外卖客户端都用了微信支付，如果都用的静态库，那么每个静态库都会被加载到内存中，但是如果使用了动态库，那么就会加载同一个。就像UIKit在系统中只有一个一样，不需要重复加载。
我们本期不讨论动态库，先学习一下静态库---Library

## 一、认识library


### 1.1 静态库构成
在开发过程中，经常会见到.a的文件，比如百度地图、高德地图等等，一般一个静态库会包含下面这些东西：

1.	后缀为.a的二进制文件
2.	后缀为.h的头文件
3.	后缀为.bundle的资源文件
4.	说明文档（一般不含在内）


### 1.2 静态库的功能

1. .a文件里编译了所有的执行文件、隐藏的头文件、被包含的.a文件
2. .h文件就是暴露给用户的头文件 
3. .bundle是一个文件夹，就是一个封闭的沙盒，后面我们说怎么打包Bundle

### 1.3 使用时需要注意的问题

1. 一般使用静态库时，需要注意静态库依赖的动态库。 这里需要强调，在我测试的结果看来，静态库可以包含其他静态库，但是不能包含其他的动态库。后面具体讲一下我的结论。
2. 静态库中如果有使用到单独的文件作为类目（Category），那么我们在使用时需要在对应的Targets中的Build Settings下的 Other Linker Flags，添加-ObjC 或者-all_load。可能很多人都有过类似的操作，但是不清楚为什么。具体解释一下的话就是链接的时候，有选择的链接文件，如果不懂可以看这里[OtherLinkerFlags](http://www.cnblogs.com/robinkey/archive/2013/05/27/3101095.html) <br>[![2016-05-31_22-31-10.md.png](http://imgchr.com/images/2016-05-31_22-31-10.md.png)](http://imgchr.com/image/PTi)
3. 静态库使用时请注意要求的最低版本，否则会有意想不到的结果。
4. 一般静态库在制作时会有两种，真机版本和模拟器版本，主要是因为两者支持的cpu不同。所以也要注意。我们后面会补充如何通过命令合并两者。
5. 其他坑等待各位客官在评论区补充吧

## 二、为什么使用静态库

### 2.1 安全性
其实使用静态库的目的就一个，那就是保密。我们不希望第三方看到我们的某些机密数据，但是还需要开放给第三方使用我们提供的某些服务，这是否就需要静态库来解决这种问题。比如我们常见的微信SDK，支付宝SDK，高德地图SDK等等。因为源代码已经被编译成了.a文件，那么想获得源码只能通过反编译。关于静态库反编译的问题我还没有研究过，等大家补充。


### 2.2 通用性

不需要针对某用户单独制作，所有第三方开发者通用。

### 2.3 可传递

文件结构简单，很方便就可以传输。比如pods就很方便的可以管理很多第三方库。


## 三、制作静态库

因为目前网上有太多的制作教程了。我这里不同的是：

1. Xcode 7.2
2. __大量的xib文件 （重点）__
3. __现有工程，后有打包SDK的需求 （重点）__

不得不说一下第三条，在我们已经将要完成整个工程的时候，客户提了这么一个需求。起初我觉得没什么难度，不会需要太久，我就说帮他们免费做了。后来发现自己入坑了。<br>
背景是已经有了10多个功能界面，现在需要整体打包成SDK提供给第三方使用。10多个功能模块中还包含了一个蓝牙静态库，支付宝、微信支付库，百度地图库。10多个用xib绘制的ViewController。几十张图片。<br>
[![2016-05-31_23-00-12.md.png](http://imgchr.com/images/2016-05-31_23-00-12.md.png)](http://imgchr.com/image/PTz)
<br>请原谅我用拼音首字母命名的懒惰（这是我的一个恶趣味）
好了，需求说完了。那么接下来制作就开始了，一开始我感觉自己头都要炸了。有这么几个问题围绕着我：

1. 在原有工程上添加Target Or 新建一个工程
2. 所有的图片资源使用要重写(imageNamed:获取失效)。
3. xib里的图片还能用吗？
4. xib文件创建ViewController需要重写

然后我怒抽1包烟，操作了两把Dota。决定先写一个Demo测试一下我的疑问，然后果断新建一个工程来做。
下面介绍步骤：

### 编译.a文件 Step1-7

### Step1 新建工程
<br>
[![2016-05-31_23-21-39.md.png](http://imgchr.com/images/2016-05-31_23-21-39.md.png)](http://imgchr.com/image/PT9)
<p>这一步没有什么好注意的<p>
<br>

### Step2 删掉默认文件
<br>
[![2016-05-31_23-26-25.md.png](http://imgchr.com/images/2016-05-31_23-26-25.md.png)](http://imgchr.com/image/PTB)
<br>

### Step3 创建一个带有Xib文件的ViewController
<br>
[![2016-05-31_23-28-08.md.png](http://imgchr.com/images/2016-05-31_23-28-08.md.png)](http://imgchr.com/image/PTS)
<br>
在xib上添加一个UIImageView，并且添加一张图片并设置好。<br>
[![2016-05-31_23-48-29.md.png](http://imgchr.com/images/2016-05-31_23-48-29.md.png)](http://imgchr.com/image/PTR)
<br>

### Step4 创建一个不带有Xib文件的RootViewController
<br>
在viewDidLoad里添加如下代码

	- (void)viewDidLoad {
	    [super viewDidLoad];
	
	    
	//    测试能否使用 imageNamed: 方法
	    UIImageView *imageV = [[UIImageView alloc] initWithFrame:self.view.frame];
	    [imageV setImage:[UIImage imageNamed:@"wxpay.png"]];
	    [self.view addSubview:imageV];
	    
	    
	    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
	        
	//        测试能否使用init 直接加载xib文件
	//        测试xib中能否直接使用图片
	        TestViewController *vc = [[TestViewController alloc] init];
	        [self presentViewController:vc animated:YES completion:nil];
	
	    });
	}

### Step5 设置暴露的头文件
[![2016-06-01_00-04-02.md.png](http://imgchr.com/images/2016-06-01_00-04-02.md.png)](http://imgchr.com/image/PTg)

当然暴露的不一定非得是头文件，执行文件也可以暴露给第三方。

### Step6 Run并获得静态库
[![2016-06-01_00-08-47.md.png](http://imgchr.com/images/2016-06-01_00-08-47.md.png)](http://imgchr.com/image/PT4)
<br>注意这里，如果选择真机，编译出来的就是真机版本的静态库，选择模拟器对应的就是模拟器版本的静态库。我们两个分别都Run一次。然后在build目录下就可以找到两个静态库了。在window-projects中找build目录。<br>
[![2016-06-01_00-13-13.md.png](http://imgchr.com/images/2016-06-01_00-13-13.md.png)](http://imgchr.com/image/PTD)
<br>
[![2016-06-01_00-16-04.md.png](http://imgchr.com/images/2016-06-01_00-16-04.md.png)](http://imgchr.com/image/PTK)
<p>然后你会在这里看到两个文件目录， __Debug-iphoneos__ 就是真机版本， __Debug-iphonesimulator__ 就是模拟器版本。<p>

合并两种版本的方法 `lipo -create -output` 命令，很简单的命令,请看这里[命令详解](http://blog.csdn.net/cuiweijie3/article/details/8671240)。

### Step7 测试静态库

这一步就不上图了，直接说结果：

1. [UIImage imageNamed:@"wxpay.png"] 失效。因为.a文件中不能包含图片文件
2. `测试能否使用init 直接加载xib文件 `失败，因为同样不能包含xib文件
3. `测试xib中能否直接使用图片`测试失败，以为xib文件本身就没有被编译进来。


### 编译bundle文件 Step8-10

### Step8 添加Target
[![2016-06-01_00-31-31.md.png](http://imgchr.com/images/2016-06-01_00-31-31.md.png)](http://imgchr.com/image/PTt)

这里需要创建OS X的bundle，然后修改成iOS版的，怎么修改？看下图：<br>

[![2016-06-01_00-33-45.md.png](http://imgchr.com/images/2016-06-01_00-33-45.md.png)](http://imgchr.com/image/PTE)

### Step9 给Bundle添加包含的文件
[![2016-06-01_00-35-45.md.png](http://imgchr.com/images/2016-06-01_00-35-45.md.png)](http://imgchr.com/image/PTO)


### Step10 Run并且获得.bundle文件
[![2016-06-01_00-39-34.md.png](http://imgchr.com/images/2016-06-01_00-39-34.md.png)](http://imgchr.com/image/PTa)
<br>注意run的时候一定要选好target，否则你还是执行的静态库文件的target哟。这个应该是不区分真机和模拟器。

### Step11 修改imageNamed功能




在静态库中的RootViewController.h添加一个类目：

	@interface UIImage (LibFindImage)
	
	+(UIImage*)imageNamed_lib:(NSString *)name;
	
	@end

在静态库中的RootViewController.m实现类目：


	@implementation UIImage (LibFindImage)
	
	
	+(UIImage*)imageNamed_lib:(NSString *)name{
	
	    
	    NSBundle* bundle=[NSBundle bundleWithPath:[[NSBundle mainBundle]pathForResource:bundleName ofType:@"bundle"]];
	    [bundle load];
	    
	    
	    if ([name pathExtension]) {
	
	        
	        return [UIImage imageWithContentsOfFile:[bundle pathForResource:[name stringByDeletingPathExtension] ofType:[name pathExtension]]];
	    }
	    
	
	    return [UIImage imageWithContentsOfFile:[bundle pathForResource:name ofType:@"png"]];
	}
	
	@end


### Step12 加载Xib功能


给TestViewController添加如下方法：

	-(instancetype)init{
	    
	    NSBundle *bundle = [NSBundle bundleWithURL:[[NSBundle mainBundle] URLForResource:bundleName withExtension:@"bundle"]];
	    if (bundle) {
	        NSFileManager *fm = [NSFileManager defaultManager];
	        if ([fm fileExistsAtPath:[NSBundle pathForResource:NSStringFromClass([self class]) ofType:@"nib" inDirectory:[bundle bundlePath]]]) {
	            
	            if (self = [super initWithNibName:NSStringFromClass([self class]) bundle:bundle]) {
	      
	            }
	            
	            return self;
	            
	        }
	        
	    }
	    if (self = [super init]){
	        
	    }
	    
	    return self;
	}


需要注意的是：

1. xib编译后后缀会变为.nib
2. 为了可以减少代码量和方便修改，最好有一个父类BaseViewController，统一修改init方法，会更加简单。

### Step13 测试静态库+bundle

当然是成功了~

## 四、制作我的工程SDK

有了上面的测试，事情简单多了，我首先还测试了静态库能否包含静态库，发现可以。但是不可以包含动态库。原因大概是静态库和动态库的加载逻辑不通，静态库程序一打开就要编译链接进去。动态库只有在运行时才会被掉起。所以不会相互包含。 所以支付宝最新的SDK只能作为依赖库使用。

然后新建工程->添加原工程中的ViewController、资源文件、其他静态库->制作bundle->修改BaseViewController的init方法->修改所有imageNamed:方法为imageNamed_lib ->bingo打包


这里遇到了一个小问题，当我使用了单独的文件作为类目时，引用静态库时我尝试使用-ObjC解决，但是不可以，出现了`duplicate symbols for architecture arm64`，最终尝试了-force_load 也不行，最后还是把类目合并到了其他的代码中，没有单独使用。各位可以探讨一下原因。

这次文章截图比较丰富，暂时不传源码了，主要在制作过程，代码含量不高。如果有需要单独在评论区留言吧。欢迎大家批评指点。儿童节快乐~

声明：zippowxk原创文章，如要转载请联系luxuntec@163.com，保留法律权利

