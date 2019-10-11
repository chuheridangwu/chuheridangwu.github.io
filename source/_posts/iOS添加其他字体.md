---
title: iOS添加其他字体样式
date: 2019-04-10 14:40:34
tags: [字体]
categories: iOS
---

# 获取系统的所有字体
```
    for (NSString *familename in [UIFont familyNames]) {
        NSArray *Ary = [UIFont fontNamesForFamilyName:familename];
        for (NSString *name in Ary) {
            NSLog(@"name : --%@",name);
        }
    }
```
<!-- more -->
# 添加字体样式
下载字体样式ttf文件 -> 拷贝到资源文件 Build Phases -> Copy Bundle Resources中 -> 在info.plist文件中添加拓展的字体名称 -> 找到对应的字体名称使用

## 下载字体样式
可以百度字体样式下载，选择自己喜欢的字体进行下载，字体样式文件后缀是.ttf

## 添加到资源文件
将下载的字体文件移动到项目中
![](/images/iOSFont-1.png)

## info.plist中添加字体文件名称
在info.plist文件中新增 一行 Fonts provided by application，在item中填字体文件名称
![](/images/iOSFont-2.png)

## 获取真实的字体名称
文件名称是随时可以更改的，并不能代表字体名称，网上很多方法是让右键点击文件查看 -> 显示简介 ->全名 

注意： 有时候简介里显示的全名也并不一定是字体的真实名称，如果想获取真实的字体名称，遍历字体样式查看真实的，比如图中的字体名称是方正像素12，打印出来的字体名称是FZXS12
![](/images/iOSFont-3.png)

## 常见的字体样式
![](/images/iOSFont-0.png)

## 参考链接
[一文让你彻底了解iOS字体相关知识](https://www.cnblogs.com/dsxniubility/p/4699352.html)