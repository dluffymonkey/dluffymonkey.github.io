---
title: iOS崩溃日志收集上传
date: 2017-12-05
tags: iOS
categories: iOS Tips
---

需要保存到日志里面的信息有：手机号  设备标识UUID  设备类型  网络类型  是否越狱 版本号 操作系统版本   时间   崩溃原因

```objective-c
#import <Foundation/Foundation.h>

/**
 崩溃日志收集
 */
@interface LQUncaughtExceptionHandler : NSObject

+ (void)setDefaultHandler;

+ (NSUncaughtExceptionHandler *)getHandler;

+ (void)takeException:(NSException *)exception;

@end
```
<!--more -->

```objective-c
#import "LQUncaughtExceptionHandler.h"
#import "LoginInfoManager.h"
#import "DeviceInfo.h"

/**
 沙盒地址

 @return 返回沙盒地址字符串
 */
NSString *applicationDocumentsDirectory() {

    return  [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
}

void UncaughtExceptionHandler(NSException *exception) {
    NSDate   *date = [NSDate date];
	NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
	[formatter setDateStyle:NSDateFormatterMediumStyle];
	[formatter setTimeStyle:NSDateFormatterShortStyle];
	[formatter setDateFormat:@"YYYY-MM-dd hh:mm:ss"];

	NSString *mobile = [LoginInfoManager sharedInstance].user.mobile;
	NSString *uuid = [DeviceInfo getDeviceNumber];
	NSString *deviceString = [DeviceInfo deviceString];
	NSString *network = [DeviceInfo getNetWork];
	NSString *prisonBreak = [DeviceInfo getSystemStatus];
	NSString *version = APP_VERSION;
	NSString *phoneVersion = [[UIDevice currentDevice] systemVersion];
	NSString *dateTime = [formatter stringFromDate:date];
	NSString *name = [exception name];
	NSString *reason = [exception reason];   // 崩溃的原因  可以有崩溃的原因(数组越界,字典nil,调用未知方法...) 崩溃的控制器以及方法
	NSArray  *callStackSymbolsArray = [exception callStackSymbols];

	NSString *logInfo = [NSString stringWithFormat:@"========异常错误报告========\nmobile:%@\nuuid:%@\ndeviceString:%@\nnet:%@\nprisonBreak:%@\nversion:%@\ndateTime:%@\nphoneVersion:%@\nname:%@\nreason:\n%@\ncallStackSymbols:\n%@",mobile,uuid,deviceString,network,prisonBreak,version,phoneVersion,dateTime,name,reason,[callStackSymbolsArray componentsJoinedByString:@"\n"]];
	NSString *path = [applicationDocumentsDirectory() stringByAppendingPathComponent:@"ExceptionLog.txt"];
	[logInfo writeToFile:path atomically:YES encoding:NSUTF8StringEncoding error:nil];
}
```

```objective-c
@implementation LQUncaughtExceptionHandler

// 沙盒地址
- (NSString *)applicationDocumentsDirectory {
	return [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
}

+ (void)setDefaultHandler {
  NSSetUncaughtExceptionHandler(&UncaughtExceptionHandler);
}

+ (NSUncaughtExceptionHandler *)getHandler {
	return NSGetUncaughtExceptionHandler();
}

+ (void)takeException:(NSException *)exception {
  	NSDate   *date = [NSDate date];
    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
    [formatter setDateStyle:NSDateFormatterMediumStyle];
    [formatter setTimeStyle:NSDateFormatterShortStyle];
    [formatter setDateFormat:@"YYYY-MM-dd hh:mm:ss"];
    
    NSString *mobile = [LoginInfoManager sharedInstance].user.mobile;
    NSString *uuid = [DeviceInfo getDeviceNumber];
    NSString *deviceString = [DeviceInfo deviceString];
    NSString *network = [DeviceInfo getNetWork];
    NSString *prisonBreak = [DeviceInfo getSystemStatus];
    NSString *version = APP_VERSION;
    NSString *phoneVersion = [[UIDevice currentDevice] systemVersion];
    NSString *reason = [exception reason];   // 崩溃的原因  可以有崩溃的原因(数组越界,字典nil,调用未知方法...) 崩溃的控制器以及方法
    NSString *name = [exception name];
    NSString *dateTime = [formatter stringFromDate:date];
    NSArray  *callStackSymbolsArray = [exception callStackSymbols];
    
    NSString *logInfo = [NSString stringWithFormat:@"========异常错误报告========\nmobile:%@\nuuid:%@\ndeviceString:%@\nnet:%@\nprisonBreak:%@\nversion:%@\ndateTime:%@\nphoneVersion:%@\nname:%@\nreason:\n%@\ncallStackSymbols:\n%@",mobile,uuid,deviceString,network,prisonBreak,version,dateTime,phoneVersion,name,reason,[callStackSymbolsArray componentsJoinedByString:@"\n"]];
    NSString *path = [applicationDocumentsDirectory() stringByAppendingPathComponent:@"ExceptionLog.txt"];
    [logInfo writeToFile:path atomically:YES encoding:NSUTF8StringEncoding error:nil];
}

@end
```



