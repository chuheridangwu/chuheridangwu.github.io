---
title: NSFileManager常用方法
date: 2018-12-25 11:29:06
tags: [文件夹]
categories: iOS
---

&nbsp;&nbsp;&nbsp;&nbsp;学习的过程中总觉得只靠死记硬背的做法没有意义，用过一段时间之后还是会忘掉，又需要重新记忆，没有一种方式能把知识链接起来，很是麻烦。
最近在思考一种新的学习方法，感觉比死记要好一点。学习一个新的知识，比如说这个类NSFileManager文件管理，这个类很明确，就是用来管理文件的。进一步想，如果是我们自己去设计这个类，会去怎么设计，文件都需要怎么管理，要有哪些功能？
&nbsp;&nbsp;&nbsp;&nbsp;既然是文件，那么创建、移动、复制、删除、查询、区分是文件夹还是文件、两个文件是否相同、获取文件夹里面的子文件、判断是否是空的文件夹、读取写入文件内容、文件名称排序、 这些方法肯定是要有的。文件要有什么属性？路径、大小、创建时间,嗯~ 这是我刚刚能想到的，就这么多
<!-- more-->
&nbsp;&nbsp;&nbsp;&nbsp;来我们看看苹果提供给我们使用的方法都有哪些内容。它对NSFileManager这个类都有哪些功能和怎么设计的?
苹果除了在功能上除了前面想到的基础功能之外，增加了iCloud项目，和访问文件权限的功能、符号和硬链接（我也不懂这是什么！），并且在功能的使用上，提供的方法比较详细，基本覆盖了日常各种姿势的使用。
&nbsp;&nbsp;&nbsp;&nbsp;在设计上，基础功能使用NSFileManager类来实现，用代理来实现对文件操作动作的监听，设计NSDirectoryEnumerator类用来遍历当前目录和子目录，应用和文件扩展程之间的通讯使用NSFileProviderService，文件的属性使用字典的分类来实现。建立了一个NSPathUtilities文件对路径做常用处理，解耦很彻底。大写的服气
```
NSFileManagerDelegate: 用来处理移动、复制、删除、链接文件时发生的问题
NSDirectoryEnumerator: 递归遍历所有目录及子目录,获取对应文件的属性和层级
NSFileProviderService: 应用程序和文件之间的自定义通讯通道，没用过
NSFileAttributes: 一个字典，包含文件的各种信息，大小、创建时间、最近打开的时间等等
```
个人感觉写代码不是要死记当前类的方法，更多的是一种设计上的处理，一种逻辑上的处理，比如这个类要有什么方法，就是基于当前这个类的作用自然延伸的。不需要去死记，里面的一些特殊的方法也是基于某些场景所给的。自然而然。还在理解学习方式中。

## NSFileManagerDelegate
NSFileManager的代理，里面有16个方法,处理移动、复制、删除、链接,包含正常处理或者处理的时候出错了是否继续的方法。
![](/images/NSFileManager.png)

## 常用方法

`-` (nullable NSDirectoryEnumerator<NSString *> *)enumeratorAtPath:(NSString *)path; 
 递归遍历目录及下面的子目录
`-` (nullable NSArray<NSString *> *)contentsOfDirectoryAtPath:(NSString *)path error:(NSError **)error  
只获取当前目录下的文件
```
假设有A文件，A有B、C文件，B、C里有D文件
        NSDirectoryEnumerator *enumDic = [manager enumeratorAtPath:@"/Users/mlive/Desktop/A"];
        打印enumDic.allObjects [c,c/d,b,b/d]
        NSArray *dir = [manager contentsOfDirectoryAtPath:@"/Users/mlive/Desktop/A" error:nil];
        打印dir [c,b]
      // NSDirectoryEnumerator类有skipDescendents方法和level属性，如果想知道当前文件在第几层可以使用level，如果在遍历过程中想跳过某个文件，可以使用skipDescendents方法
```


`-` (BOOL)moveItemAtPath:(NSString *)srcPath toPath:(NSString *)dstPath error:(NSError **)error;
将指定路径上的文件或目录同步移动到新位置。需要注意的是：dstPath路径必须要包含文件或目录的名称，不包含的话会报Domain=NSCocoaErrorDomain Code=516 
```
    NSFileManager *manager = [NSFileManager defaultManager];
    NSError *error;
    BOOL isMove =  [manager moveItemAtPath:@"/Users/mlive/Desktop/srcPath" toPath:@"/Users/mlive/Desktop/dstPath/src"  error:&error];
```


