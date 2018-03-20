---
title: 插件管理
date: 2018-01-31
tags:
categories: iOS Tips
---

#### 安装和卸载

```objective-c
安装此管理工具命令：
curl -fsSL https://raw.github.com/alcatraz/Alcatraz/master/Scripts/install.sh | sh
```

```objective-c
卸载此管理工具命令：
rm -rf ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins/Alcatraz.xcplugin
```

<!-- more -->

```objective-c
插件管理工具Alcatraz的安装和使用
注释插件「VVDocumenter-Xcode」
扫面图片工具「KSImageNamed」
代码对齐工具「XAlign」
快速进入沙盒工具「ZLGotoSandboxPlugin」
右边显示小地图「SCXcodeMiniMap」
代码补全支持模糊查询「FuzzyAutocomplete」
简单直观的标记本次commit修改的部位「GitDiff」
把xcode的编辑页面可支持vim操作「xvim」
敲代码时debug视图自动隐藏「BBUDebuggerTuckAway」
高亮显示正在编辑的行「Backlight-for-XCode」
pod相关的操作可以在xcode菜单进行「cocoapods」
输入颜色时有一个色板给你选「ColorSense」
switch枚举的时候会自动生成代码「SCXcodeSwitchExpander」
一键删除Derived Data「DerivedData Exterminator 」
debug栏打印时自动把/ueo6转化成汉字「DXXcodeConsoleUnicodePlugin」
快捷键标记，和统一查看「XToDo」
将JSON格式化输出为模型的属性「ESJsonFormat」
用来生成 @3x 的图片资源对应的 @2x 和 @1x 版本「RTImageAssets」
```

因为一些原因 xcode8.0之后都不能直接使用了 需要借助[update_xcode_plugins](https://link.jianshu.com/?t=https://github.com/inket/update_xcode_plugins) 这个项目来实现8.0+上的Alcatraz正常使用

1.下载解压到桌面

2.cd到这个文件夹中去

3.在终端输入： gem install update_xcode_plugins

4.继续输入：update_xcode_plugins

5.输入命令：update_xcode_plugins --unsign

6.最后输入：update_xcode_plugins --install-launch-agent

重启xcode  会有load bundle这个按钮 选中这个点击即可！至此，xcode8.0+上使用Alcatraz成功。。。。

