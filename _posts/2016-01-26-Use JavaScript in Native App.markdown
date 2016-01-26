---
layout:     post
title:      "Use JavaScript in Native App"
subtitle:   " \"Make A Bridge Between Web Pages and Native App\""
date:       2016-1-25 12:00:00
author:     "Wxk"
header-img: "img/2016-1-26.png"
tags:
    - iOS技能
---


### 目录：
1. [简单的跳转操作实现][1]
2. [复杂的业务需求实现][2]



<p id = "simple"></p>

---
### 简单实现详解
当你的项目中有这样一种需求： 1.从native程序中跳转到web网页（比如微信公众账号） 2.从web网页中跳转到某些native的程序页面中，想满足这样的操作其实非常之简单，只需要你学会使用UIWebView中的代理方法：

`- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType`

类似的代码和教程在网络上比比皆是，掌握起来简单，故不做过多介绍，初级选手请戳→[Here][3]

<p id = "complicate"></p>

[1]:	#simple
[2]:	#complicate
[3]:	http://www.jianshu.com/p/d83b824d8b24

---