---
title: React Native
date: 2018-01-14
tags: iOS
categories: iOS Tips
---


<http://www.jianshu.com/p/ff36c9dcf50b>

<http://www.jianshu.com/p/5c0659ace329>

<http://www.tuicool.com/articles/qEJbAfy>

<http://blog.163.com/huang_tai_shan/blog/static/25917911120168245653718/>

<http://www.jianshu.com/p/3dc9d70a790f>  集成到原生项目中

<http://www.open-open.com/lib/view/open1463060059009.html>   集成到原生项目中

<http://www.tuicool.com/articles/BfInEv> (配合这个创建自工程的方法<http://www.open-open.com/lib/view/open1452090394558.html>)集成到原生项目中

<!-- more -->

记住一定要有  JavascriptCore.framework

还有如果报错:"std::__1::mutex::~mutex()", referenced from:       -[RCTModuleData .cxx_destruct] in libReact.a(RCT

修改方法如图：

![img](http://img.blog.csdn.net/20170228141159376)

如果cmd+R无法刷新 试试把模拟器的硬键盘打开

<http://www.lcode.org/react-native-integrating/>  一个RN大拿

<http://www.jianshu.com/u/9895046b765d>

<http://www.jianshu.com/p/2b1c14f1a9a8>  打包

示例应用：<https://github.com/fbsamples/f8app>

<http://www.tuicool.com/articles/bIz6Bnm>  去哪儿app的RN实践讲解

Webstorm 破解版：<http://www.jianshu.com/p/492309c60348>

react-native run-ios报错

localhost:AwesomeProject allan$ react-native run-ios

Found Xcode project AwesomeProject.xcodeproj

xcrun: error: unable to find utility "instruments", not a developer tool or in PATH

Command failed: xcrun instruments -s

xcrun: error: unable to find utility "instruments", not a developer tool or in PATH

解决办法：（注意前提是finder的应用程序中得有Xcode.app这个名字的Xcode）

localhost:AwesomeProject allan$ sudo xcode-select -s /Applications/Xcode.app/Contents/Developer/