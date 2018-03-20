---
title: NSOperation简介
date: 2017-12-08
tags: iOS
categories: iOS Tips
---

#### 1 NSOperation简介

NSOperation是苹果提供给我们的一套多线程解决方案。实际上NSOperation是基于GCD更高一层的封装，但是比GCD更简单易用、代码可读性也更高。

NSOperation需要配合NSOperationQueue来实现多线程。**因为默认情况下，NSOperation单独使用时系统同步执行操作，并没有开辟新线程的能力，只有配合NSOperationQueue才能实现异步执行。**

因为NSOperation是基于GCD的，那么使用起来也和GCD差不多，其中，NSOperation相当于GCD中的任务，而NSOperationQueue则相当于GCD中的队列。NSOperation实现多线程的使用步骤分为三步：

- 1 创建任务：先将需要执行的操作封装到一个NSOperation对象中。

- 2 创建队列：创建NSOperationQueue对象。

- 3 将任务加入到队列中：然后将NSOperation对象添加到NSOperationQueue中。

之后呢，系统就会自动将NSOperationQueue中的NSOperation取出来，在新线程中执行操作。

下面我们来学习下NSOperation和NSOperationQueue的基本使用。

<!-- more -->

#### 2 NSOperation和NSOperationQueue的基本使用

##### 2.1 创建任务

NSOperation是个抽象类，并不能封装任务。我们只有使用它的子类来封装任务。我们有三种方式来封装任务。

- 1 使用子类NSInvocationOperation
- 2 使用子类NSBlockOperation
- 3 定义继承自NSOperation的子类，通过实现内部相应的方法来封装任务。

在不使用NSOperationQueue，单独使用NSOperation的情况下系统同步执行操作，下面我们学习以下任务的三种创建方式。

###### 2.1.1 使用子类`- NSInvocationOperation:`

```objective-c
// 1.创建NSInvocationOperation对象
NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];

// 2.调用start方法开始执行操作
[op start];

- (void)run
{
    NSLog(@"------%@", [NSThread currentThread]);
}
```

```objective-c
输出结果：
NSOperation[15834:2384555] ------<NSThread: 0x7fa3e2e05410>{number = 1, name = main}
```

从中可以看到，在没有使用NSOperationQueue、单独使用NSInvocationOperation的情况下，NSInvocationOperation在主线程执行操作，并没有开启新线程。

###### 2.1.2 使用子类`- NSBlockOperation`

```objective-c
NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
    // 在主线程
    NSLog(@"------%@", [NSThread currentThread]);
}];

[op start];
```

```objective-c
输出结果：
NSOperation[15884:2387780] ------<NSThread: 0x7fb2196012c0>{number = 1, name = main}
```

我们同样可以看到，在没有使用NSOperationQueue、单独使用NSBlockOperation的情况下，NSBlockOperation也是在主线程执行操作，并没有开启新线程。

**但是，NSBlockOperation还提供了一个方法`addExecutionBlock:`，通过`addExecutionBlock:`就可以为NSBlockOperation添加额外的操作，这些额外的操作就会在其他线程并发执行。**

```objective-c
- (void)blockOperation
{
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        // 在主线程
        NSLog(@"1------%@", [NSThread currentThread]);
    }];    

    // 添加额外的任务(在子线程执行)
    [op addExecutionBlock:^{
        NSLog(@"2------%@", [NSThread currentThread]);
    }];
    [op addExecutionBlock:^{
        NSLog(@"3------%@", [NSThread currentThread]);
    }];
    [op addExecutionBlock:^{
        NSLog(@"4------%@", [NSThread currentThread]);
    }];

    [op start];
}
```

```objective-c
输出结果：
NSOperation[15896:2390616] 1------<NSThread: 0x7ff633f03be0>{number = 1, name = main}
NSOperation[15896:2390825] 2------<NSThread: 0x7ff633e24600>{number = 2, name = (null)}
NSOperation[15896:2390657] 3------<NSThread: 0x7ff633c411e0>{number = 3, name = (null)}
NSOperation[15896:2390656] 4------<NSThread: 0x7ff633f1d3e0>{number = 4, name = (null)}
```

可以看出，`blockOperationWithBlock:`方法中的操作是在主线程中执行的，而`addExecutionBlock:`方法中的操作是在其他线程中执行的。

###### 2.1.3 定义继承自NSOperation的子类

h文件

```objective-c
#import <Foundation/Foundation.h>
@interface LQOperation : NSOperation

@end
```

m文件

```objective-c
#import "LQOperation.h"

@implementation LQOperation
/**
 * 需要执行的任务
 */
- (void)main
{
    for (int i = 0; i < 2; ++i) {
        NSLog(@"1-----%@",[NSThread currentThread]);
    }    
}
@end
```

然后使用的时候导入头文件`LQOperation.h`。

```objective-c
// 创建YSCOperation
LQOperation *op = [[LQOperation alloc] init];
[op start];
```

