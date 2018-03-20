---
title: Jenkins自动打包IOS
date: 2017-12-26
tags: iOS
categories: iOS Tips
---

#### Jenkins自动打包IOS与分发fir

##### 背景

CI（持续集成）是现在非常流行的软件开发实践，可以自动化的持续打包部署代码或者安装包，减少了手工干预，大大的提高了工作效率，因此，不论如何，都需要对它做一些了解和实践。

每次开发完，我们要做的是先找开发打包，打完包后拿来放到fir.im上，再生成二维码，最后把二维码做到我们内部分发的网站上分发，然后才能开始测试。整个流程很长，如果当天有很多小问题不断的修复，那感觉真是坑爹，一两个小时时间就浪费在这个过程上。使用CI能够很好的解决这个问题。

<!-- more -->

##### 环境

我使用的是现在使用比较普遍的Jenkins。打包的对象是IOS。所以首先，你必须有一个Mac。

>操作系统：Mac OS X EI Caption 10.11.5 
>Jenkins版本：2.17

##### 安装

Jenkins的安装非常简单了，只要直接安装DMG就行了。安装完毕之后，进入*http://localhost:8080*就可以看到Jenkins的配置界面了，由于是Web端配置，所以使用起来非常友好。

一下两个命令可能会经常用到，由于安装Jenkins的时候，默认是自动启动的，有时候为了减少不必要的资源消耗，可以关闭Jenkins。

```
Start Jenkins: sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
Stop Jenkins: sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist12
```

第一条命令是启动，第二条命令是关闭。

##### 工作流

本次使用Jenkins的工作流如下图所示（Jenkins不仅仅只有这些功能，这些只是本次使用的功能）

![Jenkins工作流](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-Jenkins%E5%B7%A5%E4%BD%9C%E6%B5%81-1.png)

#### 构建工程

使用Jenkins部署打包IOS程序有一个最大的前提，就是你必须要有一个Mac，否则免谈。

Jenkins的安装就不说了，百度一下一大堆，首先要新建一个自由风格的项目。

##### 项目配置

项目名称和描述都是给自己看的，因此可以凭自己的喜好填写，只要自己能看得懂即可。接下来的构建方式我并没有使用，因为对我来说确实用不到，应用打包完成上传fir.im之后，基本上这个包就可以丢弃了。有需要的可以按需选择。

源码管理，我选择SVN，其他也是一样的，配置仓库地址和账户。

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713489712796.jpg)

*Credentials*是用户，右侧的ADD按钮可以添加用户，其他项默认配置即可。

构建触发器也是非必须选项，如果不配置则需要手工触发，我这里配置5分钟检查一次代码仓库。

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713490615348.jpg)

具体的配置规则百度即可。

接下来是Xcode的配置，IOS打包有两种方式，一种是命令行打包，直接写打包脚本即可。另一种是Jenkins的Xcode插件来实现打包。我使用的是Xcode插件来配置，因为我们的开发说他对项目的工程架构有做过改动，我也尝试过一下使用命令行，由于对shell一窍不通，所以还是放弃了。

- Xcode插件——General build settings

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713495959005.jpg)

`Target`是构建的目标。如果留空，那么就会构建所有的目标。

`Clean before build`这个选项建议勾选，构建之前最好Clean一下，至于为什么，我也不知道，我们开发之前打包的时候没有Clean，有时候会出一些莫名其妙的bug。

`Configuration`此处填写Release或者Debug，指的是IOS打包的类型，Debug版本还是Release版本。

`Pack application and build .ipa`需要打钩，因为我们需要打包成ipa文件来安装。

`ipa filename pattern`打包成ipa文件的名称，就是最终的xxx.ipa。

`Output directory`打包输出的路径，我这里填写的`${WORKSPACE}`指的是`/Users/Shared/Jenkins/Home/workspace`

其他项我使用的都是默认的，需要的请自己研究，项目右侧的问号都有很明确的提示，唯一不爽的就是提示都是英文的。。。。。

- Xcode插件——Code signing & OS X keychain options

这部分应该算是比较重要的地方，因为这里是配置打包签名的地方。

首先需要在钥匙串访问中找到开发者证书对证书进行权限开放，否则Jenkins无法获取证书。

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713504489601.jpg)

然后在配置项中做如下配置。

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713505500822.jpg)

`Keychain path`是固定填写*${HOME}/Library/Keychains/login.keychain*

`Keychain password`指的是你的授权密码，不是证书密码。

注意：*${HOME}*目录指的是Jenkins的根目录，也就是*/Users/Shared/Jenkins*，如果你的jenkins是新装的，你会发现Library目录中压根就没有Keychains目录。你需要去*/Users/SvenWeng/Library*目录下把Keychains目录复制过来。

