---
title: NSObject对象占用多少内存
date: 2019-10-09 13:51:48
categories: oc
---

# 一个NSObject对象占多少内存
1. 转换成C++代码看NSObject对象本质
2. 使用 runtime 和 malloc查看内存大小
3. 利用苹果源码查看内存分配
4. 使用xcode工具进行调试
5. 利用LLDB 查看/更改内存的值
6. 查看Student对象的内存大小 （拓展问题）

<!--more-->

## 转成C++ 看oc对象的本质
 
### 代码编译过程
 
Objective-c ->  C/C++  -> 汇编语言 ->机器语言

### oc转c++命令

`clang -rewrite-objc main.m -o main.cpp`

不同平台支持的代码不一样，只转跑在iOS平台上的C++代码

```
xcrun -sdk iphoneos12.4 clang -arch arm64  -rewrite-objc main.m -o main.cpp

xcrun: xcode工具运行 

-sdk -iphoneos12.4 : iphoneos12.4的sdk 

-arch : 架构 (模拟器:i386 /32位:armv7 /64位:arm64)
```

**报错**
1. 报错 xcodebuild: error: SDK "iphones" cannot be located.
 
原因是xcode命令行找不到 iphones的sdk
xcodebuild -showsdks : 查看所有可用sdk
xcrun -sdk iphoneos12.4 clang -arch arm64 -rewrite-objc main.m -o main.cpp

### NSObject 转成 C++ 形式代码
```
// NSObject Implementation  结构体的内存地址是它第一个成员的内存地址
struct NSObject_IMPL {
	Class isa;
};

// 指向结构体的指针 （指针占用多少字节 64位：8个字节  32位：4个字节） 
typedef struct objc_class *Class; 

// NSObject 定义
@interface NSObject <NSObject> {
    Class isa  ;
}
```

## 利用runtime 和  malloc 查看内存大小

* runtime

```
#import <objc/runtime.h> 

class_getInstanceSize([NSObject class]) //获取实例对象的内存大小
// 获取NSObject类实例对象的成员变量所占用的大小
NSLog(@"%zd",class_getInstanceSize([NSObject class]));

```
* malloc

```
// 获取obj指针指向内存的大小
NSLog(@"%zd",malloc_size((__bridge const void*)obj));
```

* 看苹果源码
 
苹果源码地址： https://opensource.apple.com/tarballs/
搜索 objc,下载runtime源码

* class_getInstanceSize() 相关源码
 
```
size_t class_getInstanceSize(Class cls)
{
    if (!cls) return 0;
    return cls->alignedInstanceSize();
}

 // 返回类成员变量的大小
uint32_t alignedInstanceSize() {
     return word_align(unalignedInstanceSize());
   	}
   	
```

**也就是说，分配给NSObject对象16个字节，它真正使用的只有8个字节。**

为什么确定是16个，查看alloc相关源码

* alloc 相关源码 内存分配最后都要调用 allocWithZone() ,搜索allocWithZone

```
id
_objc_rootAllocWithZone(Class cls, malloc_zone_t *zone)
{
    id obj;

#if __OBJC2__
    // allocWithZone under __OBJC2__ ignores the zone parameter
    (void)zone;
    obj = class_createInstance(cls, 0);
#else
    if (!zone) {
        obj = class_createInstance(cls, 0);
    }
    else {
        obj = class_createInstanceFromZone(cls, 0, zone);
    }
#endif

    if (slowpath(!obj)) obj = callBadAllocHandler(cls);
    return obj;
}



id 
class_createInstance(Class cls, size_t extraBytes)
{
    return _class_createInstanceFromZone(cls, extraBytes, nil);
}



id
_class_createInstanceFromZone(Class cls, size_t extraBytes, void *zone, 
                              bool cxxConstruct = true, 
                              size_t *outAllocatedSize = nil)
{
    if (!cls) return nil;

    assert(cls->isRealized());

    // Read class's info bits all at once for performance
    bool hasCxxCtor = cls->hasCxxCtor();
    bool hasCxxDtor = cls->hasCxxDtor();
    bool fast = cls->canAllocNonpointer();

    size_t size = cls->instanceSize(extraBytes);
    if (outAllocatedSize) *outAllocatedSize = size;

    id obj;
    if (!zone  &&  fast) {
        obj = (id)calloc(1, size);
        if (!obj) return nil;
        obj->initInstanceIsa(cls, hasCxxDtor);
    } 
    else {
        if (zone) {
            obj = (id)malloc_zone_calloc ((malloc_zone_t *)zone, 1, size);
        } else {
            obj = (id)calloc(1, size);
        }
        if (!obj) return nil;

        // Use raw pointer isa on the assumption that they might be 
        // doing something weird with the zone or RR.
        obj->initIsa(cls);
    }

    if (cxxConstruct && hasCxxCtor) {
        obj = _objc_constructOrFree(obj, cls);
    }

    return obj;
}

// 分配内存
  size_t instanceSize(size_t extraBytes) {
        size_t size = alignedInstanceSize() + extraBytes;
        // CF requires all objects be at least 16 bytes.
        if (size < 16) size = 16;
        return size;
    }


```

## 通过Xcode调试看内存地址
Debug -> Debug Workflow -> View Memory 
 
 * 一个字节 = 8个二进制位
 * 一个16进制位 = 4个二进制位 （2x2x2x2）
 * 两个16进制位 = 一个字节
 
 
### 使用 memory read 读取内存地址

```
 memory read 数量格式字节数 内存地址
 memory read 0x100505bc0

 memory write 内存地址 更改的值
 memory write 0x100505bc8 9
 更改 0x100505bc0 第8位的值
 
 memory read 可以用x 表示
 x/数量格式字节数 内存地址
 x/3xw 0x10010
 
 格式
 x : 16进制
 f : 浮点
 d : 10进制
 
```
 
 **字节大小**
 
 * b : byte 1字节
 * h : half word 2字节
 * w : word 4字节
 * g : giant word 8字节

## 扩展 

### Student对象占用对用字节

```
@interface Student : NSObject
{
    @public
    int _age;
    int _no;
}
@end

@implementation Student
@end
```
先转成C++看Student本质

```
struct NSObject_IMPL {
	Class isa;
};

struct Student_IMPL {
	struct NSObject_IMPL NSObject_IVARS;
	int _age;
	int _no;
};
```
Student强转Student_IMPL结构体，证明Student就是Student_IMPL结构体


```
struct Student_IMPL *si = (__bridge  struct Student_IMPL*)s;
NSLog(@"age: %d no:%d",si->_age,si->_no);
```

## 概念
* 内存对齐： 结构体的最终大小必须是最大成员的倍数。(只是内存对齐概念的其中之一)

* Clang: Clang是一个C语言、C++、Objective-C语言的轻量级编译器

* malloc： malloc的全称是memory allocation，中文叫动态内存分配，用于申请一块连续的指定大小的内存块区域以void*类型返回分配的内存区域地址，当无法知道内存具体位置的时候，想要绑定真正的内存空间，就需要用到动态的分配内存，且分配的大小就是程序要求的大小。
* calloc: 在内存的动态存储区中分配n个长度为size的连续空间，函数返回一个指向分配起始地址的指针；如果分配不成功，返回NULL。