```objective-c
#import <Foundation/Foundation.h>
#import <SystemConfiguration/CaptiveNetwork.h>  //  获取wifi
#import <CoreMotion/CoreMotion.h>  //  加速计
#import <ifaddrs.h>    //获取Ip
#import <arpa/inet.h>
#import <net/if.h>
#define IOS_CELLULAR    @"pdp_ip0"
#define IOS_WIFI        @"en0"
#define IOS_VPN         @"utun0"
#define IP_ADDR_IPv4    @"ipv4"
#define IP_ADDR_IPv6    @"ipv6"

@interface DeviceInfo : NSObject
#pragma mark - 获取设备当前网络IP地址
+ (NSString *)getIPAddress:(BOOL)preferIPv4;
+ (NSString *)getWifiName;

+ (NSString *)getDeviceNumber;
+ (NSString *)getSystemStatus;  // 0 代表没有越狱。 1 代表越狱
+ (NSString *)getSystemVersion;
+ (NSString *)deviceString;
+ (NSString *)getNetWork;
@end
```

```objective-c
# import "DeviceInfo.h"
# import "sys/utsname.h"
# import <CoreTelephony/CoreTelephonyDefines.h>
# import <CoreTelephony/CTTelephonyNetworkInfo.h>
# import <AFNetworkReachabilityManager.h> //有用到AFNetworking

# define ARRAY_SIZE(a) sizeof(a)/sizeof(a[0])    //获取是否越狱

const char* jailbreak_tool_pathes[] = {
	"/Applications/Cydia.app",
	"/Library/MobileSubstrate/MobileSubstrate.dylib",
	"/bin/bash",
	"/usr/sbin/sshd",
	"/etc/apt"
};
```

