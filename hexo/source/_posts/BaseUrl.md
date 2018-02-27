#### 问题描述

URLWithString:(NSString *)str relativeToURL:(NSURL *)baseURL中baseURL结尾字段的相关问题拼接后被去掉的问题，情况如下图：

```
NSURL *url = [NSURL URLWithString:@"http://example.com/v1"];
NSLog(@"%@",url);
NSURL *newURL = [NSURL URLWithString:@"foo?bar=baz" relativeToURL:url];
NSLog(@"newURL:%@",[newURL absoluteString]);

```

![img](http://upload-images.jianshu.io/upload_images/3327593-ef971bfa8f031577.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

其中“v1”拼接后就被去掉了。

#### 解决方法一

在baseURL后拼接一个空字符串，即可解决这个问题，如果baseURL本身结尾带`／`，则空字符串不产生作用；如果baseURL结尾没有`／`，则拼接完成后会在baseURL上加入`／`。
如果这时再执行URLWithStringrelativeTOURL：就不会去掉任何字段了

```
url = [url URLByAppendingPathComponent:@""];
NSLog(@"****url:%@",[url absoluteString]);
newURL = [NSURL URLWithString:@"foo?bar=baz" relativeToURL:url];
NSLog(@"====newURL:%@",[newURL absoluteString]);

```

![img](http://upload-images.jianshu.io/upload_images/3327593-c4a16665ef4ad0e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

####  解决方法二

baseURL一定要以`／`结尾