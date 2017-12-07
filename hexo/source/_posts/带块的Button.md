---
title: 带块的button
date: 2017-12-06 18:06
tags: iOS
categories: iOS Tips
---

今天简单的记录一下怎么用块来实现Button的点击事件

首先，我想要的做的效果是直接通过类方法来初始化Button，然后同时把点击事件的操作放在块中，最后返回创建好的Button。

- 第一步，创建一个UIButton的扩展(category)

- 第二步，在h文件中添加初始化Button的类方法声明
  <!-- more -->


```objective-c
#import <UIKit/UIKit.h>

typedef void (^ActionBlock)(UIButton *button);

@interface UIButton (Block)

@property (nonatomic,copy) ActionBlock actionBlock;

/**
 通过block对Button的点击事件进行封装
 
 @param frame frame大小
 @param title 内容
 @param titleColor 内容颜色
 @param bgImgName 背景图片
 @param completion 点击事件
 @return Button
 */
+ (UIButton *)createButtonWithFrame:(CGRect)frame
                              title:(NSString *)title
                         titleColor:(UIColor *)titleColor
                        bgImageName:(NSString *)bgImgName
                        actionBlock:(void(^)(UIButton *sender))completion;


/**
 通过block对Button的点击事件进行封装

 @return 返回初始化后的button
 */
+ (UIButton *)button;

@end

```

- 第三步，在m文件中实现方法
```objective-c
#import "UIButton+Block.h"
#import <objc/runtime.h>

static NSString *keyWithMethod = @"keyWithMethod"; //关联对象的key
static NSString *keyWithBlock = @"keyWithBlock";

@implementation UIButton (Block)

+ (UIButton *)createButtonWithFrame:(CGRect)frame
                              title:(NSString *)title
                         titleColor:(UIColor *)titleColor
                        bgImageName:(NSString *)bgImgName
                        actionBlock:(void (^)(UIButton *sender))completion {
    
    UIButton *button = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    button.frame = frame;
    [button setTitle:title forState:UIControlStateNormal];
    [button setTitleColor:titleColor forState:UIControlStateNormal];
    [button setBackgroundImage:[UIImage imageNamed:bgImgName] forState:UIControlStateNormal];
    [button addTarget:button action:@selector(buttonTapAction:) forControlEvents:UIControlEventTouchUpInside];
    
    /*
     *用runtime中的函数通过key关联对象
     *
     *objc_setAssociatedObject(id _Nonnull object, const void * _Nonnull key, id _Nullable value, objc_AssociationPolicy policy)
     *id object                 表示关联者，是一个对象，变量名也是object
     *const void *key           获取被关联者的索引
     *id value                  被关联者，这里是一个block
     *objc_AssociationPolicy    policy 关联时采用的协议，有assign，retain，copy等协议，一般使用OBJC_ASSOCIATION_RETAIN_NONATOMIC
     */
    objc_setAssociatedObject(button, &keyWithMethod, completion, OBJC_ASSOCIATION_COPY_NONATOMIC);
    
    return button;
}

+ (UIButton *)button {
    UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
    [button addTarget:button action:@selector(buttonTapAction:) forControlEvents:UIControlEventTouchUpInside];
    return button;
}

- (void)setActionBlock:(ActionBlock)actionBlock{
    objc_setAssociatedObject(self, &keyWithBlock, actionBlock, OBJC_ASSOCIATION_COPY_NONATOMIC );
}

- (ActionBlock)actionBlock{
    return objc_getAssociatedObject(self ,&keyWithBlock);
}

- (void)buttonTapAction:(UIButton *)button {
    //通过key获取被关联对象
    //objc_getAssociatedObject(id object, const void *key)
    void (^tapBlock)(UIButton *) = objc_getAssociatedObject(button, &keyWithMethod);
    
    if (tapBlock) {
        tapBlock(button);
    }
 
    ActionBlock block2 = (ActionBlock)objc_getAssociatedObject(button, &keyWithBlock);
    if(block2){
        block2(button);
    }
}

@end
```

- 第四步，简单的使用
```objective-c
UIButton *button = [UIButton createButtonWithFrame:CGRectMake(20, 100, 200, 200) title:@"带块的button" titleColor:[UIColor redColor] bgImageName:@"" actionBlock:^(UIButton *button) {
        NSString *str = [button titleForState:UIControlStateNormal];
        NSLog(@"%@",str);
}];
[self.view addSubview:button];

UIButton *btn = [UIButton button];
btn.frame = CGRectMake(20, 100, 200, 200);
[btn setTitle:@"1234" forState:UIControlStateNormal];
btn.actionBlock = ^(UIButton *button) {
    NSString *str = [button titleForState:UIControlStateNormal];
    NSLog(@"%@",str);
};
[self.view addSubview:btn];
```
**注意：**由于扩展不能直接添加属性，所以要用运行时来自己添加属性的`get`、`set`方法。

最后，简单的了解一下`objc_setAssociatedObject`，`objc_getAssociatedObject`方法

​	Objective-C有两个扩展机制：`Associative`和`Category`。`Category`用来扩展类方法，`Associative`用于扩展属性。`Associative`机制的原理是把两个对象关联起来，让一个对象成为另外一个对象的一部分。它可以在不修改类的定义的前提下为其对象增加存储空间，这在我们无法访问类的源码时(例如给UILable添加一个`selected`的BOOL属性)是非常有用的。`Associative`基于关键字的，因此我们可以使用不同的关键字为任何对象添加任意多的`Associative`。`Associative`可以保证被关联的对象在对象的整个生命周期都是可用的。`Associative`基于runtime，是运行时里的东西,所以头文件需要引用`#import<objc/runtime.h>`文件。

`Associative`提供了3个方法

```objective-c
- objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
- objc_getAssociatedObject(id object, const void *key)
- objc_removeAssociatedObjects(id object)
```
第一个用于给关联对象赋值：	

`objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)`注意的是返回值类型为Object类型，注意一些不是Object类型的例如：BOOL是结构类型。当碰到这种情况可以考虑通过中间类型来转换，如设置BOOL类型属性的时候可以转换为NSNumber类型，获取的时候再转换成BOOL类型即可。
四个参数分别是：源对象、关键字、关联对象和关联策略。
关键字是一个void类型的指针，例如`static NSString *keyWithMethod = @"keyWithMethod"; //关联对象的key`每一个关联的关键字必须是唯一的。

关联策略是枚举类型，如下：

```objective-c

typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
       	                                    *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};
```

 第二个用于获取关联的对象:

`objc_getAssociatedObject(id object, const void *key)`用于获取关联对象的值。这里需要注意的是返回值类型为Object类型，注意一些不是Object类型的例如：BOOL是结构类型。

第三个用于断开关联的对象：

` objc_removeAssociatedObjects(id object)` 是断开关联，需要注意的是他会断开所有关联，所以不推荐这种方式。需要断开关联的时候使用`objc_setAssociatedObject`函数，传入`nil`值即可。