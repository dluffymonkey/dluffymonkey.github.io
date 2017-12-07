---
title: Android Studio gradle 
date: 2017-11-30 20:06
tags: Android
categories: 初识Android
---

#### Android Studio gradle离线包gradle-x.x-all.zip离线配置路径问题详细解决步骤

升级AndroidStudio 到3.0以后新建项目突然一直卡在了building… ，网上说的路径也不清晰，经过各种尝试，终于找到解决办法

一、找到gradle文件夹路径：

Windows系统在 C:\Users\Administrator\.gradle
Mac OS是在：用户/(当前用户目录)/.gradle

**注意**：Mac OS下默认以点开头的目录都不显示，显示方法：

```objective-c
defaults write com.apple.finder AppleShowAllFiles -bool true;
KillAll Finder
```

下载本地gradle文件：[gradle-4.1-all.zip][a](密码: wxrb)、[gradle-4.1-all.zip][b](密码: k5gc)。

[a]: https://pan.baidu.com/s/1pLIDhAr  "Optional Title Here"
[b]: https://pan.baidu.com/s/1mixK0YG  "Optional Title Here"

<!--more -->

二、下载之后，打开一个Android项目

​	找到项目下面的gradle/wrapper/gradle-wrapper.properties，打开，修改distributionUrl后面的gradle版本为下载好的本地的gradle版本，比如http\://services.gradle.org/distributions/gradle-4.1-all.zip，

​	把刚才下载的包放到：用户/(当前用户目录)/.gradle/wrapper/dists/gradle-x.x-all/5wdhop0rtjyqx8wdt9x1nljp5(此处为文件夹名，可能不一样，一长串字母数字的文件夹)

​	然后将你下载的离线的gradle-x.x-all.zip放在这个数字字母组合的目录下即可（不需要解压！！！），重启Android Studio后，新建项目即可，嗖嗖嗖的就构建好了。

​	**注意**：x.x为你下载的gradle版本，在终端弄了以后要注意！可能需要强制退出Finder后重新启动才可以生效！！！



