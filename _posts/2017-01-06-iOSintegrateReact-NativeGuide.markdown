
---
layout:     post
title:      "iOS integrate React-Native Guide"
subtitle:   " \"如何导入react-native到iOS\""
date:       2017-1-6 12:00:00
author:     "Wxk"
header-img: "img/post-bg-2017.JPG"
tags:

    - React-Native
---

>17年的技术方向主要放在 ReactNative，技术栈虽然比较高，但是语言难度比较小，从中领会一些编程思想，会映射到iOS方向上。下面是最近编写的一片指导，没修改直接po上来了。

## 前提
- 硬件：Mac 良好网络环境 
- 软件：macOS环境
- 假定读者具有一定的```Objective-C```和```JavaScript```能力

## 详细步骤：

### 1.软件环境准备
#### 1.1 安装Xcode

React Native目前需要Xcode 7.0 或更高版本。你可以通过App Store或是到Apple开发者官网上下载。这一步骤会同时安装Xcode IDE和Xcode的命令行工具。
> 虽然一般来说命令行工具都是默认安装了，但你最好还是启动Xcode，并在`Xcode | Preferences | Locations`菜单中检查一下是否装有某个版本的`Command Line Tools`。Xcode的命令行工具中也包含一些必须的工具，比如`git`等。

#### 1.2 安装Homebrew

