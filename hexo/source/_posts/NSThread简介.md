---
title: NSThread简介
date: 2017-12-10
tags: iOS
categories: iOS Tips
---

#### 1 pthread

​	pthread简单介绍下，pthread是一套通用的多线程的API，可以在Unix / Linux / Windows 等系统跨平台使用，使用C语言编写，需要程序员自己管理线程的生命周期，使用难度较大，所以我们在iOS开发中几乎不使用pthread，简单了解下就可以了。

>POSIX线程（英语：POSIX Threads，常被缩写为Pthreads）是POSIX的线程标准，定义了创建和操纵线程的一套API。实现POSIX 线程标准的库常被称作Pthreads，一般用于Unix-like POSIX 系统，如Linux、Solaris。但是Microsoft Windows上的实现也存在，例如直接使用Windows API实现的第三方库pthreads-w32；而利用Windows的SFU/SUA子系统，则可以使用微软提供的一部分原生POSIX API。

<!-- more -->


##### 1.1 pthread的使用方法

- 首先要包含头文件`#import <pthread.h>`

- 其次要创建线程，并开启线程执行任务

  ```objective-c
  // 创建线程——定义一个pthread_t类型变量
  pthread_t thread;
  // 开启线程——执行任务
  pthread_create(&thread, NULL, run, NULL);

  void *run(void *param)    // 新线程调用方法，里边为需要执行的任务
  {
      NSLog(@"%@", [NSThread currentThread]);

      return NULL;
  }
  ```

- `pthread_create(&thread, NULL, run, NULL);` 中各项参数含义：

  - 第一个参数&thread是线程对象
  - 第二个和第四个是线程属性，可赋值NULL
  - 第三个run表示指向函数的指针(run对应函数里是需要在新线程中执行的任务)



#### 2 NSThread

​	NSThread是苹果官方提供的，使用起来比pthread更加面向对象，简单易用，可以直接操作线程对象。不过也需要需要程序员自己管理线程的生命周期(主要是创建)，我们在开发的过程中偶尔使用NSThread。比如我们会经常调用[NSThread currentThread]来显示当前的进程信息。

##### 2.1 创建、启动线程

- 先创建线程，再启动线程

  ```objective-c
  NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];
  [thread start];    // 线程一启动，就会在线程thread中执行self的run方法
  ```

- 创建线程后自动启动线程

  ```objective-c
  [NSThread detachNewThreadSelector:@selector(run) toTarget:self withObject:nil];
  ```

- 隐式创建并启动线程

  ```objective-c
  [self performSelectorInBackground:@selector(run) withObject:nil];
  ```

##### 2.2  线程相关用法

```objective-c
// 获得主线程
+ (NSThread *)mainThread;    

// 判断是否为主线程(对象方法)
- (BOOL)isMainThread;

// 判断是否为主线程(类方法)
+ (BOOL)isMainThread;    

// 获得当前线程
NSThread *current = [NSThread currentThread];

// 线程的名字——setter方法
- (void)setName:(NSString *)n;    

// 线程的名字——getter方法
- (NSString *)name;   
```

##### 2.3 线程状态控制方法

- 启动线程方法

  ```objective-c
  // 线程进入就绪状态 -> 运行状态。当线程任务执行完毕，自动进入死亡状态
  - (void)start;
  ```

- 阻塞（暂停）线程方法

  ```objective-c
  // 线程进入阻塞状态
  + (void)sleepUntilDate:(NSDate *)date;
  + (void)sleepForTimeInterval:(NSTimeInterval)ti;
  ```

- 强制停止线程

  ```objective-c
  // 线程进入死亡状态
  + (void)exit;
  ```

##### 2.4 线程的状态转换

​	当我们新建一条线程`NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil];`，在内存中的表现为：	![1](/images/2017_12_08_thread_one.png)	当调用`[thread start];`后，系统把线程对象放入可调度线程池中，线程对象进入就绪状态，如下图所示。