```objective-c
输出结果：
NSOperation[16566:2501606] 1-----<NSThread: 0x7f8030d05150>{number = 1, name = main}
NSOperation[16566:2501606] 1-----<NSThread: 0x7f8030d05150>{number = 1, name = main}
```

可以看出：在没有使用NSOperationQueue、单独使用自定义子类的情况下，是在主线程执行操作，并没有开启新线程。

##### 2.2 创建队列

和GCD中的并发队列、串行队列略有不同的是：`NSOperationQueue`一共有两种队列：主队列、其他队列。其中其他队列同时包含了串行、并发功能。下边是主队列、其他队列的基本创建方法和特点

- 主队列

  - 凡是添加到主队列中的任务（NSOperation），都会放到主线程中执行

    ```objective-c
    NSOperationQueue *queue = [NSOperationQueue mainQueue];
    ```

- 其他队列（非主队列）

  - 添加到这种队列中的任务（NSOperation），就会自动放到子线程中执行

  - 同时包含了：串行、并发功能

    ```objective-c
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    ```

##### 2.3 将任务加入到队列中

前边说了，NSOperation需要配合NSOperationQueue来实现多线程。
那么我们需要将创建好的任务加入到队列中去。总共有两种方法

###### 2.3.1 ``-(void)addOperation:(NSOperation *)op;``

- 需要先创建任务，再将创建好的任务加入到创建好的队列中去

  ```objective-c
  - (void)addOperationToQueue
  {
      // 1.创建队列
      NSOperationQueue *queue = [[NSOperationQueue alloc] init];

      // 2. 创建操作  
      // 创建NSInvocationOperation    
      NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];    
      // 创建NSBlockOperation    
      NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
          for (int i = 0; i < 2; ++i) {
              NSLog(@"1-----%@", [NSThread currentThread]);
          }
      }];

      // 3. 添加操作到队列中：addOperation:   
      [queue addOperation:op1]; // [op1 start]    
      [queue addOperation:op2]; // [op2 start]
  }

  - (void)run
  {
      for (int i = 0; i < 2; ++i) {
          NSLog(@"2-----%@", [NSThread currentThread]);
      }
  }
  ```

  ```objective-c
  输出结果：
  NSOperationQueue[16201:2452281] 1-----<NSThread: 0x7fe4824080e0>{number = 3, name = (null)}
  NSOperationQueue[16201:2452175] 2-----<NSThread: 0x7fe482404a50>{number = 2, name = (null)}
  NSOperationQueue[16201:2452175] 2-----<NSThread: 0x7fe482404a50>{number = 2, name = (null)}
  NSOperationQueue[16201:2452281] 1-----<NSThread: 0x7fe4824080e0>{number = 3, name = (null)}
  ```

  可以看出：NSInvocationOperation和NSOperationQueue结合后能够开启新线程，进行并发执行，NSBlockOperation和NSOperationQueue也能够开启新线程，进行并发执行。

###### 2.3.2 ``- (void)addOperationWithBlock:(void (^)(void))block;``

- 无需先创建任务，在block中添加任务，直接将任务block加入到队列中。

  ```objective-c
  - (void)addOperationWithBlockToQueue
  {
      // 1. 创建队列
      NSOperationQueue *queue = [[NSOperationQueue alloc] init];

      // 2. 添加操作到队列中：addOperationWithBlock:
      [queue addOperationWithBlock:^{
          for (int i = 0; i < 2; ++i) {
              NSLog(@"-----%@", [NSThread currentThread]);
          }
      }];
  }
  ```

  ```objective-c
  输出结果：
  NSOperationQueue[16293:2457487] -----<NSThread: 0x7ffa6bc0e1e0>{number = 2, name = (null)}
  NSOperationQueue[16293:2457487] -----<NSThread: 0x7ffa6bc0e1e0>{number = 2, name = (null)}
  ```

  可以看出addOperationWithBlock:和NSOperationQueue能够开启新线程，进行并发执行。

#### 3 控制串行执行和并行执行的关键

之前我们说过，NSOperationQueue创建的其他队列同时具有串行、并发功能，上边我们演示了并发功能，那么他的串行功能是如何实现的？

这里有个关键参数`maxConcurrentOperationCount`，叫做**最大并发数**。

