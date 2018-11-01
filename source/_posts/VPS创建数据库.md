---
title: 搬瓦工centos7.0创建数据库
date: 2018-08-07 11:24:09
tags: [数据库]
categories: 数据库
---

鉴于国内的搜索，就搞了个搬瓦工用来翻墙，只用来搜索有些浪费，想搞点其他的东西，先建个数据库用python爬一些数据存进去，自己写接口玩吧，顺便学习一下后台的一些知识！

## 创建
使用yum命令安装mysql，yum 是一个在Fedora和RedHat以及SUSE中的Shell前端软件包管理器。基於RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。yum提供了查找、安装、删除某一个、一组甚至全部软件包的命令，命令简洁而又好记

### 安装
输入`yum install -y mysql-server mysql mysql-devel`命令，将`mysql` `mysql-server` `mysql-devel`都安装好(**注意:安装mysql时我们并不是安装了mysql客户端就相当于安装好了mysql数据库了，我们还需要安装mysql-server服务端才行**)
<!--more-->
### 启动数据库
`service mysqld start`

### 设置开机启动
`chkconfig mysqld on`
查看是否开启开机启动
`chkconfig --list | grep mysql`

### 设置密码
数据库安装完后默认有一个**没有密码**的root管理员账号，需要自己设置
`SET PASSWORD FOR root=PASSWORD("密码"); `

### 登录
`mysql -u root -p`

## 遇到的问题
### 使用Navicat Premium软件无法进行远程连接，报错Host xxxx is not allowed to connect to this MySQL server
第一，先检查接口是否正确，mysql默认的端口是3306。
第二，看是否打开权限，可以选择该表或者授权
首先连接mysql

```
[root@host]# mysql -u root -p
Enter password:******
```
#### 授权，允许外网访问

```
GRANT ALL PRIVILEGES ON *.* TO ‘myuser’@'%’ IDENTIFIED BY ‘mypassword’ WITH GRANT OPTION;
```
`myuser`: 用户名
`mypassword`: 密码

#### 改表

```
mysql>use mysql
mysql>GRANT SELECT,INSERT,UPDATE,DELETE ON [db_name].* TO [username]@[ipadd] identified by '[password]';
```
[username]:远程登入的使用者代码
［db_name]:表示欲开放给使用者的数据库称
[password]:远程登入的使用者密码
[ipadd]:IP地址或者IP反查后的DNS Name，此例的内容需填入'60-248-32-13.HINET-IP.hinet.net' ，包函上引号(')
（其实就是在远端服务器上执行，地址填写本地主机的ip地址。）

也可以写

```
mysql>use mysql;
mysql>update user set host = ‘%’ where user = ‘root’;mysql>select host, user from user;
```

## 参考网址
[CentOs 下安装Mysql数据库](https://www.jianshu.com/p/674f56b2fe56)
[python 使用ssh隧道连接mysql](https://blog.imdst.com/python-shi-yong-sshsui-dao-lian-jie-mysql/)