- Xcode插件——Advanced Xcode build options

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713511269062.jpg)

`Xcode Project File`是项目的地址，xxx.xcodeproj的地址。

`Build output directory`是构建后的输出地址，我这里设置于ipa文件地址一致。

注意：如果你的项目是使用*.workspace*。请使用`Xcode Workspace File`，留空`Xcode Project File`.

到此，Xcode插件的配置就完成了。

##### 构建后执行

这里我使用了`Post-Build Script Plug-in`插件。

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713515952747.jpg)

进入我们构建后的目录，使用fir-cli的命令把打包后的ipa文件上传到fir.im上，其中`-T`参数是使用fir.im的token，可以注册后登陆fir.im查看自己的token。`-Q`参数是上传完毕之后把对应的二维码下载到当前目录下。

下方还有一个勾选项`Execute script only if build succeeds`。构建成功才执行这个命令，也就避免了重复上传的问题。

##### 邮件通知

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713518531932.jpg)

我使用的是Jenkins自带的邮件功能，看勾选的结果是构建失败才会发送邮件，经过我自己的测试，构建成功也能收到邮件，不过我配置的163服务邮箱经常坑爹的拒绝我使用smtp发送邮件。

邮件的内容还是比较人性化的，比如如果档次构建SVN有变动，Jenkins会把变动日志和打包的日志一起发给你，比如这样

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713523250962.jpg)

至此，整个构建就结束了，可以手工触发来看看结果。

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713519707240.jpg)

蓝色表示构建成功，红色表示构建失败。

#### 一些注意点

配置过程中总是会出这样那样奇奇怪怪的错误。我整理了一下我自己遇到的问题。

- scheme name是什么鬼

好吧，其实我也不太了解，翻译上是计划名称。其实这个是Xcode的一个配置（不知道这么说准确不准确）。进入方式：*Xcode menu > Product > Scheme > Edit scheme*点击*Manage Scheme*

![img](http://7xsgl3.com1.z0.glb.clouddn.com/2016-08-16-14713527408190.jpg)

被我框起来涂掉的地方就是Scheme名称。

- Code Sign error: No matching provisioning profile found: Your build settings specify a provisioning profile with the UUID “XXXXX”, however, no such provisioning profile was found.

这个报错还是由于证书的问题，Xcode工程的证书放在*/Users/SvenWeng/Library/MobileDevice/Provisioning Profiles*这里，而Jenkins的根目录是没有这个东西的，之前我放的是一个错的证书，所以导致了这样的报错。需要把工程证书拷贝到Jenkins根目录下的*Library*中。

- FATAL: Unable to unlock the keychain.

这个报错是由于你解锁*keychain*的密码不正确，在配置项中需要配置授权密码（也就是你的mac的登录密码），而不是证书的密码。

- xcodebuild: error: ‘./workspace/.xcworkspace’ does not exist.

这个报错很明显了，我的工程使用的是*.xcodeproj*，而不是*.scworkspace*，因此在Xcode插件配置的时候需要配置`Xcode Project File`而不是`Xcode Workspace File`

- ERROR: Failed to update svn://xxxx 
  org.tmatesoft.svn.core.SVNException: svn: E204899: Cannot create new file ‘/Users/Shared/Jenkins/Home/workspace/HLSC/.xcodeproj/.svn/lock’: No such file or directory

这个报错也很明显了，是SVN配置的有问题。

- xcodebuild: error: The project ‘xxx.xcodeproj’ does not contain a target named ‘XXX’.

这个报错是target named指定错误，检查一下Xcode的工程是否有相关配置，我报这个错是因为正确的名字是xxx_iOS,而我打成了xxx_IOS，就是一个大小写的区别。

#### 后续

后续可以做的事很多啦，比如写个Python脚本来处理本地的文件，把打包的内容做一下归档，然后生成的二维码通过FTP方式上传到内部的分发网站上。编写自动化单元测试，每次代码改动的时候做一下自动化测试，把Android客户端也用这种方式自动打包出来等等。。。。

#### 总结

持续集成看上去很高大上，其实走一遍流程，也并不是那么困难，困难的地方Jenkins都帮你处理好了。

配置的时候如果是IOS开发人员，或者对IOS有一定的了解，那么配置起来应该是非常快的，否则最好有一个人来指导你，今天配置的时候我们的IOS开发总能在关键点上给一个方向，省去很多时间，不然现在可能都还配不出来，很多报错网络上找不到，即使找到了，大多也是英文的，虽然看得懂，但是看起来总是有点不爽。

耐心，耐心，耐心，在配置过程中一定会有各种各样奇奇怪怪的报错，有时候看起来很烦，但是一定要有耐心，多静下心来想想，多看看日志，日志的报错一般来说都是非常清晰准确的。
