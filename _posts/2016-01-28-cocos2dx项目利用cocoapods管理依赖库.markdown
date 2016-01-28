---
layout: post
title:  "cocos2dx项目利用cocoapods管理依赖库"
date:   2016-01-28 21:56:52
categories: iOS Cocos2d-x Cocoapods
---

# cocos2dx项目利用cocoapods管理依赖库

目前项目是一个 cocos2dx 引擎和应用混编的应用。所以会涉及到很多iOS原生的功能开发。所以引入了cocoapods来管理依赖。

本身cocos2dx 引擎也是跑在原生的iOS window 上。理解了这点之后就可以很顺利的在游戏和应用切换。废话少说，用一个新建的cocos应用演示下如何利用cocoapods加入第三方依赖库。并且展示下碰到的问题。

首先`cocos new app -l lua`创建cocos应用

然后切换到`frameworks/runtime-src/proj.ios_mac`的iOS路径

然后初始化cocoapods `pod init`，之后会创建一个`Podfile`，在`Podfile`中随意加入你要加入的库，例如'AFNetworking'


    # Uncomment this line to define a global platform for your project
    # platform :ios, '8.0'
    # Uncomment this line if you're using Swift
    # use_frameworks!
    
    target 'app-mobile' do
    pod 'AFNetworking'
    
    end
    
    target 'app-desktop' do
    
    end

然后`pod install`,不出意外会报错:

![](/images/Screen_Shot_2016-01-27.png)


其实仔细读警告，是因为cocos设置的参数不合理，所以我们需要在`HEADER_SEARCH_PATH`以及`OTHER_LDFLAGS`中增加`$(inherited)` flag，参考如下:

![](/images/Screen_Shot_2016-01-28.png)
  

最后再次`pod install`即可


