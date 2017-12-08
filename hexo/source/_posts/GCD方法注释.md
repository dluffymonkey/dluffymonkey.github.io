---
title: GCD学习 —— 四
date: 2017-12-09 11:36
tags: iOS
categories: iOS Tips
---
#### GCD方法注释

- `Dispatch  Queues` 中的任务按照FIFO的顺序进行处理，并且，由于加入任务的方式不同，执行分为同步／异步。
- `Dispatch Groups` 可以帮助我们处理如何判断多线程全部执行结束的问题
- `Dispatch Semaphores` 帮助我们控制多任务对有限数量资源的访问
- `Dispatch Objects` 帮助我们对线程队列进行更加细致的控制（挂起、恢复、取消、激活等操作）
- `Dispatch Once` 可以帮助我们确保某个函数只执行一次，可用于单例的实现
  <!-- more -->
```objective-c
/******************dispatch_queue**********************/
/* 
功能：将块函数添加到线程队列中异步执行（异步：执行后不管结果直接返回）
参数：queue：指定的队列   block／work 块函数（context：传入block块函数中的参数）
返回值：空
*/
void dispatch_async(dispatch_queue_t queue, dispatch_block_t block);
void dispatch_async_f(dispatch_queue_t queue, void *_Nullable context, dispatch_function_t work);

/* 
功能：将块函数添加到线程队列中同步执行（异步：执行完成后返回结果）
参数：queue：指定的队列   block／work 块函数（context：传入block块函数中的参数）
返回值：空
*/
void dispatch_sync(dispatch_queue_t queue, dispatch_block_t block);
void dispatch_sync_f(dispatch_queue_t queue, void *_Nullable context, dispatch_function_t work);

/* 
功能：将块函数添加到线程队列中按照迭代次数执行，所有迭代完成后返回结果
参数：iterations：迭代次数  queue：指定的队列   block／work 块函数（context：传入block块函数中的参数）
返回值：空
*/
void dispatch_apply(size_t iterations, dispatch_queue_t queue, DISPATCH_NOESCAPE void (^block)(size_t));
void dispatch_apply_f(size_t iterations, dispatch_queue_t queue, void *_Nullable context, void (*work)(void *_Nullable, size_t));

/*
功能：获取当前执行中的队列
参数：无
返回值：当前队列或者空
*/
dispatch_queue_t dispatch_get_current_queue(void);

/* 
功能：获取主队列
参数：无
返回值：主队列或者空
*/
dispatch_queue_t dispatch_get_main_queue(void)
{
	return DISPATCH_GLOBAL_OBJECT(dispatch_queue_t, _dispatch_main_q);
}

/* 
功能：获取全局并发队列
参数：identifier：队列优先级  typedef long dispatch_queue_priority_t;
\- DISPATCH_QUEUE_PRIORITY_HIGH:         QOS_CLASS_USER_INITIATED
\- DISPATCH_QUEUE_PRIORITY_DEFAULT:      QOS_CLASS_DEFAULT
\- DISPATCH_QUEUE_PRIORITY_LOW:          QOS_CLASS_UTILITY
\- DISPATCH_QUEUE_PRIORITY_BACKGROUND:   QOS_CLASS_BACKGROUND
\#define DISPATCH_QUEUE_PRIORITY_HIGH 2
\#define DISPATCH_QUEUE_PRIORITY_DEFAULT 0
\#define DISPATCH_QUEUE_PRIORITY_LOW (-2)
\#define DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN
  flags：保留供将来使用，输入除了0以外的任何值可能返回空值
返回值：全局队列或者空
*/
dispatch_queue_t dispatch_get_global_queue(long identifier, unsigned long flags);
#define DISPATCH_QUEUE_SERIAL NULL  //串行队列
#define DISPATCH_QUEUE_SERIAL_INACTIVE//暂停状态串行队列
dispatch_queue_attr_make_initially_inactive(DISPATCH_QUEUE_SERIAL)
#define DISPATCH_QUEUE_CONCURRENT //并发队列
DISPATCH_GLOBAL_OBJECT(dispatch_queue_attr_t, _dispatch_queue_attr_concurrent)
#define DISPATCH_QUEUE_CONCURRENT_INACTIVE  //暂停状态并发队列
dispatch_queue_attr_make_initially_inactive(DISPATCH_QUEUE_CONCURRENT)
  
/* 
功能：设置属性值，用于在队列的创建时加入
参数：attr：队列属性值
返回值：队列属性值
*/
dispatch_queue_attr_t dispatch_queue_attr_make_initially_inactive(
dispatch_queue_attr_t _Nullable attr);
#define DISPATCH_QUEUE_SERIAL_WITH_AUTORELEASE_POOL 
dispatch_queue_attr_make_with_autorelease_frequency(
DISPATCH_QUEUE_SERIAL, DISPATCH_AUTORELEASE_FREQUENCY_WORK_ITEM)
#define DISPATCH_QUEUE_CONCURRENT_WITH_AUTORELEASE_POOL
dispatch_queue_attr_make_with_autorelease_frequency(
 DISPATCH_QUEUE_CONCURRENT, DISPATCH_AUTORELEASE_FREQUENCY_WORK_ITEM)
dispatch_queue_attr_t dispatch_queue_attr_make_with_autorelease_frequency(
dispatch_queue_attr_t _Nullable attr,dispatch_autorelease_frequency_t frequency);

/* 
功能：创建队列
参数：label：队列附带信息，可有可无  attr：队列属性值  target：目标队列，相当于目标队列计数加一
返回值：引用的队列
*/
dispatch_queue_t dispatch_queue_create(const char *_Nullable label, dispatch_queue_attr_t _Nullable attr);
dispatch_queue_t dispatch_queue_create_with_target(const char *_Nullable label, dispatch_queue_attr_t _Nullable attr, dispatch_queue_t _Nullable target)

/* 
功能：获取队列描述信息
参数：label：队列附带信息，可有可无  attr：队列属性值  target：目标队列，相当于目标队列计数加一
返回值：队列附带信息
*/
const char *dispatch_queue_get_label(dispatch_queue_t _Nullable queue);

//dispatch_qos_class_t dispatch_queue_get_qos_class(dispatch_queue_t queue,
int *_Nullable relative_priority_ptr);

/* 
功能：给指定对象设置目标队列
参数：object：目标对象  queue：目标队列
返回值：无
*/
void dispatch_set_target_queue(dispatch_object_t object, dispatch_queue_t _Nullable queue);

/* 
功能：dispatch类入口函数
参数：无
返回值：无
*/
void dispatch_main(void);

/* 
功能：在指定时间后再目标队列执行block任务
参数：when：时间 queue：目标队列  block／work：要执行的任务  context：传入任务中的参数
返回值：无
*/
void dispatch_after(dispatch_time_t when, dispatch_queue_t queue, dispatch_block_t block);
void dispatch_after_f(dispatch_time_t when, dispatch_queue_t queue, void *_Nullable context, dispatch_function_t work);

/* 
功能：给指定队列增加一个阻塞其它异步执行任务的任务
参数：queue：队列  block／work：任务  context：传入任务的参数
返回值：无
*/
void dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);
void dispatch_barrier_async_f(dispatch_queue_t queue, void *_Nullable context, dispatch_function_t work);

/* 
功能：给指定队列增加一个阻塞其它同步执行任务的任务
参数：queue：队列  block／work：任务  context：传入任务的参数
返回值：无
*/
void dispatch_barrier_sync(dispatch_queue_t queue, DISPATCH_NOESCAPE dispatch_block_t block);
void dispatch_barrier_sync_f(dispatch_queue_t queue, void *_Nullable context, dispatch_function_t work);

/* 
功能：当指定队列键值改变时，或者是所有属性值都释放后，调用销毁函数destructor
参数：queue：队列  key：键名  context：新内容  destructor：销毁函数
返回值：无
*/
void dispatch_queue_set_specific(dispatch_queue_t queue, const void *key, void *_Nullable context, dispatch_function_t _Nullable destructor);

/* 
功能：获取指定队列特定键内容
参数：queue：队列  key：键名  
返回值：键值
*/
void *_Nullable dispatch_queue_get_specific(dispatch_queue_t queue, const void *key);

/* 
功能：获取当前队列特定键内容
参数： key：键名  
返回值：键值
*/
void *_Nullable dispatch_get_specific(const void *key);

/* 
功能：验证当前块任务运行在指定队列上
参数：queue：队列 
返回值：无
*/
void dispatch_assert_queue(dispatch_queue_t queue)
DISPATCH_ALIAS_V2(dispatch_assert_queue);

/* 
功能：验证当前块任务运行在指定队列上，并且该任务阻塞队列中的其它任务
参数：queue：队列 
返回值：无
*/

void dispatch_assert_queue_barrier(dispatch_queue_t queue);

/* 
功能：验证当前块任务没有运行在指定队列上
参数：queue：队列 
返回值：无
*/
void dispatch_assert_queue_not(dispatch_queue_t queue)
DISPATCH_ALIAS_V2(dispatch_assert_queue_not);

/*************************dispatch_group*********************************/

/* 
功能：创建派遣队列组
参数：无
返回值：队列组
*/
dispatch_group_t dispatch_group_create(void);

/* 
功能：给指定队列添加异步执行任务，将队列加入组
参数：group：队列组 queue：指定队列  block／work：任务  context：传入任务的参数
返回值：无
*/
void dispatch_group_async(dispatch_group_t group, dispatch_queue_t queue, dispatch_block_t block);
void dispatch_group_async_f(dispatch_group_t group, dispatch_queue_t queue, void *_Nullable context, dispatch_function_t work);

/* 
功能：等待（阻塞线程）一直到（队列组中所有任务执行结束或者是时间结束）
参数：group：队列组 timeout：时间
返回值：0表示成功，非0.错误
*/
long dispatch_group_wait(dispatch_group_t group, dispatch_time_t timeout);

/* 
功能：队列组中所有任务执行结束之后，执行新的block 任务
参数：group：任务组 queue：指定队列  block／work：新任务  context：传入任务的参数
返回值：任务组
*/
void dispatch_group_notify(dispatch_group_t group, dispatch_queue_t queue, dispatch_block_t block);
void dispatch_group_notify_f(dispatch_group_t group, dispatch_queue_t queue, void *_Nullable context, dispatch_function_t work);

/* 
功能：管理显示队列组中所有任务
参数：group：队列组 
返回值：队列组
*/
void dispatch_group_enter(dispatch_group_t group);

/* 
功能：管理显示队列组中以执行结束的任务
参数：group：队列组
返回值：队列组
*/
void dispatch_group_leave(dispatch_group_t group);

/*****************************dispatch_semaphore********************************/

/* 
功能：创建信号量
参数：value：信号量资源数
返回值：信号量或空（失败）
*/
dispatch_semaphore_t dispatch_semaphore_create(long value);

/* 
功能：等待获取信号量，获取到后开始继续执行，或是时间结束
参数：dsema：信号量 timeout：限定时间
返回值：无
*/
long dispatch_semaphore_wait(dispatch_semaphore_t dsema, dispatch_time_t timeout);

/* 
功能：执行结束，不需要占用资源，释放信号量
参数：dsema：信号量
返回值：无
*/
long dispatch_semaphore_signal(dispatch_semaphore_t dsema);

/*****************************dispatch_object********************************/

/* 
功能：增加队列引用计数
参数：object：操作队列
返回值：无
*/
void dispatch_retain(dispatch_object_t object);

/* 
功能：减少队列引用计数
参数：object：操作队列
返回值：无
*/
void dispatch_release(dispatch_object_t object);

/* 
功能：获取对象应用程序上下文
参数：object：对象
返回值：定义内容或空
*/
void *_Nullable dispatch_get_context(dispatch_object_t object);

/* 
功能：设置指定对象的应用程序上下文
参数：object：对象 context：上下文内容
返回值：无
*/
void dispatch_set_context(dispatch_object_t object, void *_Nullable context);

/* 
功能：设置对象销毁函数，在该对象所有引用释放后，销毁该对象
参数：object：对象 finalizer：销毁函数指针
返回值：无
*/
void dispatch_set_finalizer_f(dispatch_object_t object, dispatch_function_t _Nullable finalizer);

/* 
功能：激活指定非活动对象
参数：object：对象（一般是线程队列） 
返回值：无
*/
void dispatch_activate(dispatch_object_t object);

/* 
功能：挂起／阻塞指定对象（一般是线程队列）
参数：object：对象 
返回值：无
*/
void dispatch_suspend(dispatch_object_t object);

/* 
功能：恢复指定对象（一般是线程队列）
参数：object：对象 
返回值：无
*/
void dispatch_resume(dispatch_object_t object);

/* 
功能：同步等待一个对象完成操作，或者是直到超出规定时间
参数：object：对象  timeout：限定时间
返回值：0成功，非0失败
*/
long dispatch_wait(void *object, dispatch_time_t timeout);

/* 
功能：在指定对象完成工作后，将一个通知块任务加入指定队列
参数：object：对象  queue：队列 notification_block：通知块
返回值：无
*/
void dispatch_notify(void *object, dispatch_object_t queue, dispatch_block_t notification_block);

/* 
功能：取消指定对象
参数：object：对象 
返回值：无
*/
void dispatch_cancel(void *object);

/* 
功能：判断指定对象是否被取消
参数：object：对象 
返回值：0表示未取消，其它表示取消
*/
long dispatch_testcancel(void *object);

/* 
功能：已编程方式记录指定对象的调试调度信息
参数：object：对象 
返回值：无
*/
void dispatch_debug(dispatch_object_t object, const char *message, ...);
void dispatch_debugv(dispatch_object_t object, const char *message, va_list ap);

/**********************************dispatch_once**********************************/

/* 
功能：只执行任务函数一次
参数：predicate：dispatch_once_t 对象  block／function要执行的任务函数 context：传入的内容
返回值：无
*/
void dispatch_once(dispatch_once_t *predicate, DISPATCH_NOESCAPE dispatch_block_t block);
void dispatch_once_f(dispatch_once_t *predicate, void *_Nullable context, dispatch_function_t function);

/************************************dispatch_time*************************************/

/* 
功能：创建时间对象，在指定时间的基础上再添加一段时间
参数：when：时间  delta：时间段（纳秒）
返回值：时间对象
*/
dispatch_time_t dispatch_time(dispatch_time_t when, int64_t delta);

/* 
功能：创建时间对象，在指定时间的基础上再添加一段时间
参数：when：时间  delta时间段（纳秒）
返回值：时间对象
*/
dispatch_time_t dispatch_walltime(const struct timespec *_Nullable when, int64_t delta);
```