[Homebrew](http://brew.sh/), Mac系统的包管理器，用于安装NodeJS和一些其他必需的工具软件。

	/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 
#### 1.3 安装Node
Node 使用Homebrew来安装[Node.js](https://nodejs.org/). 
> React Native目前需要NodeJS 5.0或更高版本。本文发布时Homebrew默认安装的是最新版本，一般都满足要求。 

	brew install node
	
安装完node后建议设置npm镜像以加速后面的过程（或使用科学上网工具,推荐Shadowsocks）。 

	npm config set registry https://registry.npm.taobao.org --global npm config set disturl https://npm.taobao.org/dist --global
	
#### 1.4 安装Yarn、React Native的命令行工具（react-native-cli）
 [Yarn](http://yarnpkg.com)是Facebook提供的替代npm的工具，可以加速node模块的下载。React Native的命令行工具用于执行创建、初始化、更新项目、运行打包服务（packager）等任务。
 
 	npm install -g yarn react-native-cli 
 	
如果你看到`EACCES: permission denied`这样的权限报错，那么请参照上文的homebrew译注，修复`/usr/local`目录的所有权： ```bash sudo chown -R `whoami` /usr/local ``` 	

#### 1.5 推荐安装的工具

__Watchman__

[Watchman](https://facebook.github.io/watchman/docs/install.html)是由Facebook提供的监视文件系统变更的工具。安装此工具可以提高开发时的性能（packager可以快速捕捉文件的变化从而实现实时刷新）。 
	
	brew install watchman
	
#### 1.6 测试使用

先```cd```到你要存放工程的目录，并使用命令

	react-native init AwesomeProject 
	cd AwesomeProject
	react-native run-ios
	
或者执行命令```react-native init AwesomeProject```后，在Xcode中使用<kbd>Command+R</kbd>编译并执行创建好的```AwesomeProject ```工程。正确结果应该是```模拟器```跑起来，```终端```会弹出Package监听信息窗口
		
### 2.将```javascript```包嵌入原生工程
[参考react-native中文网](http://reactnative.cn/docs/0.39/integration-with-existing-apps.html#%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5)

#### 2.1 创建工程

使用Xcode创建一个工程，或者使用你现有的工程。

#### 2.2 准备JS相关文件

文件应包含：

1. js文件集合
2. 资源文件集合
3. package.json文件 （不需要包含在工程中）

>```javascript```文件导出有两种方式：__1.源码方式 2.打包成JSBundle方式__。如果用源码方式，在Debug时，可以在界面输出警告或者错误信息，方便调试。比如我碰到过缺少依赖库导致的图片不显示的问题，在bundle模式下是无法识别出问题的所在。在```javascript```源码方式下，RN会给出警告提醒```“RCTImage not found”```

#### 2.3 导入文件

如果是JSBundle文件，只需导入```index.ios.jsbundle```文件。如果是```javascript```源码需要和其他资源文件一样，导入时选中创建目录的方式，也就是导入后为蓝色的文件夹。

>根本区别在于两种图片路径在编译后不同，如果默认导入方式，所有文件都会放在沙盒根目录中，这可能导致按照路径访问资源时访问不到。比如```src/img/arrow_img.png```，```javascript```按照相对路径访问，但是如果```arrow_img.png```存放在根目录中，```javascript```就会访问不到

> 对于一个典型的React Native项目来说，一般`package.json`和`index.ios.js`等文件会放在项目的根目录下。而iOS相关的原生代码会放在一个名为`ios/`的子目录中,这里也同时放着你的Xcode项目文件（`.xcodeproj`）。

#### 2.4 安装依赖库

```javascript```程序可能需要相应的依赖库，在```node```中，```package.json```是存放依赖信息的文件（当然还包含其他信息）。

使用npm（node包管理器，Node package manager）来安装React和React Native模块。这些模块会被安装到项目根目录下的`node_modules/`目录中。 在包含有package.json文件的目录（一般也就是项目根目录）中运行下列命令来安装： 

	npm install 
	
#### 2.5 React-Native框架	

为了更简单的集成复杂的RN依赖库到Xcode工程中，强烈建议使用```Cocoapods```
##### 2.5.1 安装Cocoapods

使用命令安装Cocoapods

	sudo gem install cocoapods
	
##### 2.5.2 配置Podfile

使用命令创建一个Podfile

	pod init
	
Podfile的内容如下：	

	# target的名字一般与你的项目名字相同
	target 'ReactnativeJHX' do
	
	  # 'node_modules'目录一般位于根目录中
	  # 但是如果你的结构不同，那你就要根据实际路径修改下面的`:path`
	  pod 'React', :path => 'ReactnativeJHX/rn/node_modules/react-native', :subspecs => [
	    'Core',
	    'RCTText',
	    'RCTNetwork',
	    'RCTWebSocket', # 这个模块是用于调试功能的
	    # 在这里继续添加你所需要的模块
	    'RCTActionSheet’,
	    'RCTAnimation',
	    'RCTVibration',
	    'RCTSettings',
	    'RCTLinkingIOS',
	    'RCTImage'
	  ]
	
	end

需要注意几个地方：

- 第2行中，	```target ```后面应该跟着工程名
- 第6行中，```path``` 应指定好```react-native```的目录
- ```subspecs```指定好需要依赖的库，参照```node_modules/react-native/React.podspec```文件中给出的可用库名称。


##### 2.5.3 检查React.podspec

一个Spec对应一个库文件如下：

	    s.subspec 'RCTImage' do |ss|
	    ss.dependency       'React/Core'
	    ss.source_files   = "Libraries/RCTImage/**/*.{h,m}"
	    ss.preserve_paths = "Libraries/RCTImage/**/*.js"
	  end

```RCTImage```是库的名字，在```podfile```引用的名字。<br>
```dependency```是依赖库路径。<br>
```ss.source_files```是引入的资源目录。<br>
```ss.preserve_paths```是相关的js文件。

>之前我使用Pod时，```React.podspec```中缺少了```RCTImage```，导致图片无法加载。可以手动引入库，或者修改```React.podspec```后执行一次```$ pod install```

##### 2.5.4 执行cocoapods

在podfile目录下，执行命令

	pod install

来引入依赖库，再次打开Finder会发现多了几个文件。使用Xcode打开```ReactnativeJHX.xcworkspace```，cocoapods已经为我们引入了需要的库。


##### 2.5.5 手动添加依赖

找到Target对应```Link binary with library```,点击```+```,点击```Add other...```，找到```node_modules/react-native```中，依赖库基本都在这里。找到需要的依赖库，引入对应的工程文件即可。


#### 2.6 在Cocoa中调起JS

在cocoa代码中，插入一下代码

    NSURL *jsCodeLocation;

    jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"index.ios" withExtension:@"jsbundle"];
    
    RCTRootView *rootView =
    [[RCTRootView alloc] initWithBundleURL : jsCodeLocation
                         moduleName        : @"RnJhxTsys"
                         initialProperties :nil
                          launchOptions    : nil];
    
    self.view = rootView;
    
这里有几点需要注意：

- ```jsCodeLocation``` 按照两种方式（JS源码、JSBundle），有两种方式创建，上面的代码中是JSBundle方式。如果使用源码方式替换为下面的代码：

		 jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index.ios" fallbackResource:nil];
		 
- ```moduleName``` 应该是在```AppRegister```注册的类
- ```RCTRootView``` 可以作为任意一个UIViewController的View使用，也可以添加到任意的View上。

#### 2.7 React-Native和原生交互

设计到跨语言调用，RN提供了很好的支持。[参考react-native中文网](http://reactnative.cn/docs/0.39/native-modules-ios.html#content)

#### 2.7.1 开放接口类给JS

1. 定义一个类```ToolsManager```，确认协议```RCTBridgeModule```
2. 实现 ```RCT_EXPORT_MODULE();``` 开放一个类给JS。
3. 定义方法 showTab:(BOOL)isshow 开放给js调用

		//ToolsManager.m
		RCT_EXPORT_METHOD(showTab:(BOOL)isshow)
		{
		    NSLog(@"show tabbar %d",isshow);
		}

4. js调用方式 

		//index.ios.js
		import { NativeModules } from 'react-native';
		var ToolsManager = NativeModules.ToolsManager;
		ToolsManager.showTab(true);


#### 2.7.2 原生调用JS

1. 引入类```#import "RCTBridge.h"``````#import "RCTEventDispatcher.h"```
2. 初始化```Bridge```
3. 发送事件

		  NSString *eventName = @"eventName"
		  [self.bridge.eventDispatcher sendAppEventWithName:@"EventReminder" body:@{@"name": eventName}];

4. JS处理事件

		import { NativeAppEventEmitter } from 'react-native';
		
		var subscription = NativeAppEventEmitter.addListener(
		  'EventReminder',
		  (reminder) => console.log(reminder.name)
		);
		...
		// 千万不要忘记忘记取消订阅, 通常在componentWillUnmount函数中实现。
		subscription.remove();
		
		
[参考ReactNative中文网](http://reactnative.cn/)
