\- (void)setupGradientWithFrame:(CGRect)rect {

​    UIColor *startColor = UIColorFromRGB(0xFC864A);

​    UIColor *endColor = UIColorFromRGB(0xF85E45);

​    NSArray *colors = @[(id)startColor.CGColor, (id)endColor.CGColor];

​    NSArray *locations = @[@(0.1),@(0.7)];

​    CAGradientLayer *gradientLayer = [CAGradientLayer layer];

​    gradientLayer.colors = colors;

​    gradientLayer.locations = locations;

​    gradientLayer.startPoint = CGPointMake(0, 0);

​    gradientLayer.endPoint = CGPointMake(0, 1);

​    gradientLayer.frame = rect;

​    [self.layer addSublayer:gradientLayer];

}