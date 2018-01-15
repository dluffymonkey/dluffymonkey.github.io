---
title: 友盟统计使用笔记
date: 2017-12-22 10:32:47
tags: iOS
categories: iOS Tips
---

### iOS开发-友盟统计使用笔记

一般项目中集成统计功能随因产品类型不同而使用功能不同，但大多数统计一般只有一个目的，就是记录用户习惯，研究用户习惯，从而为用户带来更好的体验。 
这里记录一下之前在项目中使用友盟统计时的过程和踩过的坑。 

#### 准备阶段

在使用前，准备好如下几点

- 申请友盟后台账号。
- 在账号中选择统计功能，并添加新应用，从而获取AppKey。注册步骤参考[友盟官方文档](http://dev.umeng.com/analytics/ios-doc/integration)。

<!-- more -->

#### SDK获取

与一般SDK一样，有两种方式，[下载SDK](http://dev.umeng.com/analytics/ios-doc/sdk-download)拖入，或用CocoaPods。 
需要注意的是：

- 自行下载SDK需要将SDK中的UMMobClick.framework添加到项目，并添加CoreTelephony.framework和libz.tbd和libsqlite.tbd系统依赖库。
- 使用cocoapods时，要在Podfile内加入` pod’UMengAnalytics-NO-IDFA’`。 并执行 pod install 命令。

#### 集成

首先要在AppDelegate.m文件中进行编辑，集成并开启友盟统计功能。 
这里建议在pch文件中导入头文件，或者在某个全局引用的文件中导入头文件，因为基本项目所有地方都会用到。

```objective-c
#import <UMMobClick/MobClick.h>
```

引入头文件之后，下一步就是开启统计功能。

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    //启动SDK的方法一般写在此处来保证一定执行，理论上说只要在主线程中调用，都会执行
    UMConfigInstance.appKey = @“你的友盟账号中添加应用的AppKey”;
    UMConfigInstance.channelId = @“App Store”;//一般是这样写，用于友盟后台的渠道统计，当然苹果也不会有其他渠道，写死就好
    UMConfigInstance.ePolicy =SEND_INTERVAL; //上传模式，这种为最小间隔发送90S，也可按照要求选择其他上传模式。也可不设置，在友盟后台修改。
    [MobClickstartWithConfigure:UMConfigInstance];//开启SDK
}
```

如果项目要求进行分版本统计，则可加入以下代码

```objective-c
NSString *version = [[[NSBundlemainBundle] infoDictionary]objectForKey:@"CFBundleShortVersionString"];
[MobClicksetAppVersion:version];
```

修正： 
建议添加分版本统计功能，因为如果不加，则友盟会像个疯子一样的乱抓数据，比如，会使用你的build version来进行版本统计。。。 

这就会导致，你提了几个审核的包，里面就会有几个版本。 

建议添加分版本统计功能 

建议添加分版本统计功能 

建议添加分版本统计功能 

重要的事说三遍。。。

编译程序，如不报错，恭喜集成成功。

#### 功能使用

友盟统计为我们提供了多种统计功能，事件统计，页面统计，账号统计，错误分析，社交统计等等。这里只详细说一下事件和页面统计，其他有兴趣可查看[友盟官方文档](http://dev.umeng.com/analytics/ios-doc/integration)。

##### 事件统计

一般是最常用的统计功能，不管是其中的计数事件，还是计算事件，都可以为我们提供足够的用户行为数据。 
事件的统计是通过我们在项目中埋点来触发事件，友盟就会帮我们进行统计。 
当然，这两种都是包含在友盟后台的自定义事件的功能当中，在使用之前，我们需要在友盟后台定义足够的事件ID，然后服务器才会对相应的事件请求进行统计。

###### 计数事件

这是项目中使用最多的功能（目前），使用方式是，在需要统计的用户行为所触发的对应处理方法或点击事件中加入统计埋点方法。 
有下几种传输方式：

- 单纯统计某处用户点击数量（即单纯计次事件）：

```objective-c
[MobClickevent:@"友盟后台定义的事件ID"];//当用户的行为触发对应相应方法后，该代码执行，会自动请求友盟后台，将对应事件ID上传。
```

- 在不同情况下触发的相同行为（即携带参数的计次事件，一般用在从不同位置进入同一个页面之类的事件）：

```objective-c
NSDictionary *dict = @{@"参数1" :@"值",@"参数2" :@"值"};//创建携带参数，key和value可使用中文，方便阅读。
[MobClickevent:@"友盟后台定义的事件ID"attributes:dict]; //此方法会在后台生成对应属性下的对应触发次数的统计。
```

###### 计算事件

此类事件包含计数事件的所有功能。使用情景是当出现每次的事件触发时会有不同的数值数据跟随产生，则可使用计算事件。如

```objective-c
[MobClickevent:@"pay"attributes:@{@"book" :@"Swift Fundamentals"} counter:110];//支付行为，商品参数，所花金额。就可使用计算事件。
```

##### 页面统计

使用方法很简单，只要在对应页面生命周期方法内调用统计方法即可：

```objective-c
- (void)viewWillAppear:(BOOL)animated
{
    [superviewWillAppear:animated];
    [MobClickbeginLogPageView:@"页面名称"];//(页面名称可为中文可为英文)
}
- (void)viewWillDisappear:(BOOL)animated
{
    [superviewWillDisappear:animated];
    [MobClickendLogPageView:@"页面名称"];
}
```

但问题来了，我们一个项目中少则十几二十个页面，多则上百甚至几百个页面，难道我们要挨个页面去将统计方法加到页面生命周期方法内吗？

NO，坚决是NO。重复写代码可不是一个程序员该干的事情。

这里的解决方法是利用runtime中的方法交换机制，来重写系统方法，让每个页面在调用时都会走我们重写过的方法，再执行页面中的原有方法。 我的解决步骤是：

首先，一般的项目都会写一个BaseViewController或者类似的基类，用于整合一些控制器可以用到的通用功能（如果您的所有控制器都继承的是UIViewController，那恭喜，您的统计数据将会非常庞大）。创建一个该基类的分类文件，并导入头文件`#import<objc/runtime.h>`。 
接下来，在分类文件的.m文件内，粘贴以下代码：

```objective-c
/**
 在NSObject的load方法中交换方法内容。
 */
+ (void)load{
    staticdispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
      [self method_exchange:@selector(viewWillAppear:)with:@selector(um_viewWillAppear:)]; 
      [self method_exchange:@selector(viewWillDisappear:)with:@selector(um_viewWillDisappear:)];		});
}

/**
 交换方法，将IMP部分交换
 @param oldMethod 旧方法
 @param newMethod 新方法
 */
+ (void)method_exchange:(SEL)oldMethod with:(SEL)newMethod{
    Class class = [selfclass];
    SEL originalSelector = oldMethod;
    SEL swizzledSelector = newMethod;

    Method originalMethod =class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod =class_getInstanceMethod(class, swizzledSelector);

    BOOL success =class_addMethod(class, originalSelector,method_getImplementation(swizzledMethod),method_getTypeEncoding(swizzledMethod));
    if (success) {
        class_replaceMethod(class, swizzledSelector,method_getImplementation(originalMethod),method_getTypeEncoding(originalMethod));
    } else {
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}

/**
 重写后的viewWillAppear方法
 */
- (void)um_viewWillAppear:(BOOL)animated
{
    //这里调用自身并不会产生循环调用的死循环，因为在调用时，这个方法已被替换成系统的viewWillAppear方法了。
    [selfum_viewWillAppear:animated];
    NSLog(@"\n—————————————进入页面: %@ ————————————",NSStringFromClass([self class]));
    [MobClickbeginLogPageView:NSStringFromClass([selfclass])];
}

/**
 重写后的viewWillDisappear方法
 */
-(void)um_viewWillDisappear:(BOOL)animated{
    [selfum_viewWillDisappear:animated];
    NSLog(@"\n————————————离开页面: %@ ————————————————",NSStringFromClass([self class]));
    [MobClickendLogPageView:NSStringFromClass([selfclass])];
}
```

因人而异，打印日志部分如果不想看效果可去除，其他部分可按需自行修改。

由于重写的是load方法，因此无需引用，系统在创建控制器之后自动会运行该方法。

编译，通过，运行，此时无论跳转到任何页面，都会有统计方法被触发了。

**修改：**

后来经过某位大神劈头盖脸一顿说，幡然醒悟。。。 是的，我们已经有了一个自定义的基类了，你还重写个毛线的系统方法啊。。。

因此，上述方法，仅实用与以系统类作为继承基类的项目（不过应该很少了）。

也就是说，由于系统类的实现方法我们无法进行修改，所以才需要使用runtime的交换机制来重写实现方法，但是，我们已经有自定义的基类了，直接在基类的实现方法里加入对应代码就好：

```objective-c
- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    [MobClick beginLogPageView:NSStringFromClass([self class])];
}

- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];
    [MobClick endLogPageView:NSStringFromClass([self class])];    
}
```

这样就可以了，无需使用runtime机制。经验证没有任何错误。

现实往往就是这么残酷，你想了很久才想到的方法，别人一句话就可以推翻。

其他功能有兴趣请自行查看[友盟官方文档](http://dev.umeng.com/analytics/ios-doc/integration)。


#### 坑

1. 事件统计方法传入参数前在封装字典的时候，一定要对参数进行是否为nil的防崩处理，自己写死的参数还好，如果是后台传来的参数，直接写入字典，程序会崩到你生活不能自理。
2. 友盟后台批量导入事件的时候事件ID一定不能有中文，否则事件不能成功导入，但如果单独创建事件事件ID可以带中文（不知道友盟抽的什么风）。
3. 在程序内部，事件ID尽量写成宏定义或者扩展字符串，以便如果哪天事件ID换了，只修改一处即可。
4. 在重写系统方法中一定要调用本方法，一定要，否则原生的系统方法无法执行。

就这样吧。