- 最大并发数：`maxConcurrentOperationCount`

  - `maxConcurrentOperationCount`默认情况下为-1，表示不进行限制，默认为并发执行。
  - 当`maxConcurrentOperationCount`为1时，进行串行执行。
  - 当`maxConcurrentOperationCount`大于1时，进行并发执行，当然这个值不应超过系统限制，即使自己设置一个很大的值，系统也会自动调整。

  ```objective-c
  - (void)opetationQueue
  {
      // 创建队列
      NSOperationQueue *queue = [[NSOperationQueue alloc] init];

      // 设置最大并发操作数
      //    queue.maxConcurrentOperationCount = 2;
      queue.maxConcurrentOperationCount = 1; // 就变成了串行队列

      // 添加操作
      [queue addOperationWithBlock:^{
          NSLog(@"1-----%@", [NSThread currentThread]);
          [NSThread sleepForTimeInterval:0.01];
      }];
      [queue addOperationWithBlock:^{
          NSLog(@"2-----%@", [NSThread currentThread]);
          [NSThread sleepForTimeInterval:0.01];
      }];
      [queue addOperationWithBlock:^{
          NSLog(@"3-----%@", [NSThread currentThread]);
          [NSThread sleepForTimeInterval:0.01];
      }];
      [queue addOperationWithBlock:^{
          NSLog(@"4-----%@", [NSThread currentThread]);
          [NSThread sleepForTimeInterval:0.01];
      }];
      [queue addOperationWithBlock:^{
          NSLog(@"5-----%@", [NSThread currentThread]);
          [NSThread sleepForTimeInterval:0.01];
      }];

      [queue addOperationWithBlock:^{
          NSLog(@"6-----%@", [NSThread currentThread]);
          [NSThread sleepForTimeInterval:0.01];
      }];
  }
  ```

  ```objective-c
  最大并发数为1输出结果：
  NSOperationQueue[16320:2464630] 1-----<NSThread: 0x7fc892d0b3a0>{number = 2, name = (null)}
  NSOperationQueue[16320:2464631] 2-----<NSThread: 0x7fc892c0a7b0>{number = 3, name = (null)}
  NSOperationQueue[16320:2464630] 3-----<NSThread: 0x7fc892d0b3a0>{number = 2, name = (null)}
  NSOperationQueue[16320:2464631] 4-----<NSThread: 0x7fc892c0a7b0>{number = 3, name = (null)}
  NSOperationQueue[16320:2464631] 5-----<NSThread: 0x7fc892c0a7b0>{number = 3, name = (null)}
  NSOperationQueue[16320:2464630] 6-----<NSThread: 0x7fc892d0b3a0>{number = 2, name = (null)}
  ```

  ```objective-c
  最大并发数为2输出结果：
  NSOperationQueue[16331:2466366] 2-----<NSThread: 0x7fd729f0f270>{number = 3, name = (null)}
  NSOperationQueue[16331:2466491] 1-----<NSThread: 0x7fd729f4e290>{number = 2, name = (null)}
  NSOperationQueue[16331:2466367] 3-----<NSThread: 0x7fd729d214e0>{number = 4, name = (null)}
  NSOperationQueue[16331:2466366] 4-----<NSThread: 0x7fd729f0f270>{number = 3, name = (null)}
  NSOperationQueue[16331:2466366] 6-----<NSThread: 0x7fd729f0f270>{number = 3, name = (null)}
  NSOperationQueue[16331:2466511] 5-----<NSThread: 0x7fd729e056c0>{number = 5, name = (null)}
  ```

  可以看出：当最大并发数为1时，任务是按顺序串行执行的。当最大并发数为2时，任务是并发执行的。而且开启线程数量是由系统决定的，不需要我们来管理。这样看来，是不是比GCD还要简单了许多？

#### 4 操作依赖

NSOperation和NSOperationQueue最吸引人的地方是它能添加操作之间的依赖关系。比如说有A、B两个操作，其中A执行完操作，B才能执行操作，那么就需要让B依赖于A。具体如下：

```objective-c
- (void)addDependency
{
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];

    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"1-----%@", [NSThread  currentThread]);
    }];
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"2-----%@", [NSThread  currentThread]);
    }];

    [op1 addDependency:op2];    // 让op1 依赖于 op2，则先执行op2，再执行op1

    [queue addOperation:op1];
    [queue addOperation:op2];
}
```

```objective-c
输出结果：
操作依赖[16423:2484866] 2-----<NSThread: 0x7fc138e1e7c0>{number = 2, name = (null)}
操作依赖[16423:2484866] 1-----<NSThread: 0x7fc138e1e7c0>{number = 2, name = (null)}
```

可以看到，无论运行几次，其结果都是op2先执行，op1后执行。

#### 5 一些其他方法

- `- (void)cancel;` NSOperation提供的方法，可取消单个操作
- `- (void)cancelAllOperations;` NSOperationQueue提供的方法，可以取消队列的所有操作
- `- (void)setSuspended:(BOOL)b;` 可设置任务的暂停和恢复，YES代表暂停队列，NO代表恢复队列
- `- (BOOL)isSuspended;` 判断暂停状态

**注意：**

- 这里的暂停和取消并不代表可以将当前的操作立即取消，而是当当前的操作执行完毕之后不再执行新的操作。

- 暂停和取消的区别就在于：暂停操作之后还可以恢复操作，继续向下执行；而取消操作之后，所有的操作就清空了，无法再接着执行剩下的操作。

  ​

转载自[iOS多线程--彻底学会多线程](http://www.jianshu.com/p/4b1d77054b35)