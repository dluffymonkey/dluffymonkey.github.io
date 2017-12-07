---
title: 移动UITextFeild的光标
date: 2016-11-28 19:06
tags: iOS
categories: iOS Tips
---
有时候如果用没有边框的UITextField的时候，自己在layer上画边框的话，UITextField的光标会直接挨着左边的边框，界面看着不友好，这里可以使用下面的代码向右移动光标的位置：

    //把textField的光标向后移
    //移动的像素
    NSInteger m = 8;
    UIView *paddingView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, m, 40)];
    textF.leftView = paddingView;
    textF.leftViewMode = UITextFieldViewModeAlways;
