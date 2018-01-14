---
title: iOS后台倒计时
date: 2017-12-19 16:12
tags: iOS
categories: iOS Tips
---

### 场景

我们经常遇到这样的场景，比如电商类App到零点的时候开始抢购，比如商品限购倒计时等等。这种场景下需要我们将客户端的时间与服务器保持一致，最重要的是，要防止用户通过断网修改系统时间，来影响客户端的逻辑。下面是我个人的分析和实现步骤，只为了帮助有同样需求的人，知识有限，欢迎大神们补充。

如果不想看分析的同学，可以直接调到 “奉上代码” 处查看具体实现。

<!-- more -->

### 分析

研究之前，对京东做了一下抓包，数据如下

- 京东抓包秒杀数据

```objective-c
"miaoshaInfo": {
            "title": "京东秒杀",
            "miaoshaRemainTime": "79836",
            "miaosha": true
        },

```

数据中可以看出，京东秒杀商品返回了秒杀剩余的时间。通过进入后台，再次进入前台，以及断网修改时间的尝试，发现并不影响倒计时的运行。

- 仿“京东秒杀”实现思路
  1、程序进入后台时计时器不停止，这种做法网上有较多案例。例如：[iOS 后台完成倒计时的功能](https://www.jianshu.com/p/e2194f4b2734) 这种方案简书里面就有很多，有兴趣的同学可以在搜索一下。对于这种方案，我个人觉得有一定的审核风险，并没有使用。

  2、在程序进入后台和进入前台时分别记录时间，程序进入前台获得时间差 IntervalTime，然后在定时器响应的时候获得正确的剩余时间（miaoshaRemainTime - IntervalTime）。我用的是这个方案，不过这个方案有个缺点，用户通过修改时间影响倒计时。👇我们就研究一下，如果获取精准的后台停留时间。

### iOS关于时间的处理

- NSDate
  NSDate是我们平时使用较多的一个类，先看下它的定义：

```objective-c
NSDate objects encapsulate a single point in time,
independent of any particular calendrical system or time zone.
Date objects are immutable,
representing an invariant time interval relative to an absolute reference date (00:00:00 UTC on 1 January 2001).

```

NSDate对象描述的是时间线上的一个绝对的值，和时区和文化无关，它参考的值是：以UTC为标准的，2001年一月一日00：00：00这一刻的时间绝对值。我们用编程语言描述时间的时候，都是以一个时间线上的绝对值为参考点，参考点再加上偏移量（以秒或者毫秒，微秒，纳秒为单位）来描述另外的时间点。

获取时间的API

```objective-c
NSDate* date = [NSDate date];
NSLog(@"current date interval: %f", [date timeIntervalSinceReferenceDate]);

```

timeIntervalSinceReferenceDate返回的是距离参考时间的偏移量，这个偏移量的值为502945767秒，502945767/86400/365=15.9483056507，86400是一天所包含的秒数，365大致是一年的天数，15.94当然就是年数了，算出来刚好是此刻距离2001年的差值。

关于NSDate最重要的一点是：NSDate是受手机系统时间控制的。也就是说，当你修改了手机上的时间显示，NSDate获取当前时间的输出也会随之改变。在我们做App的时候，明白这一点，就知道NSDate并不可靠，因为用户可能会修改它的值。

- CFAbsoluteTimeGetCurrent()
  官方定义如下：

```objective-c
Absolute time is measured in seconds relative to the absolute reference date of Jan 1 2001 00:00:00 GMT. 
A positive value represents a date after the reference date, 
a negative value represents a date before it.
 For example, the absolute time -32940326 is equivalent to December 16th, 1999 at 17:54:34. 
Repeated calls to this function do not guarantee monotonically increasing results. 
The system time may decrease due to synchronization with external time references or due to an explicit user change of the clock.

```

从上面的描述不难看出CFAbsoluteTimeGetCurrent()的概念和NSDate非常相似，只不过参考点是：以GMT为标准的，2001年一月一日00：00：00这一刻的时间绝对值。

同样CFAbsoluteTimeGetCurrent()也会跟着当前设备的系统时间一起变化，也可能会被用户修改。

- gettimeofday
  这个API也能返回一个描述当前时间的值，代码如下：

```objective-c
struct timeval now;
struct timezone tz;
gettimeofday(&now, &tz);
NSLog(@"gettimeofday: %ld", now.tv_sec);

```

使用gettimeofday获得的值是Unix time。Unix time又是什么呢？

Unix time是以UTC 1970年1月1号 00：00：00为基准时间，当前时间距离基准点偏移的秒数。上述API返回的值是1481266031，表示当前时间距离UTC 1970年1月1号 00：00：00一共过了1481266031秒。

实际上NSDate也有一个API能返回Unix time：

```objective-c
NSDate* date = [NSDate date];
NSLog(@"timeIntervalSince1970: %f", [date timeIntervalSince1970]);

```

gettimeofday和NSDate，CFAbsoluteTimeGetCurrent()一样，都是受当前设备的系统时间影响。只不过是参考的时间基准点不一样而已。我们和服务器通讯的时候一般使用Unix time。

- mach_absolute_time()
  mach_absolute_time()可能用到的同学比较少，但这个概念非常重要。

  前面提到我们需要找到一个均匀变化的属性值来描述时间，而在我们的iPhone上刚好有一个这样的值存在，就是CPU的时钟周期数（ticks）。这个tick的数值可以用来描述时间，而mach_absolute_time()返回的就是CPU已经运行的tick的数量。将这个tick数经过一定的转换就可以变成秒数，或者纳秒数，这样就和时间直接关联了。

  不过这个tick数，在每次手机重启之后，会重新开始计数，而且iPhone锁屏进入休眠之后tick也会暂停计数。

  mach_absolute_time()不会受系统时间影响，只受设备重启和休眠行为影响。

- CACurrentMediaTime()
  CACurrentMediaTime()可能接触到的同学会多一些，先看下官方定义：

```objective-c
/* Returns the current CoreAnimation absolute time. This is the result of
 * calling mach_absolute_time () and converting the units to seconds. */
CFTimeInterval CACurrentMediaTime (void)

```

CACurrentMediaTime()就是将上面mach_absolute_time()的CPU tick数转化成秒数的结果。以下代码：

```objective-c
double mediaTime = CACurrentMediaTime();
NSLog(@"CACurrentMediaTime: %f", mediaTime);

```

返回的就是开机后设备一共运行了(设备休眠不统计在内)多少秒，另一个API也能返回相同的值：

```objective-c
NSTimeInterval systemUptime = [[NSProcessInfo processInfo] systemUptime];
NSLog(@"systemUptime: %f", systemUptime);

```

CACurrentMediaTime()也不会受系统时间影响，只受设备重启和休眠行为影响。

- sysctl
  iOS系统还记录了上次设备重启的时间。可以通过如下API调用获取：

```objective-c
#include <sys/sysctl.h>
-(long)bootTime
{
    int mib[2] = {CTL_KERN, KERN_BOOTTIME};
    size_t size;
    struct timeval  boottime;
    
    size = sizeof(boottime);
    if (sysctl(mib, MIB_SIZE, &boottime, &size, NULL, 0) != -1)
    {
        return boottime.tv_sec;
    }
    return 0;
}

```

返回的值是上次设备重启的Unix time。
这个API返回的值也会受系统时间影响，用户如果修改时间，值也会随着变化。

到此处，估计有的同学已经想到了方案，我们可以通过获取系统运行的时间，来获得进入后台的时间差。

```objective-c
//系统当前运行了多长时间
+(NSTimeInterval)uptimeSinceLastBoot
{
    //获取当前设备时间时间戳 受用户修改时间影响
    struct timeval now;
    struct timezone tz;
    gettimeofday(&now, &tz);
//    NSLog(@"gettimeofday: %ld", now.tv_sec);
 
    //获取系统上次重启的时间戳 受用户修改时间影响
    struct timeval boottime;
    int mib[2] = {CTL_KERN, KERN_BOOTTIME};
    size_t size = sizeof(boottime);
    
    double uptime = -1;
    
    if (sysctl(mib, 2, &boottime, &size, NULL, 0) != -1 && boottime.tv_sec != 0)
    {
        //因为两个参数都会受用户修改时间的影响，因此它们想减的值是不变的
        uptime = now.tv_sec - boottime.tv_sec;
        uptime += (double)(now.tv_usec - boottime.tv_usec) / 1000000.0;
    }
    return uptime;
}

```

gettimeofday和sysctl都会受系统时间影响，但他们二者做一个减法所得的值，就和系统时间无关了。这样就可以避免用户修改时间影响我们。

### 奉上代码

##### 倒计时

```objective-c
@interface QGCountDown ()
@property(nonatomic,retain) dispatch_source_t timer;

@end


@implementation QGCountDown

/**
 *  每秒回调一次
 */
-(void)countDownWithPER_SECBlock:(void (^)())PER_SECBlock
{
    if (_timer==nil) {
        dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
        _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0,queue);
        dispatch_source_set_timer(_timer,dispatch_walltime(NULL, 0),1.0*NSEC_PER_SEC, 0); //每秒执行
        dispatch_source_set_event_handler(_timer, ^{
            dispatch_async(dispatch_get_main_queue(), ^{
                PER_SECBlock();
            });
        });
        dispatch_resume(_timer);
    }
}
/**
 *  主动销毁定时器
 *  @return 格式为年-月-日
 */
-(void)destoryTimer
{
    if (_timer) {
        dispatch_source_cancel(_timer);
        _timer = nil;
    }
}
@end

```

##### 通知监听

```objective-c
/**
 *  添加通知
 */
- (void)addNotification
{
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(applicationWillResignActive:)
                                                 name:UIApplicationWillResignActiveNotification object:nil]; //监听是否触发home键挂起程序.
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(applicationDidBecomeActive:)
                                                 name:UIApplicationDidBecomeActiveNotification object:nil];
}

/**
 *  移除通知
 */
- (void)removeNotification
{
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIApplicationWillResignActiveNotification object:nil];
    [[NSNotificationCenter defaultCenter] removeObserver:self name:UIApplicationWillResignActiveNotification object:nil];
}



- (void)applicationWillResignActive:(NSNotification *)notification
{
    self.systemUpTime = [NSString uptimeSinceLastBoot];
}

- (void)applicationDidBecomeActive:(NSNotification *)notification
{
    NSTimeInterval currentupTime = [NSString uptimeSinceLastBoot];
    NSTimeInterval intervalTime = currentupTime - self.systemUpTime;
    if (intervalTime > 0) {
        self.intervalTime = intervalTime;
    }
}

```

##### 获取系统当前运行了多长时间

```objective-c
//系统当前运行了多长时间
+(NSTimeInterval)uptimeSinceLastBoot
{
  //获取当前设备时间时间戳 受用户修改时间影响
  struct timeval now;
  struct timezone tz;
  gettimeofday(&now, &tz);
//    NSLog(@"gettimeofday: %ld", now.tv_sec);

  //获取系统上次重启的时间戳 受用户修改时间影响
  struct timeval boottime;
  int mib[2] = {CTL_KERN, KERN_BOOTTIME};
  size_t size = sizeof(boottime);

  double uptime = -1;

  if (sysctl(mib, 2, &boottime, &size, NULL, 0) != -1 && boottime.tv_sec != 0)
  {
      //因为两个参数都会受用户修改时间的影响，因此它们想减的值是不变的
      uptime = now.tv_sec - boottime.tv_sec;
      uptime += (double)(now.tv_usec - boottime.tv_usec) / 1000000.0;
  }
  return uptime;
}

```

##### 秒杀逻辑

```objective-c
//秒杀配置项
- (void)countDownConfig
{
    if (self.detailModel.isMiaoSha) {
        if (self.countDown) {
            [self.countDown destoryTimer];
        }
        self.countDown = [[QGCountDown alloc]init];
        //后台传过来的剩余时间
        __block long remainTime = self.detailModel.countdown.remainTime.longLongValue;
        __weak typeof(self) weakSelf = self;
        //每秒回调一次
        [self.countDown countDownWithPER_SECBlock:^{
        
            remainTime = remainTime - 1;
            if (weakSelf.intervalTime > 0) {
                remainTime = remainTime - weakSelf.intervalTime;
                weakSelf.intervalTime = 0;
            }
            [weakSelf updateDataInVisibleCellWithRemainTime:remainTime];
            
        }];

    }
}

//刷新秒杀倒计时
- (void)updateDataInVisibleCellWithRemainTime:(long)timeInterval
{
    if (timeInterval == 0) {
        //废弃倒计时，清空本地秒杀数据，重新请求
        [self.countDown destoryTimer];
        self.detailModel.countdown = nil;
        [self.tableView reloadData];
        [self getProductDetail];
    }
    
    int days = (int)(timeInterval/(3600*24));
    int hours = (int)((timeInterval-days*24*3600)/3600);
    int minutes = (int)(timeInterval-days*24*3600-hours*3600)/60;
    int seconds = (int)(timeInterval-days*24*3600-hours*3600-minutes*60);
    NSString *hoursStr;NSString *minutesStr;NSString *secondsStr;
    
    if (hours < 10)
        //小时
        hoursStr = [NSString stringWithFormat:@"0%d",hours];
    else
        //小时
        hoursStr = [NSString stringWithFormat:@"%d",hours];
    
    //分钟
    if(minutes<10)
        minutesStr = [NSString stringWithFormat:@"0%d",minutes];
    else
        minutesStr = [NSString stringWithFormat:@"%d",minutes];
    //秒
    if(seconds < 10)
        secondsStr = [NSString stringWithFormat:@"0%d", seconds];
    else
        secondsStr = [NSString stringWithFormat:@"%d",seconds];
    //此处只刷新可见的Cell
    NSArray * cellArray = self.tableView.visibleCells;
    for (UITableViewCell * cell in cellArray) {
        if ([cell isKindOfClass:[你的秒杀倒计时cell]]) {
           //刷新
        }
    }
    
}
```