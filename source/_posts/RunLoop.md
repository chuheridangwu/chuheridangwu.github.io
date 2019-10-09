---
title: RunLoop
date: 2018-11-01 11:25:58
tags: [RunLoop]
categories: iOS
---

RunLoop可以通过NSRunLoop和CFRunLoopRef来访问
* RunLoop本质是一个do-while循环，在循环内部处理(Source、Timer、Observer)
* 一个线程对应一个RunLoop,主线程的RunLoop默认启动，子线程的需要手动调用run方法启动，并且保证RunLoop含有CFRunLoopSourceRef、CFRunLoopTimerRef其中一个才能运行
* RunLoop只能选择一种Mode启动，如果当前Mode中不包含Source(Source0、Source1)或者Timer，会直接退出RunLoop
* NSRunLoop是CFRunLoopRef基于OC的一层包装，如果要了解RunLoop内部结构，需要研究CFRunLoopRef层面的Api
<!-- more-->

## RunLoop跟线程的关系
* 每一个线程都有一个RunLoop对象
* 主线程的RunLoop对象已经创建好了，子线程的需要自己创建
* RunLoop在第一次获取时创建，在线程结束时销毁

## RunLoop相关的类
### CFRunLoopRef
NSRunLoop类
### CFRunLoopModeRef
* RunLoop的运行模式
* 一个RunLoop包含若干个Mode,每个Mode又包含若干个Source/Timer/Observer
* 每次RunLoop启动时，只能指定其中一个Mode,获取当前Runloop的Mode类型可以使用currentMode方法

#### RunLoopMode的几种类型
* kCFRunLoopDefaultMode: App的默认Mode,通常主线程是在这个Mode下运行
* UITrackingRunLoopMode: 界面追踪Mode,用于ScrollView追踪触摸滑动，保证界面滑动时不受其他Mode影响
* kCFRunLoopCommonModes: 是苹果占位用的Mode,并不是一个真正的Mode，kCFRunLoopCommonModes包含以下mode类型,可以通过打印查看
![](/images/RunLoop_2.png)

* GSEventReceiveRunLoopMode: 接受系统事件的内部Mode
* UIInitializationRunLoopMode：刚启动App时进入的第一个Mode

### CFRunLoopSourceRef
事件源(输入源)，分为Source0和Source1
Source0: 非基于Port端口的
Source1: 基于Port端口，通过内核和其他线程通信，接受、分发系统事件

### CFRunLoopTimerRef
基于时间的触发器，常见的NSTimer

scheduledTimerWithTimeInterval()：方法默认是加入当前Runloop中并且是NSDefaultRunLoopMode模式，如果想让定时器在UITrackingRunLoopMode下继续使用，需要更改它的Mode
```
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;
```

### CFRunLoopObserverRef
RunLoop的观察者，能够观察到RunLoop的状态改变

```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0),  // 1 即将进入RunLoop
    kCFRunLoopBeforeTimers = (1UL << 1), // 2  即将处理Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 4 即将处理Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 32 即将进入睡眠
    kCFRunLoopAfterWaiting = (1UL << 6), // 64 刚从睡眠中唤醒
    kCFRunLoopExit = (1UL << 7), // 128 即将退出RunLoop
    kCFRunLoopAllActivities = 0x0FFFFFFFU
};
```
#### CFRunLoopObserverRef的常用方法
* CFRunLoopObserverRef的创建和移除

 ```
 CF_EXPORT CFRunLoopObserverRef CFRunLoopObserverCreate(CFAllocatorRef allocator, CFOptionFlags activities, Boolean repeats, CFIndex order, CFRunLoopObserverCallBack callout, CFRunLoopObserverContext *context);
 CF_EXPORT CFRunLoopObserverRef CFRunLoopObserverCreateWithHandler(CFAllocatorRef allocator, CFOptionFlags activities, Boolean repeats, CFIndex order, void (^block) (CFRunLoopObserverRef observer, CFRunLoopActivity activity))
CF_EXPORT void CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFRunLoopMode mode);
```
* 举个栗子

    ```
    CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(),kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
        
    });
    CFRunLoopAddObserver(CFRunLoopGetMain(), observer, kCFRunLoopDefaultMode);
    CFRunLoopRemoveObserver(CFRunLoopGetMain(), observer, kCFRunLoopDefaultMode);
    //CoreFoundation框架中，通过Copy、Retain、Create 创建出来的对象都要主动释放掉
    CFRelease(observer);
    ```
