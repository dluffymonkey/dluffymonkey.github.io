#### Xib加载控制器的时候,出现的一个坑

今天在公司的项目中,iOS8出现了push到一个Xib的控制器时,出现crash的现象,直接蹦到了main函数,错误信息:libc++abi.dylib: terminate_handler unexpectedly threw an exception.当实例化这个控制器的时候,用的是

```objective-c
LQFundBrowseViewController *fundBrowseVC = [[LQFundBrowseViewController alloc]init];
```

后来实例化控制器的方法修改成了

```objective-c
LQFundBrowseViewController *fundBrowseVC = [[LQFundBrowseViewController alloc]initWithNibName:@"LQFundBrowseViewController" bundle:[NSBundle mainBundle]];
```

就没了这个crash. 
按道理说,alloc init方法还是会调用initWithNibName的方法,但是如果你创建控制器的时候,用到了xib,而且后来实例化的时候,没有指定xib的名称,**并且项目中出现了一个同名的自定义的view(LQFundBrowseView,上面是LQFundBrowseViewController,这就叫同名)**那么系统就会优先匹配这个同名的view作为控制器的view,尽管你在xib里面做了view的关联, 
解决方法: 
1:new file控制器的时候,命名把viewContoller直接简写成VC(eg:把LQFundBrowseViewController命名为LQFundBrowseVC) 
2:使用指定xib名称的实例化方法,即上面的第二种方法

但是为什么在iOS9/iOS10里面就没有出现这种情况呢? 