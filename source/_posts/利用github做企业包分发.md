---
title: 利用github做企业包分发
date: 2018-12-27 17:14:53
tags: [企业包]
categories: iOS
---

1. 生成企业包
2. 将文件上传到github仓库
3. 找到IPA包和图片的路径，修改下载链接，找到manifest.plist文件的链接
4. 在网页填写对应的链接
<!--more-->
### 生成企业包
#### 勾选 Include manifest for over-the-air installation,不勾选就需要我们自己创建manifest.plist文件了
![](/images/enterprise.png)
#### 填写对应的下载地址，现在我们还不知道，可以随便写
![](/images/enterprise1.png)
#### 生成的文件
![](/images/enterprise2.png)
### 在github建立仓库上传对应文件
需要的文件manifest.plist 、ipa、57x57、512x512的图片,把对应的文件上传到仓库
![](/images/enterprise3.png)
### 文件对应的下载路径
文件正确的下载路径， 点开仓库对应的ipa -> 找到Download按钮 -> 右键，选择复制链接
![](/images/enterprise4.png)
#### 修改manifest.plist文件对应的下载位置
manifest.plist文件对应的格式，填写正确的下载地址
![](/images/enterprise5.png)
#### manifest.plist链接
点击manifest.plist文件 -> 点击 raw -> 跳转到新的链接，当前链接就是正确的链接
![](/images/enterprise6.png)
![](/images/enterprise7.png)
### 填写正确的IPA下载链接
直接填写manifest.plist文件的地址是不行的，需要在前面加上`itms-services://?action=download-manifest&url=`，测试可以直接使用搭建的博客测试
```
itms-services://?action=download-manifest&url=manifest.plist文件路径
```

    举个网页的栗子
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>应用下载</title>
</head>
<body>
<a href="itms-services://?action=download-manifest&url=manifest.plist文件路径">点击开始安装App</a>
</body>
</html>
```

### 来源
[iOS 如何利用github进行企业应用ipa分发](https://my.oschina.net/u/2473136/blog/1550018)