### RunLoop类之间的关系
![](/images/RunLoop_0.png)
## RunLoop的内部逻辑
官方文档的图
![](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Art/runloop.jpg)
网友进行整理的逻辑图
![RunLoop_1](/images/RunLoop_1.png)
如果想通过源码看具体的内部细节，可以通过[下载源码](https://opensource.apple.com/release/os-x-10105.html)查看_CFRunLoopRun()方法查看具体实现细节。


## RunLoop源码相关
从[苹果开源网址](https://opensource.apple.com/)中下载资源，因为RunLoop是CoreFoundation框架进行的封装，我们找到对应的CF文件[进行下载](https://opensource.apple.com/release/os-x-10105.html)
```
// 获取当前RunLoop对象
CF_EXPORT CFRunLoopRef CFRunLoopGetCurrent(void);

// 获取主线程RunLoop对象
CF_EXPORT CFRunLoopRef CFRunLoopGetMain(void);
```

RunLoop对象跟线程的关系，具体可以根据源码
```
// 获取当前RunLoop对象，调用_CFRunLoopGet0()方法
CFRunLoopRef CFRunLoopGetCurrent(void) {
    CHECK_FOR_FORK();
    CFRunLoopRef rl = (CFRunLoopRef)_CFGetTSD(__CFTSDKeyRunLoop);
    if (rl) return rl;
    return _CFRunLoopGet0(pthread_self());
}

// should only be called by Foundation
// t==0 is a synonym for "main thread" that always works
// _CFRunLoopGet0()方法的具体实现
CF_EXPORT CFRunLoopRef _CFRunLoopGet0(pthread_t t) {
    if (pthread_equal(t, kNilPthreadT)) {
	t = pthread_main_thread_np();
    }
    __CFLock(&loopsLock);
    if (!__CFRunLoops) {
        __CFUnlock(&loopsLock);
	CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
	CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
	CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
	if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
	    CFRelease(dict);
	}
	CFRelease(mainLoop);
        __CFLock(&loopsLock);
    }
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFUnlock(&loopsLock);
    if (!loop) {
	CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFLock(&loopsLock);
	loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
	if (!loop) {
	    CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
	    loop = newLoop;
	}
        // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFUnlock(&loopsLock);
	CFRelease(newLoop);
    }
    if (pthread_equal(t, pthread_self())) {
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
        }
    }
    return loop;
}
```

## RunLoop在程序中的使用技巧
* 开启一个常驻线程(让一个子线程不进入消亡状态，等待其他线程发来消息，处理其他时间)
    * 在子线程中开启一个定时器
    * 在子线程中进行一些长期监控
* 可以控制定时器在特定模式下执行
* 可以让某些事件(行为、任务)在特定模式下执行
* 可以添加Observer监听RunLoo状态，比如主线程卡顿 
 
### 在对应的Mode实现相应的功能
举个栗子：比如想在视图滚动的时候做一些事情,可以使用UITrackingRunLoopMode来做些事情
```
[_imgView performSelector:@selector(setImage:) withObject:[UIImage imageNamed:@"RunLoop_0.png"] afterDelay:0 inModes:@[UITrackingRunLoopMode]];
```

### RunLoop开启一个常驻异步线程
每个线程里面都会有一个RunLoop对象，默认是没有运行的，异步线程执行完成之后就会死亡，如果想要一个常驻的异步线程，需要保证里面的RunLoop存活，RunLoop存活的条件是需要包含一个Source或者Timer
```
- (void)viewDidLoad {
    [super viewDidLoad];
    // 创建一个子线程
    _thread = [[XBThread alloc]initWithTarget:self selector:@selector(runThread) object:nil];
    [_thread start];
}

- (void)runThread{
    // RunLoop添加Port并且运行，保持子线程不死
    @autoreleasepool {
        [[NSRunLoop currentRunLoop] addPort:[NSPort port] forMode:NSDefaultRunLoopMode];
        [[NSRunLoop currentRunLoop] run];    
        }
}
```
另一种保持线程不死的方法，当RunLoop里面有资源之后，[[NSRunLoop currentRunLoop] run]方法，运行之后，run方法后面的代码就不会执行了。
```
- (void)viewDidLoad {
    [super viewDidLoad];
    // 创建一个子线程
    _thread = [[XBThread alloc]initWithTarget:self selector:@selector(runThread) object:nil];
    [_thread start];
}

- (void)runThread{
    while (1) {
        [[NSRunLoop currentRunLoop] run];
        // 点击屏幕之后这个log就不会打印了
        NSLog(@"*****************");
     }
}

- (void)runTimer{
   NSLog(@"************runTimer***************%@ ***",[NSThread currentThread]);
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
        [self performSelector:@selector(runTimer2) onThread:_thread withObject:nil waitUntilDone:NO];
}
```

### 自动释放池
当池子被释放的时候，让池子里面所有的对象调用release方法
* 进入RunLoop时创建自动释放池
* 在RunLoop睡眠之前释放
* 在RunLoop退出的时候释放

验证RunLoop在什么时候释放自动释放池，打印当前RunLoop，_wrapRunLoopWithAutoreleasePoolHandler是当前释放池，根据CFRunLoopObserver监听RunLoop的状态，在进入RunLoop之前kCFRunLoopEntry创建自动释放池，即将睡眠之前kCFRunLoopBeforeWaiting，和退出kCFRunLoopExit的时候进行销毁自动释放池，验证方法，activities的值 = 0x1 和0xa0;
![-w1213](/images/RunLoop_3.png)

### 检测主线程卡顿
使用Observer监听主线程的状态，创建一个异步线程，并保持存活，使用dispatch_semaphore_t做线程同步，在异步线程监听主线程状态是否在kCFRunLoopBeforeSources、kCFRunLoopAfterWaiting中，如果重复多次都在则代表主线程卡顿，使用[BSBacktraceLogger](https://github.com/bestswifter/BSBacktraceLogger)可以在异步线程获取到主线程堆栈的调用信息
利用[RunLoop检测主线程卡顿](https://juejin.im/post/59edb7596fb9a0450d103f34)出现的问题

####  dispatch_semaphore_t相关信息 
[浅谈dispatch_semaphore_t](https://www.jianshu.com/p/c815ad360a2a)

```
// 创建信号
dispatch_semaphore_t dispatch_semaphore_create(long value);
// 等待资源释放，如果传入的dsema大于0，就继续向下执行，并将信号量减1；如果dsema等于0，则等待超时才会继续向下执行。
long dispatch_semaphore_wait(dispatch_semaphore_t dsema, dispatch_time_t timeout);
// 发送信号
long dispatch_semaphore_signal(dispatch_semaphore_t dsema);
```

## RunLoop相关文档
[RunLoop的官方文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW1)
[CFRunLoopRef开源文档  CF框架](https://opensource.apple.com/release/os-x-10105.html)
[深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)
