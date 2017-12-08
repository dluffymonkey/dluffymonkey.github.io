---
title: GCD学习 —— 二
date: 2017-12-07 19:42
tags: iOS
categories: iOS Tips
---

今天继续深入了解一下GCD。

####   Dispatch Source

​	使用      `Dispatch Source `而不使用 `dispatch_async` 的唯一原因就是利用联结的优势。

​	联结的大致流程：在任一线程上调用它的一个函数` dispatch_source_merge_data` 后，会执行` Dispatch Source` 事先定义好的句柄（可以把句柄简单理解为一个 block ）。这个过程叫 Custom event ，用户事件。是 `dispatch source` 支持处理的一种事件。简单地说，这种事件是由你调用 `dispatch_source_merge_data` 函数来向自己发出的信号。

##### 1 创建dispatch源

```objective-c
dispatch_source_t source = dispatch_source_create(dispatch_source_type_t type, uintptr_t handle, unsigned long mask, dispatch_queue_t queue)
```

| 参数     | 意义                              |
| ------ | ------------------------------- |
| type   | dispatch源可处理的事件                 |
| handle | 可以理解为句柄、索引或id，假如要监听进程，需要传入进程的ID |
| mask   | 可以理解为描述，提供更详细的描述，让它知道具体要监听什么    |
| queue  | 自定义源需要的一个队列，用来处理所有的响应句柄（block）  |
<!-- more -->
##### 2 **Dispatch Source可处理的所有事件**

| 名称                             | 内容                                 |
| ------------------------------ | ---------------------------------- |
| DISPATCH_SOURCE_TYPE_DATA_ADD  | 自定义的事件，变量增加                        |
| DISPATCH_SOURCE_TYPE_DATA_OR   | 自定义的事件，变量OR                        |
| DISPATCH_SOURCE_TYPE_MACH_SEND | MACH端口发送                           |
| DISPATCH_SOURCE_TYPE_MACH_RECV | MACH端口接收                           |
| DISPATCH_SOURCE_TYPE_PROC      | 进程监听,如进程的退出、创建一个或更多的子线程、进程收到UNIX信号 |
| DISPATCH_SOURCE_TYPE_READ      | IO操作，如对文件的操作、socket操作的读响应          |
| DISPATCH_SOURCE_TYPE_SIGNAL    | 接收到UNIX信号时响应                       |
| DISPATCH_SOURCE_TYPE_TIMER     | 定时器                                |
| DISPATCH_SOURCE_TYPE_VNODE     | 文件状态监听，文件被删除、移动、重命名                |
| DISPATCH_SOURCE_TYPE_WRITE     | IO操作，如对文件的操作、socket操作的写响应          |

**注意：**

- `DISPATCH_SOURCE_TYPE_DATA_ADD`当同一时间，一个事件的的触发频率很高，那么Dispatch Source会将这些响应以ADD的方式进行累积，然后等系统空闲时最终处理，如果触发频率比较零散，那么Dispatch Source会将这些事件分别响应。
- `DISPATCH_SOURCE_TYPE_DATA_OR` 和上面的一样，是自定义的事件，但是它是以OR的方式进行累积

##### 3 一些函数

```objective-c
dispatch_suspend(queue)  //挂起队列

dispatch_resume(source)  //分派源创建时默认处于暂停状态，在分派源分派处理程序之前必须先恢复

//向分派源发送事件，需要注意的是，不可以传递0值(事件不会被触发)，同样也不可以传递负数。
dispatch_source_merge_data(dispatch_source_t  _Nonnull source, unsigned long value)

//设置响应分派源事件的block，在分派源指定的队列上运行
dispatch_source_set_event_handler(dispatch_source_t  _Nonnull source, ^(void)handler) 

dispatch_source_get_data(dispatch_source_t  _Nonnull source) //得到分派源的数据

//得到dispatch源创建，即调用dispatch_source_create的第二个参数
uintptr_t dispatch_source_get_handle(dispatch_source_t source); 

//得到dispatch源创建，即调用dispatch_source_create的第三个参数
unsigned long dispatch_source_get_mask(dispatch_source_t source); 

//取消dispatch源的事件处理--即不再调用block。如果调用dispatch_suspend只是暂停dispatch源。
void dispatch_source_cancel(dispatch_source_t source); 

//检测是否dispatch源被取消，如果返回非0值则表明dispatch源已经被取消
long dispatch_source_testcancel(dispatch_source_t source); 

//dispatch源取消时调用的block，一般用于关闭文件或socket等，释放相关资源
void dispatch_source_set_cancel_handler(dispatch_source_t source, dispatch_block_t cancel_handler); 

//可用于设置dispatch源启动时调用block，调用完成后即释放这个block。也可在dispatch源运行当中随时调用这个函数。
void dispatch_source_set_registration_handler(dispatch_source_t source, dispatch_block_t registration_handler); 
```

