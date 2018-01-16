---
title: iOS给UIView设置渐变背景色
date: 2018-01-17 16:12
tags: iOS
categories: iOS Tips
---

#### iOS给UIView设置渐变背景色

```objective-c
//UIView设置渐变色
- (void)setupGradientWithFrame:(CGRect)rect {
    UIColor *startColor = UIColorFromRGB(0xFC864A);
    UIColor *endColor = UIColorFromRGB(0xF85E45);
    NSArray *colors = @[(id)startColor.CGColor, (id)endColor.CGColor];
    NSArray *locations = @[@(0.1),@(0.7)];
    CAGradientLayer *gradientLayer = [CAGradientLayer layer];
    gradientLayer.colors = colors;
    gradientLayer.locations = locations;
    gradientLayer.startPoint = CGPointMake(0, 0);
    gradientLayer.endPoint = CGPointMake(0, 1);
    gradientLayer.frame = rect;
    [self.layer addSublayer:gradientLayer];
}
```