```objective-c
@implementation DeviceInfo

+ (NSString *)getIPAddress:(BOOL)preferIPv4
{
    NSArray *searchArray = preferIPv4 ?
    @[ IOS_VPN @"/" IP_ADDR_IPv4, IOS_VPN @"/" IP_ADDR_IPv6, IOS_WIFI @"/" IP_ADDR_IPv4, IOS_WIFI @"/" IP_ADDR_IPv6, IOS_CELLULAR @"/" IP_ADDR_IPv4, IOS_CELLULAR @"/" IP_ADDR_IPv6 ] :
    @[ IOS_VPN @"/" IP_ADDR_IPv6, IOS_VPN @"/" IP_ADDR_IPv4, IOS_WIFI @"/" IP_ADDR_IPv6, IOS_WIFI @"/" IP_ADDR_IPv4, IOS_CELLULAR @"/" IP_ADDR_IPv6, IOS_CELLULAR @"/" IP_ADDR_IPv4 ] ;
    
    NSDictionary *addresses = [self getIPAddresses];
    
    
    __block NSString *address;
    [searchArray enumerateObjectsUsingBlock:^(NSString *key, NSUInteger idx, BOOL *stop)
     {
         address = addresses[key];
         //筛选出IP地址格式
         if([self isValidatIP:address]) *stop = YES;
     } ];
    return address ? address : @"0.0.0.0";
}

+ (BOOL)isValidatIP:(NSString *)ipAddress {
    if (ipAddress.length == 0) {
        return NO;
    }
    NSString *urlRegEx = @"^([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\."
    "([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\."
    "([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\."
    "([01]?\\d\\d?|2[0-4]\\d|25[0-5])$";
    
    NSError *error;
    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:urlRegEx options:0 error:&error];
    
    if (regex != nil) {
        NSTextCheckingResult *firstMatch=[regex firstMatchInString:ipAddress options:0 range:NSMakeRange(0, [ipAddress length])];
        
        if (firstMatch) {
            NSRange resultRange = [firstMatch rangeAtIndex:0];
            NSString *result=[ipAddress substringWithRange:resultRange];
            //输出结果
            NSLog(@"%@",result);
            return YES;
        }
    }
    return NO;
}

+ (NSDictionary *)getIPAddresses
{
    NSMutableDictionary *addresses = [NSMutableDictionary dictionaryWithCapacity:8];
    
    // retrieve the current interfaces - returns 0 on success
    struct ifaddrs *interfaces;
    if(!getifaddrs(&interfaces)) {
        // Loop through linked list of interfaces
        struct ifaddrs *interface;
        for(interface=interfaces; interface; interface=interface->ifa_next) {
            if(!(interface->ifa_flags & IFF_UP) /* || (interface->ifa_flags & IFF_LOOPBACK) */ ) {
                continue; // deeply nested code harder to read
            }
            const struct sockaddr_in *addr = (const struct sockaddr_in*)interface->ifa_addr;
            char addrBuf[ MAX(INET_ADDRSTRLEN, INET6_ADDRSTRLEN) ];
            if(addr && (addr->sin_family==AF_INET || addr->sin_family==AF_INET6)) {
                NSString *name = [NSString stringWithUTF8String:interface->ifa_name];
                NSString *type;
                if(addr->sin_family == AF_INET) {
                    if(inet_ntop(AF_INET, &addr->sin_addr, addrBuf, INET_ADDRSTRLEN)) {
                        type = IP_ADDR_IPv4;
                    }
                } else {
                    const struct sockaddr_in6 *addr6 = (const struct sockaddr_in6*)interface->ifa_addr;
                    if(inet_ntop(AF_INET6, &addr6->sin6_addr, addrBuf, INET6_ADDRSTRLEN)) {
                        type = IP_ADDR_IPv6;
                    }
                }
                if(type) {
                    NSString *key = [NSString stringWithFormat:@"%@/%@", name, type];
                    addresses[key] = [NSString stringWithUTF8String:addrBuf];
                }
            }
        }
        // Free memory
        freeifaddrs(interfaces);
    }
    return [addresses count] ? addresses : nil;
}


+ (NSString *)getWifiName
{
    NSString *wifiName = @"";
    
    CFArrayRef wifiInterfaces = CNCopySupportedInterfaces();
    
    if (!wifiInterfaces) {
        return @"";
    }
    
    NSArray *interfaces = (__bridge NSArray *)wifiInterfaces;
    
    for (NSString *interfaceName in interfaces) {
        CFDictionaryRef dictRef = CNCopyCurrentNetworkInfo((__bridge CFStringRef)(interfaceName));
        
        if (dictRef) {
            NSDictionary *networkInfo = (__bridge NSDictionary *)dictRef;
            NSLog(@"network info -> %@", networkInfo);
            wifiName = [networkInfo objectForKey:(__bridge NSString *)kCNNetworkInfoKeySSID];
            
            CFRelease(dictRef);
        }
    }
    
    CFRelease(wifiInterfaces);
    return wifiName;
}

+ (NSString *)getDeviceNumber{
    NSString *deviceNumber = [self uuid];  //自己项目里面获取UUID的方法
    return deviceNumber;
}

+ (NSString *)getSystemVersion{
    NSString *systemVersion = [[UIDevice currentDevice] systemVersion];
    return systemVersion;
}

+ (NSString *)getSystemStatus{
    BOOL isJailBreak = [self checkIsJailBreak];
    NSString *systemStatus = @"";
    if (isJailBreak) {
        systemStatus = @"1";
    }else{
        systemStatus = @"0";
    }
    return systemStatus;
}

+ (BOOL)checkIsJailBreak{
    for (int i=0; i<ARRAY_SIZE(jailbreak_tool_pathes); i++) {
        if ([[NSFileManager defaultManager] fileExistsAtPath:[NSString stringWithUTF8String:jailbreak_tool_pathes[i]]]) {
            NSLog(@"The device is jail broken!");
            return YES;
        }
    }
    NSLog(@"The device is NOT jail broken!");
    return NO;
}

+ (NSString *)deviceString {
    struct utsname systemInfo;
    uname(&systemInfo);
    return [NSString stringWithCString:systemInfo.machine encoding:NSUTF8StringEncoding];
}

+ (NSString *)getNetWork {
    
    UIApplication *app = [UIApplication sharedApplication];
    NSArray *children;
    if ([[app valueForKeyPath:@"_statusBar"] isKindOfClass:NSClassFromString(@"UIStatusBar_Modern")]) {
        return [self getIphoneXNetInfo];
    } else {
        children = [[[app valueForKeyPath:@"_statusBar"] valueForKeyPath:@"foregroundView"] subviews];
    }
    int type = 0;
    for (id child in children) {
        if ([child isKindOfClass:[NSClassFromString(@"UIStatusBarDataNetworkItemView") class]]) {
            type = [[child valueForKeyPath:@"dataNetworkType"] intValue];
        }
    }

    NSString *stateString = @"wifi";
    switch (type) {
        case 0:
            stateString = @"";
            break;

        case 1:
            stateString = @"2G";
            break;

        case 2:
            stateString = @"3G";
            break;

        case 3:
            stateString = @"4G";
            break;

        case 4:
            stateString = @"LTE";
            break;

        case 5:
            stateString = @"wifi";
            break;

        default:
            break;
    }

    return stateString;
}

+ (NSString *)getIphoneXNetInfo {
    AFNetworkReachabilityStatus status = [AFNetworkReachabilityManager sharedManager].networkReachabilityStatus;
    if(status == AFNetworkReachabilityStatusReachableViaWiFi) {
        return @"wifi";
    }else if(status == AFNetworkReachabilityStatusNotReachable) {
        return @"";
    }else {
        NSArray *typeStrings2G = @[CTRadioAccessTechnologyEdge,
                                   CTRadioAccessTechnologyGPRS,
                                   CTRadioAccessTechnologyCDMA1x];
        NSArray *typeStrings3G = @[CTRadioAccessTechnologyHSDPA,
                                   CTRadioAccessTechnologyWCDMA,
                                   CTRadioAccessTechnologyHSUPA,
                                   CTRadioAccessTechnologyCDMAEVDORev0,
                                   CTRadioAccessTechnologyCDMAEVDORevA,
                                   CTRadioAccessTechnologyCDMAEVDORevB,
                                   CTRadioAccessTechnologyeHRPD];
        
        NSArray *typeStrings4G = @[CTRadioAccessTechnologyLTE];
        // 该 API 在 iOS7 以上系统才有效
        if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 7.0) {
            CTTelephonyNetworkInfo *teleInfo= [[CTTelephonyNetworkInfo alloc] init];
            NSString *accessString = teleInfo.currentRadioAccessTechnology;
            if ([typeStrings4G containsObject:accessString]) {
                return @"4G";
            } else if ([typeStrings3G containsObject:accessString]) {
                return @"3G";
            } else if ([typeStrings2G containsObject:accessString]) {
                return @"2G";
            } else {
                return @"";
            }
        } else {
            return @"";
        }
    }
}

@end
```



最后一步，在项目的APPDelegate里面进行调用一下就可以了~

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
   
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    self.window.backgroundColor = [UIColor whiteColor];
    [self.window makeKeyAndVisible];
    //收集崩溃日志
    [self uploadExceptionLog];
    return YES;
}
```



```objective-c
- (void)uploadExceptionLog {
    [YQUncaughtExceptionHandler setDefaultHandler];
    // 发送崩溃日志
    NSString *path = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
    NSString *dataPath = [path stringByAppendingPathComponent:@"ExceptionLog.txt"];
    NSData *data = [NSData dataWithContentsOfFile:dataPath];
    if (data != nil) {
        [self sendExceptionLogWithData:data path:dataPath];
    }
}
```

 到这就ok了，祝工作愉快！