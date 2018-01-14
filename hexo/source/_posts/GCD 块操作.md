---
title: GCD学习 —— 三
date: 2017-12-08 19:32
tags: iOS
categories: iOS Tips
---

​	学习学习dispatch_block，在向队列中添加任务时，可以直接在对应的函数中添加 `block`。但是如果想对任务进行操作，比如监听任务、取消任务，就需要获取对应的 `block`。

#### 1 创建Block

- 第一种方式如下：

  ```objective-c
  dispatch_block_t dispatch_block_create(dispatch_block_flags_t flags, dispatch_block_t block);
  ```

  ​	在该函数中，`flags` 参数用来设置 `block` 的标记，`block` 参数用来设置具体的任务。`flags` 的类型为 `dispatch_block_flags_t` 的枚举，用于设置 `block` 的标记，定义如下：

  ```objective-c
  DISPATCH_ENUM(dispatch_block_flags, unsigned long,
      DISPATCH_BLOCK_BARRIER = 0x1,
      DISPATCH_BLOCK_DETACHED = 0x2,
      DISPATCH_BLOCK_ASSIGN_CURRENT = 0x4,
      DISPATCH_BLOCK_NO_QOS_CLASS = 0x8,
      DISPATCH_BLOCK_INHERIT_QOS_CLASS = 0x10,
      DISPATCH_BLOCK_ENFORCE_QOS_CLASS = 0x20,
  );
  ```
  <!-- more -->
- 第二种方式如下：

  ```objective-c
  dispatch_block_t dispatch_block_create_with_qos_class(dispatch_block_flags_t flags,
      dispatch_qos_class_t qos_class, int relative_priority,
      dispatch_block_t block);
  ```

  ​相比于 `dispatch_block_create` 函数，这种方式在创建 `block` 的同时可以指定了相应的优先级。`dispatch_qos_class_t` 是 `qos_class_t` 的别名，定义如下：

  ```objective-c
  #if __has_include(<sys/qos.h>)
  typedef qos_class_t dispatch_qos_class_t;
  #else
  typedef unsigned int dispatch_qos_class_t;
  #endif
  ```

  `qos_class_t` 是一种枚举，有以下类型：

  - QOS_CLASS_USER_INTERACTIVE：`user interactive` 等级表示任务需要被立即执行，用来在响应事件之后更新 UI，来提供好的用户体验。这个等级最好保持小规模。
  - QOS_CLASS_USER_INITIATED：`user initiated` 等级表示任务由 UI 发起异步执行。适用场景是需要及时，结果同时又可以继续交互的时候。
  - QOS_CLASS_DEFAULT：`default` 默认优先级
  - QOS_CLASS_UTILITY：`utility` 等级表示需要长时间运行的任务，伴有用户可见进度指示器。经常会用来做计算，I/O，网络，持续的数据填充等任务。这个任务节能。
  - QOS_CLASS_BACKGROUND：`background` 等级表示用户不会察觉的任务，使用它来处理预加载，或者不需要用户交互和对时间不敏感的任务。
  - QOS_CLASS_UNSPECIFIED：`unspecified` 未指明

  egg：

  ```objective-c
  dispatch_queue_t concurrentQuene = dispatch_queue_create("concurrentQuene", DISPATCH_QUEUE_CONCURRENT);

  dispatch_block_t block = dispatch_block_create(0, ^{
      NSLog(@"normal do some thing...");
  });
  dispatch_async(concurrentQuene, block);

  dispatch_block_t qosBlock = dispatch_block_create_with_qos_class(0, QOS_CLASS_USER_INTERACTIVE, 0, ^{
      NSLog(@"qos do some thing...");
  });
  dispatch_async(concurrentQuene, qosBlock);
  ```

  ```objective-c
  LQHelper[1528:112987] qos do some thing...
  LQHelper[1528:112985] normal do some thing...
  ```




#### 2 监听 block 执行结束

​	有时我们需要等待特定的 `block` 执行完成之后，再去执行其他任务。有两种方法可以获取到指定 `block` 执行结束的时机。

