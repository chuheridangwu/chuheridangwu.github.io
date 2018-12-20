---
title: AFNetworking源码解读
date: 2018-12-13 20:07:22
tags: [AFNetworking]
categories: iOS
---
&nbsp;&nbsp;&nbsp;&nbsp;一直在用AFNetWorking，没想过去读源码，总觉得里面写的肯定很复杂，很难理解的一些东西，自己把自己限制住了，没去做呢就觉得很难，放弃了提升自己的机会。这种思想很危险，实在要不得。
&nbsp;&nbsp;&nbsp;&nbsp;这几天花了一些时间看AFN的源码，才发现它里面的结构很简单，每个类的作用都很明确，简单易懂，重点的是在里面功能的实现细节。在网上看了几篇关于解读AFN源码的几篇文章，涂耀辉的[AFNetworking到底做了什么？](https://www.jianshu.com/p/856f0e26279d)介绍的最为详细，用几篇日志很详细的剖析了AFN的源码。
&nbsp;&nbsp;&nbsp;&nbsp;读AFN之前首先要动NSURLSession、NSURLSessionTask这个两个类，可以通过[深入了解NSURLSession](https://www.jianshu.com/p/94d214129d4d)来查看。
<!---more--->
##  AFN类之间的作用
**NSURLSession**
- AFURLSessionManager
- AFHTTPSessionManager

**序列化**
- <AFURLRequestSerialization>
 
    - AFHTTPRequestSerializer
    - AFJSONRequestSerializer
    - AFPropertyListRequestSerializer
     
- <AFURLResponseSerialization>
    * AFHTTPResponseSerializer
    * AFJSONResponseSerializer
    * AFXMLParserResponseSerializer
    * AFXMLDocumentResponseSerializer （苹果系统）
    * AFPropertyListResponseSerializer
    * AFImageResponseSerializer
    * AFCompoundResponseSerializer
    
**附加功能**

    * AFSecurityPolicy(安全策略)
    * AFNetworkReachabilityManager(检测网络)
    *  对于iOS UIKit库的扩展(UIKit)
     
在AFN的Github上，作者已经给我们介绍了AFN的主要文件。
* AFHTTPSessionManager 继承自AFURLSessionManager类，是对外的一个接口，内部网络请求都是通过AFURLSessionManager来实现，同时AFURLSessionManager内部实现。
* AFURLRequestSerialization文件 对NSURLRequest的body和Header做一些设置。
* AFURLSessionManager是AFN网络请求的主要类，其他类都是围绕AFN进行设计的，AFURLSessionManager 实现了NSURLSessionDelegate, NSURLSessionTaskDelegate等代理。
* AFURLResponseSerialization文件主要用来处理返回结果的数据解析，包含了各种解析Json、XML、Image等类
* AFNetworkReachabilityManager 关于网络链接的检测
* AFSecurityPolicy 网络安全策略，跟https相关

## AFN的设计
AFN以文件划分功能，感觉大佬就是大佬，AFHTTPSessionManager作为对外的接口，本身不做任何功能处理，AFURLSessionManager做内部功能实现。
AFURLRequestSerialization作为对请求链接和参数的处理，文件中定义AFHTTPRequestSerializer类作为父类，定义NSURLRequest的一些属性，再定义对应的AFJSONRequestSerializer类和AFPropertyListRequestSerializer类。AFURLResponseSerialization文件对请求返回的数据处理也是同样，做了很多内部类，对不同的返回数据做不同的处理。
厉害之处就在于，每个文件都只负责自己的功能，在文件内再创建不同的类做不同的事情，解耦做的好厉害。苹果的设计也是这样。棒

## AFN的技巧
嗯~说是使用技巧，就是我们平时不经常用到的一些方法。
### _cmd
_cmd代表当前方法，比如你想打印当前方法名，可以使用`NSStringFromSelector(_cmd)`

### ？：的简写
```
int a = 10;
正常写法 int b = a ? a : 15;
简写 int b = a ?: 15;
```
### 打印类的属性
description  和 debugDescription 是<NSObject>协议里面的属性

```
- (NSString *)description{
    return [NSString stringWithFormat:@"name=%@,dict=%@",_name,_dict];
}
```
### 利用KVO监听属性变化，利用KVC向对应属性赋值
在AFHTTPRequestSerializer类中，定义跟NSURLRequest一样的属性，通过kvc方法对对应的属性就行赋值
在类初始化的时候，利用kvo监听所有的属性
```
 self.mutableObservedChangedKeyPaths = [NSMutableSet set];
    for (NSString *keyPath in AFHTTPRequestSerializerObservedKeyPaths()) {
        if ([self respondsToSelector:NSSelectorFromString(keyPath)]) {
            [self addObserver:self forKeyPath:keyPath options:NSKeyValueObservingOptionNew context:AFHTTPRequestSerializerObserverContext];
        }
    }
```
如果没有赋值，则移除对应的监听,在dealloc方法中，也要移除所有的监听
```
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(__unused id)object
                        change:(NSDictionary *)change
                       context:(void *)context
{
    if (context == AFHTTPRequestSerializerObserverContext) {
        if ([change[NSKeyValueChangeNewKey] isEqual:[NSNull null]]) {
            [self.mutableObservedChangedKeyPaths removeObject:keyPath];
        } else {
            [self.mutableObservedChangedKeyPaths addObject:keyPath];
        }
    }
}
- (void)dealloc {
    for (NSString *keyPath in AFHTTPRequestSerializerObservedKeyPaths()) {
        if ([self respondsToSelector:NSSelectorFromString(keyPath)]) {
            [self removeObserver:self forKeyPath:keyPath context:AFHTTPRequestSerializerObserverContext];
        }
    }
}
```
定义队形的属性数组，使用dispatch_once只运行一次节省性能
```
static NSArray * AFHTTPRequestSerializerObservedKeyPaths() {
    static NSArray *_AFHTTPRequestSerializerObservedKeyPaths = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _AFHTTPRequestSerializerObservedKeyPaths = @[NSStringFromSelector(@selector(allowsCellularAccess)), NSStringFromSelector(@selector(cachePolicy)), NSStringFromSelector(@selector(HTTPShouldHandleCookies)), NSStringFromSelector(@selector(HTTPShouldUsePipelining)), NSStringFromSelector(@selector(networkServiceType)), NSStringFromSelector(@selector(timeoutInterval))];
    });

    return _AFHTTPRequestSerializerObservedKeyPaths;
}
```
在设置NSMutableURLRequest时，使用kvc方式赋值
```
- (NSMutableURLRequest *)requestWithMethod:(NSString *)method
                                 URLString:(NSString *)URLString
                                parameters:(id)parameters
                                     error:(NSError *__autoreleasing *)error
{
    NSURL *url = [NSURL URLWithString:URLString];
    NSMutableURLRequest *mutableRequest = [[NSMutableURLRequest alloc] initWithURL:url];
    mutableRequest.HTTPMethod = method;

    for (NSString *keyPath in AFHTTPRequestSerializerObservedKeyPaths()) {
        if ([self.mutableObservedChangedKeyPaths containsObject:keyPath]) {
            [mutableRequest setValue:[self valueForKeyPath:keyPath] forKey:keyPath];
        }
    }

    mutableRequest = [[self requestBySerializingRequest:mutableRequest withParameters:parameters error:error] mutableCopy];

	return mutableRequest;
}
```
## 常见的宏

```
DEPRECATED_ATTRIBUTE : 系统提供给我们的一个宏，一般不想别人用的时候在方法后面加上
NSParameterAssert() : 参数断言，里面的值为空不会往下面继续执行
```
## AFN中常见的错误
### Error Domain=NSURLErrorDomain Code=-1200 "An SSL error has occurred and a secure connection to the server cannot be made."

是因为苹果安全策略问题，需要在info.plist文件中增加
```
   <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
    </dict>
```

###  Code=-1016 "Request failed: unacceptable content-type: text/html" UserInfo={NSLocalizedDescription=Request failed: unacceptable content-type: text/html
是因为AFJSONResponseSerializer序列化的时候不支持text/html格式,把格式添加上就可可以
```
self.acceptableContentTypes = [NSSet setWithObjects:@"application/json", @"text/json", @"text/javascript",@"text/html",nil];
```

## 参考网址
[深入了解NSURLSession](https://www.jianshu.com/p/94d214129d4d)
[AFNetworking到底做了什么？](https://www.jianshu.com/p/856f0e26279d)
[AFNetworking的漂亮细节](https://www.jianshu.com/p/ddf79f0763a7)
如果你对do{...}while(0)为什么用在宏上面不理解，可以查看[宏定义的黑魔法](https://onevcat.com/2014/01/black-magic-in-macro/)
如果