![2](/images/2017_12_08_thread_two.png)

​	当然，可调度线程池中，会有其他的线程对象，如下图所示。在这里我们只关心左边的线程对象。

![3](/images/2017_12_08_thread_three.png)

**下边我们来看看当前线程的状态转换。**

- 如果CPU现在调度当前线程对象，则当前线程对象进入运行状态，如果CPU调度其他线程对象，则当前线程对象回到就绪状态。


- 如果CPU在运行当前线程对象的时候调用了sleep方法\等待同步锁，则当前线程对象就进入了阻塞状态，等到sleep到时\得到同步锁，则回到就绪状态。


- 如果CPU在运行当前线程对象的时候线程任务执行完毕\异常强制退出，则当前线程对象进入死亡状态。

![4](/images/2017_12_08_thread_four.png)

#### 3 线程同步

##### 3.1 第一种方式@synchronized(对象)关键字

```objective-c
-(void)taskRun
{
    while (count>0) {
        @synchronized(self) { // 需要锁定的代码
            [NSThread sleepForTimeInterval:0.1];
            count--;
            NSLog(@"threadName:%@ count:%d ",[NSThread currentThread].name, count);
        }
    }
}
```

##### 3.2 第二种方式NSLock同步锁

```objective-c
threadLock = [[NSLock alloc] init];
```

然后在需要加锁的代码块开始时调用 lock函数 在结束时调用unLock函数

```objective-c
-(void)taskRun
{
    while (count>0) {
        [threadLock lock];
        [NSThread sleepForTimeInterval:0.1];
        count--;
        NSLog(@"threadName:%@ count:%d ",[NSThread currentThread].name, count);
    }
    [threadLock unlock];
}
```

##### 3.3 第三种方式使用NSCondition同步锁和线程检查器

​	锁主要为了当检测条件时保护数据源，执行条件引发的任务；线程检查器主要是根据条件决定是否继续运行线程，即线程是否被阻塞。先创建一个NSCondition对象

```objective-c
condition = [[NSCondition alloc] init];
```

使用同步锁的方式和NSLock相似

```objective-c
-(void)taskRun
{
    while (count>0) {
        [condition lock];
        [NSThread sleepForTimeInterval:0.1];
        count--;
        NSLog(@"threadName:%@ count:%d ",[NSThread currentThread].name, count);
        [condition unlock];
    }
}
```
NSCondition可以让线程进行等待，然后获取到CPU发信号告诉线程不用在等待，可以继续执行，上述的例子我们稍作修改，我们让线程三专门用于发送信号源

```objective-c
NSThread *thread1=[[NSThread alloc]initWithTarget:self selector:@selector(taskRun) object:nil];
thread1.name=@"thread-1";

NSThread *thread2=[[NSThread alloc]initWithTarget:self selector:@selector(taskRun) object:nil];
thread2.name=@"thread-2";

NSThread *thread3=[[NSThread alloc]initWithTarget:self selector:@selector(taskRun1) object:nil];
thread3.name=@"thread-3";

[thread1 start];
[thread2 start];
[thread3 start];
```

taskRun1函数用于发送信号源

```objective-c
-(void)taskRun1
{
    while (YES) {
        [condition lock];
        [NSThread sleepForTimeInterval:2];
        [condition signal];
        [condition unlock];
    }
}
```

taskRun函数 用于执行对count的操作

```objective-c
-(void)taskRun
{
    while (count>0) {
        [condition lock];
        [condition wait];
        [NSThread sleepForTimeInterval:0.1];
        count--;
        NSLog(@"threadName:%@ count:%d ",[NSThread currentThread].name, count);
        [condition unlock];
    }
}
```

执行的结果会发现，只有在Thread-1、和Thread-2 收到信号源的时候才会执行count--，否则一直出于等待状态。



转载自[iOS多线程--彻底学会多线程](http://www.jianshu.com/p/cbaeea5368b1)