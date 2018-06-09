---
title: 防止Butoon重复点击的几种方法
date: 2018-06-09 20:37:19
tags: [UIButton]
categories: iOS
---
> 第一种,按钮点击后关闭交互，等几秒之后才允许交互

使用CGD

```
 - (void)clickBtn:(UIButton*)btn{
    btn.enabled = NO;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        btn.enabled = YES;
    });

}
```

使用延时方法,没使用GCD方便，算是一种扩展思路

```
- (void)clickBtn:(UIButton*)btn{
    btn.enabled = NO;
    //点击按钮后先取消之前的操作，再进行需要进行的操作
    [[self class] cancelPreviousPerformRequestsWithTarget:self selector:@selector(allowBtnClicks:) object:btn];
    [self performSelector:@selector(allowBtnClicks:) withObject:btn afterDelay:1.0f];
}

- (void)allowBtnClicks:(UIButton*)btn{
    btn.enabled = YES;
}
```


>第二种，使用runtime为UIButton增加属性

* 1.创建一个UIButton的分类，使用runtime为button增加一个时间间隔属性
* 2.然后重写UIButton的点击方法，使用GCD延时来选择button是否可以交互

分类`.h`文件

```
@interface UIButton (One)
@property (nonatomic, assign) NSTimeInterval acceptEventInterval;//添加点击事件的间隔时间
@end
```
分类`.m`文件

```
#import "UIButton+One.h"
#import <objc/runtime.h>

@implementation UIButton (One)

static const char *UIControl_acceptEventInterval = "UIControl_acceptEventInterval";

// get
- (NSTimeInterval)acceptEventInterval{
    return [objc_getAssociatedObject(self, UIControl_acceptEventInterval) doubleValue];
}

// set
- (void)setAcceptEventInterval:(NSTimeInterval)acceptEventInterval{
    objc_setAssociatedObject(self, UIControl_acceptEventInterval, @(acceptEventInterval), OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

// 重写添加的点击方法
- (void)sendAction:(SEL)action to:(id)target forEvent:(UIEvent *)event{
    [super sendAction:action to:target forEvent:event];
    self.enabled = NO;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(self.acceptEventInterval * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        self.enabled = YES;
    });
}

@end
```


