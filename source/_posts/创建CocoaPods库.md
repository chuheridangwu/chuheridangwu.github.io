---
title: 创建CocoaPods库
date: 2019-05-27 16:34:37
tags: [CocoaPods]
categories: iOS
---
## 制作github仓库
   在github上创建自己的仓库 
   ```
git add .
git commit -s -m "Initial Commit of Library"
git remote add origin https://github.com/chuheridangwu/swiftTool.git
git push -u https://github.com/chuheridangwu/swiftTool.git
   ```
   <!---more--->
## 支持cocoapods
### 创建spec文件
     
```
 vim swiftTool.podspec
```

### 编辑spec文件
```
Pod::Spec.new do |s|
# 这个是pod 的名称
s.name = 'swiftTool'
# 【这个版本号一定要和后面的tag一样】
s.version = '0.1.2'
s.summary = 'swift常用文件'
#使用arc
s.requires_arc = true
# 项目主页代码
s.homepage = 'https://github.com/chuheridangwu/swiftTool'
# LICENSE信息，这里是Apache;有的是MIT，换成MIT就OK了
s.license = { :type => 'MIT', :file => 'LICENSE' }
s.author = { '<作者名称>' => '<邮箱地址>' }
# 仓库地址
s.source = { :git => 'https://github.com/chuheridangwu/swiftTool.git', :tag => s.version.to_s }
        
s.ios.deployment_target = '8.0'
#设置swift试用版本
s.swift_version = '4.0'
# 上传file地址
s.source_files = 'SwiftTool/Classes/*.{swift,h,m}'
     
#资源文件地址
#  s.resource_bundles = {
     'SwiftTool' => ['SwiftTool/Assets/**/*.png']}
s.resources = "SwiftTool/Assets/SwiftTool.bundle"
# 第三方依赖库，多个的话在后面加逗号隔开
s.dependency 'MBProgressHUD', '~> 1.1.0'
# 强烈建议加上这句，取出警告
s.pod_target_xcconfig = { 'SWIFT_SUPPRESS_WARNINGS' => 'YES' } 
end
```
[cocoapods系列教程---spec文件](https://www.jianshu.com/p/f841e248bc4f)

### 创建tag
```
git tag -m "第一个版本" 版本号 // 创建tag
git push --tags   // 提交所有标签
git push origin master
```
### 注册cocoapods账号
如果没有创建Cocoapods账号, 成功会收到一封邮件，需要点击邮件链接
 ```
 pod trunk register 邮箱 '用户名' --verbose
 ```
###  查看注册信息
可以查看你已经注册的信息，其中包含你的name、email、since、Pods、sessions，其中Pods为你往CocoaPods提交的所有的Pod！
 ```
 pod trunk me
 ```
###  检测podspec文件
 ```
 pod spec lint swiftTool.podspec
 //  去除警告编译
 pod spec lint swiftTool.podspec --allow-warnings
 ```
 
### 上传到cocoapods仓库
```
 pod trunk push swiftTool.podspec
 
 看到这样的界面才算成功
  🎉  Congrats
 🚀  swiftTool (1.0.0) successfully published
 📅  May 27th, 06:11
 🌎  https://cocoapods.org/pods/swiftTool
 👍  Tell your friends!
```
### 搜索新建的库
 ```
 pod search swiftTool
 ```
## 可能出现的问题
### 1、 找不到自己创建的库
如果你使用pod search 发现搜索的库是老的库或者search不到,调用命令
```
1、 pod setup, 
2、 执行: rm ~/Library/Caches/CocoaPods/search_index.json
```
### 2、ERROR | [iOS] unknown: Encountered an unknown error (Could not find a `ios` simulator (valid values: com.apple.coresimulator.simruntime.ios-10-3, com.apple.coresimulator.simruntime.ios-12-1, com.apple.coresimulator.simruntime.ios-8-1, com.apple.coresimulator.simruntime.tvos-12-1, com.apple.coresimulator.simruntime.watchos-5-1). Ensure that Xcode -> Window -> Devices has at least one `ios` simulator listed or otherwise add one.
找不到对应的模拟器,需要升级cocoapods`sudo gem install cocoapods `

### 3、更新开源库[!] Unable to accept duplicate entry for: XXXXX (0.0.1)
主要原因是没有提交tag版本,提交过后需要更改`swiftTool.podspec文件里面的版本号`
```
git push --tags   // 提交所有标签
git push origin master
```

## 参考文档
* [制作Github上的开源库](https://www.jianshu.com/p/5f6d02cf0a31)
* [Cocoapods创建公开库和私有库](https://www.jianshu.com/p/8573d1e8193c)
* [cocoapods系列教程---让自己的开源框架支持cocoapods](https://www.jianshu.com/p/1f70f1176727)
* [【CocoaPods】细说 pod trunk](https://note.leodev.me/2016/04/05/CocoaPods-API-Elaborate-pod-trunk/)
* [CocoaPods API](https://guides.cocoapods.org/terminal/commands.html#group_trunk)