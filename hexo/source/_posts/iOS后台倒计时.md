---
title: iOS后台倒计时
date: 2017-12-18
tags: iOS
categories: iOS Tips
---

```objective-c

#import <Foundation/Foundation.h>
/** 进入后台block typedef */
/** 处理进入后台并计算留在后台时间间隔 */
typedef void (^YQHandlerEnterBackgroundBlock)(NSNotification *notify, NSTimeInterval stayBackgroundTime);

@interface Commons : NSObject
/** 添加观察者并处理后台 */
+ (void)addObserverUsingBlock:(YQHandlerEnterBackgroundBlock)block;
/** 移除后台观察者 */
+ (void)removeNotificationObserver:(nullable id)observer;
@end
```
<!-- more -->
```objective-c
#import "Commons.h"

@implementation Commons
  
//在block里面处理时间间隔，将倒计时的时间减去进入后台的时间，重新赋值给倒计时时间显示
+ (void)addObserverUsingBlock:(YQHandlerEnterBackgroundBlock)block {
    __block CFAbsoluteTime enterBackgroundTime;
    [[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationDidEnterBackgroundNotification object:nil queue:nil usingBlock:^(NSNotification * _Nonnull note) {
        if ([note.object isKindOfClass:[UIApplication class]]) {
            enterBackgroundTime = CFAbsoluteTimeGetCurrent();
        }
    }];
    __block CFAbsoluteTime enterForegroundTime;
    [[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationWillEnterForegroundNotification object:nil queue:nil usingBlock:^(NSNotification * _Nonnull note) {
        if ([note.object isKindOfClass:[UIApplication class]]) {
            enterForegroundTime = CFAbsoluteTimeGetCurrent();
            CFAbsoluteTime timeInterval = enterForegroundTime - enterBackgroundTime;
            block ? block(note, timeInterval): nil;
        }
    }];
}

+ (void)removeNotificationObserver:(id)observer {
    if (!observer) {
        return;
    }
    [[NSNotificationCenter defaultCenter]removeObserver:observer name:UIApplicationDidEnterBackgroundNotification object:nil];
    [[NSNotificationCenter defaultCenter]removeObserver:observer name:UIApplicationWillEnterForegroundNotification object:nil];
}
@end
```

```objective-c
// 开启倒计时效果
-(void)openCountdown{

    __block NSInteger time = 59; //倒计时时间
    
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_source_t _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
    
    dispatch_source_set_timer(_timer,dispatch_walltime(NULL, 0),1.0*NSEC_PER_SEC, 0); //每秒执行
    
    dispatch_source_set_event_handler(_timer, ^{
        
        if(time <= 0){ //倒计时结束，关闭
            
            dispatch_source_cancel(_timer);
            dispatch_async(dispatch_get_main_queue(), ^{

                //设置按钮的样式
                [self.authCodeBtn setTitle:@"重新发送" forState:UIControlStateNormal];
                [self.authCodeBtn setTitleColor:[UIColor colorFromHexCode:@"FB8557"] forState:UIControlStateNormal];
                self.authCodeBtn.userInteractionEnabled = YES;
            });

        }else{
            
            int seconds = time % 60;
            dispatch_async(dispatch_get_main_queue(), ^{

                //设置按钮显示读秒效果
                [self.authCodeBtn setTitle:[NSString stringWithFormat:@"重新发送(%.2d)", seconds] forState:UIControlStateNormal];
                [self.authCodeBtn setTitleColor:[UIColor colorFromHexCode:@"979797"] forState:UIControlStateNormal];
                self.authCodeBtn.userInteractionEnabled = NO;
            });
            time--;
        }
    });
    dispatch_resume(_timer);
}

```

#### 