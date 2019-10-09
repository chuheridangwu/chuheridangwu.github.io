---
title: MAC开启本地服务器
date: 2018-08-02 15:48:37
tags: [apache]
categories: 服务器
---
mac自带有apache,可以直接用命令行打开进行使用

* 开启Apache： sudo apachectl start
* 停止Apache：sudo apachectl stop
* 重启Apache：sudo apachectl restart
 
Apache 启动成功，可以通过[http://127.0.0.1/](http://127.0.0.1/)或者[http://localhost]()打开，如果网页显示`It works!`说明服务器启动成功了,站点的根目录为系统级根目录 `/Library/WebServer/Documents`

`httpd.conf`：是Apache网络服务器软件中重要的一个配置文件，它存储着Apache中许多必不可少的配置信息。最常用的一点，就是向里面添加建站网站信息，可以通过编辑 `/etc/apache2/httpd.conf ` 文件来修改 Apache 配置。[点击了解](https://baike.baidu.com/item/httpd.conf)
<!--more-->
## 更改Apache目录
### 创建文件
在Finder中创建一个`Sqites`文件夹（建哪里都可以），路径是`/Users/用户名`,Mac系统对`Library`路径进行了隐藏，点击前往，按`option`键会出现
### 切换工作目录进行备份
#### 切换工作目录
`cd /etc/apache2`
#### 备份文件
 `sudo cp httpd.conf httpd.conf.bak`
如果更改出现问题，可以使用`sudo cp httpd.conf.bak httpd.conf`命令恢复备份的httpd.conf 文件
 
### 编辑httpd.conf 文件
如果不习惯使用vim编辑器，可以按`Command + Shift + G`调出前往文件夹输入`/etc/apache2`即可看到apache配置文件，用自己熟悉编辑器打开`httpd.conf`就可以按照下面的操作进行更改了，保存的时候需要输入root密码。
#### 进入编辑页面
`sudo vim httpd.conf`
#### 打开php
mac 内置php, 默认是关闭的,找到下图`php7_module`的位置，取消注释打开
![](/images/apache_php.png)
#### 修改文件路径为当前文件路径
按 `i` 进入编辑，搜索`DocumentRoot`,将路径改为当前路径
![](/images/apache_path.png)
#### 添加Indexes
搜索`Options FollowSymLinks`，修改为`Options Indexes FollowSymLinks`
![](/images/apache_intent.png)
#### 添加ServerName
在`#ServerName www.example.com:80` 下添加`ServerName localhost:80`
![](/images/apache_servername.png)
### 切换目录重启apache
#### 切换工作目录
`cd /etc`
#### 拷贝php.ini文件
`sudo cp php.ini.default php.ini`
#### 重启apache
`sudo apachectl -k restart`
重启成功之后，在浏览器输入[http://127.0.0.1/](http://127.0.0.1/)或者[http://localhost]()打开，会将`Sites`文件夹内的目录列出来，如果没有，先清理一下浏览器缓存。
### 其他电脑通过网址进入文件
本地网络的电脑可以通过设置 -> 共享中的地址进入文件夹
![](/images/apache_setting.png)
#### 文件权限
访问文件显示没有权限访问，右击->显示简介里面设置访问权限
![](/images/apache_permission.png)

## 常见问题
### 问题1： 更改本地DNS配置文件 
需要更改我们本的`host`文件， 使用`shift + g` 打开系统搜索框，输入 `/private/etc/hosts` 或者直接使用命令行 `open /etc`打开host文件，把我们的服务器域名对应到回送地址127.0.0.1上，这样就可以直接本地进行域名访问。
文件前3行配置是系统默认配置，我们平时访问的localhost就是通过该文件直接解析成对应的IP地址进行本地访问的.<br>
其中`127.0.0.1`是IPv4标准，`::1`是IPv6标准。
我们平时通过域名访问网址时，都会首先访问这个**本地的hosts文件**来查找IP地址，如果找不到对应的结果才会访问DNS服务器进行DNS解析来进行后续的步骤。

### 问题2： Apache处于开启状态，但是
最近在使用Mac apache 时候发现localhost无法访问服务器，但是$ sudo apachectl start不会报任何错
**问题原因：**由于删除了/private/var/log/apache2文件夹，导致重启电脑后apache无法正常工作。
**解决方法：**创建apache2的文件夹（终端` $ sudo mkdir /private/var/log/apache2`），然后重启apache（终端` $ sudo apachectl restart`），会自动在apache2里面重新生成apache需要的日志，便可正常访问和使用apache服务了。  

## 参考网址
[Mac OS X 启用 Web 服务器](https://www.jianshu.com/p/d006a34a343f)
[解决开启后不能使用的网址](https://91youzhi.com/2018/05/26/ji_yu_apache_da_jian_http_https_zheng_xiang_dai_li_fan_xiang_dai_li_fu_wu_qi/)
[更改apache服务器](https://blog.csdn.net/csdn2314/article/details/81199306)