##### 4 dispatch_source的基本用法

```objective-c
dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0, dispatch_get_global_queue(0, 0));
  dispatch_source_set_event_handler(source, ^{
    dispatch_sync(dispatch_get_main_queue(), ^{
        //更新UI
        NSLog(@"更新UI");
    });
});
dispatch_resume(source);
dispatch_async(dispatch_get_global_queue(0, 0), ^{

    //网络请求
    NSLog(@"更新UI");
    dispatch_source_merge_data(source, 1); //通知队列
    [NSThread sleepForTimeInterval:0.01];
});
```
上面的例子创建一个source，source的type为ADD的方式，然后将事件触发后要执行的句柄添加到main队列里，在source创建后默认是挂起的，需要用`dispatch_resume`函数来恢复监听，在执行了`dispatch_source_merge_data`后一定要执行`[NSThread sleepForTimeInterval:0.01];`否则上面的句柄操作有可能不执行，后面为了测试监听，加入了一个for循环，用`dispatch_source_merge_data`来触发事件，但是在触发事件的响应句柄里我们只打印了一次，结果是每次相加的和，也就是10，而不是打印了4次。

**原因：**`DISPATCH_SOURCE_TYPE_DATA_ADD`是将所有触发结果相加，最后统一执行响应，但是加入`sleepForTimeInterval`后，如果interval的时间越长，则每次触发都会响应，但是如果interval的时间很短，则会将触发后的结果相加后统一触发。

这在更新UI时很有用，比如更新进度条时，没必要每次触发都响应，因为更新时还有其他的用户操作（用户输入，触碰等），所以可以统一触发

```objective-c
//创建source，以DISPATCH_SOURCE_TYPE_DATA_ADD的方式进行累加，而DISPATCH_SOURCE_TYPE_DATA_OR是对结果进行二进制或运算
dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0, 		dispatch_get_main_queue());
//事件触发后执行的句柄
dispatch_source_set_event_handler(source,^{
    NSLog(@"监听函数：%lu",dispatch_source_get_data(source));
});
//开启source
dispatch_resume(source);
dispatch_queue_t myqueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);  
dispatch_async(myqueue, ^ {
  for(int i = 1; i <= 4; i ++){
     NSLog(@"~~~~~~~~~~~~~~%d", i);
     //触发事件，向source发送事件，这里i不能为0，否则触发不了事件
     dispatch_source_merge_data(source,i);
     //当Interval的事件越长，则每次的句柄都会触发
     [NSThread sleepForTimeInterval:0.01];
  }
});
```
##### 5  使用timer定时器

```objective-c
dispatch_source_set_timer(dispatch_source_t source, dispatch_time_t start, uint64_t interval, uint64_t leeway)
```

- source 分派源
- start 数控制计时器第一次触发的时刻。参数类型是 `dispatch_time_t`，这是一个opaque类型，我们不能直接操作它。我们得需要 `dispatch_time` 和  `dispatch_walltime` 函数来创建它们。另外，常量  `DISPATCH_TIME_NOW` 和 `DISPATCH_TIME_FOREVER` 通常很有用。
- interval 间隔时间
- leeway 计时器触发的精准程度

实例：

