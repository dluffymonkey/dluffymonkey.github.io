---
title: 如何选择修饰关键字
date: 2016-11-25
tags: iOS
categories: iOS Tips
---


1.代理用nonatomic, weak修饰
weak:指明该对象并不负责保持delegate这个对象，delegate这个对象的销毁由外部控制，会在dealloc方法中销毁。

    @property (nonatomic, weak) id <LWDelegate> delegate;

strong：该对象强引用delegate，外界不能销毁delegate对象，会导致循环引用(Retain Cycles)，不会在dealloc方法中销毁，会导致内存泄漏。

    @property (nonatomic, strong) id <LWDelegate> delegate;

<!-- more -->
2.block块用nonatomic，copy
block属性的声明，首先需要用copy修饰符，因为只有copy后的block才会在堆中，栈中的block的生命周期是和栈绑定的，加了copy属性后，当其所在栈被释放的时候，这些本地变量将变得不可访问，一旦代码执行到block这段就会导致bad access。另一个需要注意的问题是关于线程安全，在声明Block属性时需要确认在调用Block时另一个线程有没有可能去修改Block，如果确定不会有这种情况发生的话，那么block属性声明可以用nonatomic。

3.NSString对象一般用nonatomic，copy（因为字符串一般要遵循NSCopying协议）
我们在声明NSString对象的时候，一般都是不希望外部来对他进行修改的，防止将NSMutableString赋值给NSString时，前者修改引起后者值变化而用的。

    @property (nonatomic, copy) NSString *lwcopyStr;
    @property (nonatomic, strong) NSString *lwstrongStr;
    
    NSMutableString *lwmuStr = [NSMutableString stringWithString:@"LWW -- MutableString"];

    self.lwcopyStr = lwmuStr;
    self.lwstrongStr = lwmuStr;
    
    [lwmuStr appendString:@"---copy"];

    NSLog(@"lwmuStr %p lwcopyStr  %p lwstrongStr  %p", lwmuStr, self.lwcopyStr,  self.lwstrongStr);
    NSLog(@"%@   %@", self.lwcopyStr, self.lwstrongStr);

![结果图.png](http://upload-images.jianshu.io/upload_images/293993-a907c5711c7921ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**注意**：在这里要用self.属性，不能用下划线属性，因为下划线是直接访问属性，没有经过get方法，看不到效果。

4.BOOL、CGFloat、NSInteger、int等一般用nonatomic，assign
assign: 简单赋值，不更改索引计数对基础数据类型，例如NSInteger，CGFloat和C数据类型（int, float, double, char等。适用简单数据类型。此标记说明设置器直接进行赋值，这也是默认值。

5.结构体，枚举属性一般用nonatomic，assign
因为在结构体里面的每个枚举成员均具有相关联的常数值，此值的类型就是包含了它的那个枚举的基础类型。每个枚举成员的常数值必须在该枚举的基础类型的范围之内，故枚举跟上面的简单数据类型一样，都是用assign的，进行简单赋值就行。每个枚举值却的的确确都有一个整数类型的数值相对应，而且可以转换，只不过这种转换必须用明晰的转型语法来表达。
**注意**：枚举不能被继承

6.一般的属性对象用nonatomic，strong
**注意**：如果是用XIB或者storyBoard拖的则是用weak，自己手动创建则是用strong。
iOS 5 中对属性的设置新增了strong 和weak关键字来修饰属性
strong 用来修饰强引用的属性；

    @property (strong) SomeClass * aObject;

对应原来的

    @property (retain) SomeClass * aObject; 
    @property (copy) SomeClass * aObject;

weak 用来修饰弱引用的属性；

    @property (weak) SomeClass * aObject;
对应原来的

    @property (assign) SomeClass * aObject;