`-` (BOOL)fileExistsAtPath:(NSString *)path isDirectory:(BOOL *)isDirectory;
path: 判断对应的路径是否有文件
isDirectory: 判断是文件还是文件夹 YES: 文件夹 NO： 文件
```
        NSString *library = [NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES) lastObject];
        // 判断是文件夹还是文件 YES: 文件夹 NO： 文件
        BOOL isDirectory;
        BOOL isExist = [manager fileExistsAtPath:doc isDirectory:&isDirectory];
        if (!isExist) {
            NSLog(@"没有找到对应的文件路径 ");
            return;
        }
```

## 扩展  -- 关于路径的处理
对路径的处理，iOS专门建立了一个NSPathUtilities文件来进行扩展，了解一下这个类的方法，处理路径能省出不少力气

```
// 数组转路径
NSArray *components = @[@"users",@"test",@"Desktop"];
NSString *path = [NSString pathWithComponents:components];
打印：users/test/Desktop

假设当前路径：@"/Users/mlive/Desktop/text.txt";
pathComponents: 路径转数组
absolutePath: 判断是否为绝对路径(依据：是否以'/'开始)  
lastPathComponent: 最后一个目录  text.txt
stringByDeletingLastPathComponent: 删除最后一个目录 /Users/mlive/Desktop
stringByAppendingPathComponent: 拼接一个目录
pathExtension: 获取扩展名 txt
stringByDeletingPathExtension: 删除扩展名， .也会删掉
stringByAppendingPathExtension: 添加扩展名
```

`-` (NSArray<NSString *> *)stringsByAppendingPaths:(NSArray<NSString *> *)paths;
给数组内所有的路径加上Users
```
        NSArray *paths = @[@"mlie/test",@"mlive/Desktop"];
        NSArray *resAry = [@"Users" stringsByAppendingPaths:paths];
```

`-` (NSArray<NSString *> *)pathsMatchingExtensions:(NSArray<NSString *> *)filterTypes;
获取对应类型的文件
```
        筛选出mp3格式和avi格式的文件
        NSArray *sourceAry = @[@"1.mp3",@"2.mp4",@"3.avi",@"4.avi",@"5.avi"];
        NSArray *resAry = [sourceAry pathsMatchingExtensions:@[@"mp3",@"avi"]];
```

获取常用到的文件夹路径,Document、Library、等等，一些路径系统帮我们创建好了方法，直接调用就可以了。
NSSearchPathDirectory: 哪一个文件目录位置，常用的NSLibraryDirectory、NSDocumentDirectory
NSSearchPathDomainMask: 搜索重要目录要使用的位置，常用的 NSUserDomainMask
```
NSString *document = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
NSString *library = [NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES) lastObject];
NSString *canche = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
NSString *temp = NSTemporaryDirectory();
NSString *home = NSHomeDirectory();
对应的文件路径：
document
/Users/test/Library/Developer/CoreSimulator/Devices/891711ED-A7B6-4933-B872-6CE554D186CC/data/Containers/Data/Application/6572C717-9790-4528-8977-9C6F6E3D1C65/Documents
library
/Users/test/Library/Developer/CoreSimulator/Devices/891711ED-A7B6-4933-B872-6CE554D186CC/data/Containers/Data/Application/6572C717-9790-4528-8977-9C6F6E3D1C65/Library
canche
/Users/test/Library/Developer/CoreSimulator/Devices/891711ED-A7B6-4933-B872-6CE554D186CC/data/Containers/Data/Application/6572C717-9790-4528-8977-9C6F6E3D1C65/Library/Caches
temp
/Users/test/Library/Developer/CoreSimulator/Devices/891711ED-A7B6-4933-B872-6CE554D186CC/data/Containers/Data/Application/6572C717-9790-4528-8977-9C6F6E3D1C65/tmp/
home
/Users/test/Library/Developer/CoreSimulator/Devices/891711ED-A7B6-4933-B872-6CE554D186CC/data/Containers/Data/Application/6572C717-9790-4528-8977-9C6F6E3D1C65    
 
Documents（NSDocumentDirectory）　　//用于写入应用相关数据文件的目录，在ios中写入这里的文件能够与iTunes共享并访问，存储在这里的文件会自动备份到云端

Library/Caches（NSCachesDirectory）　　//用于写入应用支持文件的目录，保存应用程序再次启动需要的信息。iTunes不会对这个目录的内容进行备份

tmp（use NSTemporaryDirectory（））　　//这个目录用于存放临时文件，只程序终止时需要移除这些文件，当应用程序不再需要这些临时文件时，应该将其从这个目录中删除

Library/Preferences　　//这个目录包含应用程序的偏好设置文件，使用 NSUserDefault类进行偏好设置文件的创建、读取和修改  
```

## 参考文档
[常见的宏定义](https://www.jianshu.com/p/d4110f582269)