```objective-c
//倒计时时间
__block int timeout = 3;
//创建队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
//创建timer
dispatch_source_t _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, 		queue);
//设置2s触发一次，0s的误差，2s执行
dispatch_source_set_timer(_timer,dispatch_walltime(NULL, 0),2.0 * NSEC_PER_SEC, 0); 
//触发的事件
dispatch_source_set_event_handler(_timer, ^{
    if(timeout <= 0){ 
        //倒计时结束，取消dispatch源
        dispatch_source_cancel(_timer);
    } else {
        timeout--;
        dispatch_async(dispatch_get_main_queue(), ^{
            //更新主界面的操作
            NSLog(@"倒计时:%d", timeout);
        });
    }
});
//开始执行dispatch源
dispatch_resume(_timer);
```

##### 6 dispatch_suspend挂起队列

实例：

```objective-c
//创建DISPATCH_QUEUE_SERIAL队列
dispatch_queue_t queue1 = dispatch_queue_create("com.itachi.queue1", 0);
dispatch_queue_t queue2 = dispatch_queue_create("com.itachi.queue2", 0);
//创建group
dispatch_group_t group = dispatch_group_create();
//异步执行任务
dispatch_async(queue1, ^{
    NSLog(@"任务1：queue 1...");
    sleep(1);
    NSLog(@":white_check_mark:完成任务1");
});

dispatch_async(queue2, ^{
    NSLog(@"任务1：queue 2...");
    sleep(1);
    NSLog(@":white_check_mark:完成任务2");
});

//将队列加入到group
dispatch_group_async(group, queue1, ^{
    NSLog(@":no_entry_sign:正在暂停1");
    dispatch_suspend(queue1);
});

dispatch_group_async(group, queue2, ^{
    NSLog(@":no_entry_sign:正在暂停2");
    dispatch_suspend(queue2);
});

//等待两个queue执行完毕后再执行
//当将dispatch_group_wait(group, DISPATCH_TIME_FOREVER);注释后，会产生崩溃，因为所有的任务都是异步执行的，在执行恢复queue1和queue2队列的时候，可能这个时候还没有执行queue1和queue2的挂起队列
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
NSLog(@"等待两个queue1完成, 再往下进行");

//异步执行任务
dispatch_async(queue1, ^{
    NSLog(@"任务2：queue 1");
});
dispatch_async(queue2, ^{
    NSLog(@"任务2：queue 2");
});

//在这里将这两个队列重新恢复
dispatch_resume(queue1);
dispatch_resume(queue2);
```

##### 7 进度条实例

```objective-c
//1、指定DISPATCH_SOURCE_TYPE_DATA_ADD，做成Dispatch Source(分派源)。设定Main Dispatch Queue 为追加处理的Dispatch Queue
dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0, 		dispatch_get_main_queue());
__block NSUInteger totalComplete = 0;
dispatch_source_set_event_handler(source, ^{
    //当处理事件被最终执行时，计算后的数据可以通过dispatch_source_get_data来获取。这个数据的值在每次响应事件执行后会被重置，所以totalComplete的值是最终累积的值。
    NSUInteger value = dispatch_source_get_data(source);
    totalComplete += value;
    NSLog(@"进度：%@", @((CGFloat)totalComplete/100));
    NSLog(@":large_blue_circle:线程号：%@", [NSThread currentThread]);
});

//分派源创建时默认处于暂停状态，在分派源分派处理程序之前必须先恢复。
dispatch_resume(source);

dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

//2、恢复源后，就可以通过dispatch_source_merge_data向Dispatch Source(分派源)发送事件:
for (NSUInteger index = 0; index < 100; index++) {
    dispatch_async(queue, ^{
        dispatch_source_merge_data(source, 1);
        NSLog(@":recycle:线程号：%@~~~~~~~~~~~~i = %ld", [NSThread currentThread], index);
        usleep(20000);//0.02秒
    });
}

//3、比较上面的for循环代码，将dispatch_async放在外面for循环的外面，打印结果不一样
dispatch_async(queue, ^{
    for (NSUInteger index = 0; index < 100; index++) {
        dispatch_source_merge_data(source, 1);
        NSLog(@":recycle:线程号：%@~~~~~~~~~~~~i = %ld", [NSThread currentThread], index);
        usleep(20000);//0.02秒
    }
});
//2是将100个任务添加到queue里面，而3是在queue里面添加一个任务，而这一个任务做了100次循环
```


转载自[iOS多线程——Dispatch Source](http://www.jianshu.com/p/880c2f9301b6)