- 第一种方式如下：

  ```objective-c
  long dispatch_block_wait(dispatch_block_t block, dispatch_time_t timeout);
  ```

  ​	该函数会阻塞当前线程进行等待。传入需要设置的 block 和等待时间 timeout。timeout 参数表示函数在等待 block 执行完毕时，应该等待多久。如果执行 block 所需的时间小于 timeout，则返回 0，否则返回非 0 值。此参数也可以取常量 `DISPATCH_TIME_FOREVER`，这表示函数会一直等待 block 执行完，而不会超时。可以使用 dispatch_time 函数和 `DISPATCH_TIME_NOW` 常量来方便的设置具体的超时时间。

  ​	如果 block 执行完成，`dispatch_block_wait` 就会立即返回。不能使用 `dispatch_block_wait` 来等待同一个 block 的多次执行全部结束；这种情况可以考虑使用 `dispatch_group_wait` 来解决。也不能在多个线程中，同时等待同一个 block 的结束。同一个 block 只能执行一次，被等待一次。

  **注意：**因为 `dispatch_block_wait` 会阻塞当前线程，所以不应该放在主线程中调用。

  egg：

  ```objective-c
  dispatch_queue_t concurrentQuene = dispatch_queue_create("concurrentQuene", DISPATCH_QUEUE_CONCURRENT);

  dispatch_async(concurrentQuene, ^{
      dispatch_queue_t allTasksQueue = dispatch_queue_create("allTasksQueue", DISPATCH_QUEUE_CONCURRENT);

      dispatch_block_t block = dispatch_block_create(0, ^{
          NSLog(@"开始执行");
          [NSThread sleepForTimeInterval:3];
          NSLog(@"结束执行");
      });

      dispatch_async(allTasksQueue, block);
      // 等待时长，10s 之后超时
      dispatch_time_t timeout = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(10 * NSEC_PER_SEC));
      long resutl = dispatch_block_wait(block, timeout);
      if (resutl == 0) {
          NSLog(@"执行成功");
      } else {
          NSLog(@"执行超时");
      }
  });
  ```

  ```objective-c
  执行结果：(把上面的sleepForTimeInterval:时间设置成3s)
  LQHelper[1582:121193] 开始执行
  LQHelper[1582:121194] 结束执行
  LQHelper[1582:121193] 执行成功
  执行结果：(把上面的sleepForTimeInterval:时间设置成12s)
  LQHelper[1582:121193] 开始执行
  LQHelper[1582:121194] 执行超时
  LQHelper[1582:121193] 结束执行
  ```

- 第二种方法如下：

  ```objective-c
  void dispatch_block_notify(dispatch_block_t block, dispatch_queue_t queue, dispatch_block_t notification_block);
  ```

  ​	该函数接收三个参数，第一个参数是需要监视的 block，第二个参数是监听的 block 执行结束之后要提交执行的队列 queue，第三个参数是待加入到队列中的 block。 和 `dispatch_block_wait` 的不同之处在于：`dispatch_block_notify` 函数不会阻塞当前线程。

  egg：

  ```objective-c
  NSLog(@"---- 开始设置任务 ----");
  dispatch_queue_t serialQueue =   dispatch_queue_create("com.itachi.serialqueue",   DISPATCH_QUEUE_SERIAL);

  // 耗时任务
  dispatch_block_t taskBlock = dispatch_block_create(0, ^{
      NSLog(@"开始耗时任务");
      [NSThread sleepForTimeInterval:2.f];
      NSLog(@"完成耗时任务");
  });

  dispatch_async(serialQueue, taskBlock);

  // 更新 UI
  dispatch_block_t refreshUI = dispatch_block_create(0, ^{
      NSLog(@"更新 UI");
  });

  // 设置监听
  dispatch_block_notify(taskBlock, dispatch_get_main_queue(), refreshUI);
  NSLog(@"---- 完成设置任务 ----");
  ```

  ```objective-c
  执行结果：
  LQHelper[1615:127135] ---- 开始设置任务 ----
  LQHelper[1615:127135] ---- 完成设置任务 ----
  LQHelper[1615:127227] 开始耗时任务
  LQHelper[1615:127227] 完成耗时任务
  LQHelper[1615:127135] 更新 UI
  ```

  ​

#### 3 任务的取消

iOS8 后 GCD 支持对 `dispatch block` 的取消。方法如下：

```objective-c
void dispatch_block_cancel(dispatch_block_t block);
```

​	这个函数用异步的方式取消指定的 block。取消操作使将来执行 `dispatch block` 立即返回，但是对已经在执行的 `dispatch block` 没有任何影响。当一个 block 被取消时，它会立即释放捕获的资源。如果要在一个 block 中对某些对象进行释放操作，在取消这个 block 的时候，需要确保内存不会泄漏。

```objective-c
dispatch_queue_t serialQueue = dispatch_queue_create("com.itachi.serialqueue", DISPATCH_QUEUE_SERIAL);
// 耗时任务
dispatch_block_t firstTaskBlock = dispatch_block_create(0, ^{
    NSLog(@"开始第一个任务");
    [NSThread sleepForTimeInterval:1.5f];
    NSLog(@"结束第一个任务");
});

// 耗时任务
dispatch_block_t secTaskBlock = dispatch_block_create(0, ^{
    NSLog(@"开始第二个任务");
    [NSThread sleepForTimeInterval:2.f];
    NSLog(@"结束第二个任务");
});

dispatch_async(serialQueue, firstTaskBlock);
dispatch_async(serialQueue, secTaskBlock);

// 等待1s，让第一个任务开始运行
[NSThread sleepForTimeInterval:1];

dispatch_block_cancel(firstTaskBlock);
NSLog(@"尝试过取消第一个任务");

dispatch_block_cancel(secTaskBlock);
NSLog(@"尝试过取消第二个任务");
```

```objective-c
执行结果：
LQHelper[1645:130881] 开始第一个任务
LQHelper[1645:130771] 尝试过取消第一个任务
LQHelper[1645:130771] 尝试过取消第二个任务
LQHelper[1645:130881] 结束第一个任务
```

可见 `dispatch_block_cancel` 对已经在执行的任务不起作用，只能取消尚未执行的任务。

转载自[GCD 之任务操作](http://www.jianshu.com/p/5a16dfd36fad)