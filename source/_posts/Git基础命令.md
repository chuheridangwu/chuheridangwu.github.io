---
title: Git基础命令
date: 2018-04-09 10:10:54
tags: [Git]
categories: 网络工具
---

### 初始化

```
git init                    //初始化仓库
git add README.md           //添加描述文件
git add .                   //添加新增文件 .为添加所有新增文件，添加具体某个文件可以将.换为文件名
git commit -m“第一次提交”    //提交到本地 ""里为描述
git remote add origin https://github.com/chuheridangwu/new.git  //添加远程仓库地址
git push -u origin master   //推送到远程仓库 master:分支名称master一般为主分支
```

<!---more--->

### 查看版本及回退
Git保存的是每一次的修改记录
`git add`:将当前修改添加到暂存区，
`git commit`:将暂存区的所有内容提交到当前分支

```
git status  //查看当前状态
git log     //查看提交日志
git reflog  //查看提交过的命令
git reset --hard b957a5f    //hard:强制回退  b957a5f:提交过的版本号
git checkout --file         //撤销文件内的修改过 file为文件
git rm file //删除文件，如果文件没有提交到版本库，使用rm file 就可以， file: 文件名
```

### 分支管理
**HEAD**指向的永远是当前分支

```
git branch dev      //创建dev分支 dev: 分支名称
git checkout dev    //切换到dev分支
git checkout -b dev //创建并切换到dev分支，相当于一次执行上面两个命令
git branch          //查看当前分支
git branch -d dev   //删除dev分支
git merge dev       //将dev分支跟主分支进行合并
```
>当分支合并发生冲突时，使用`git status`可以告诉我们冲突的文件，Git的用`<<<<<<<，=======，>>>>>>>`标记出不同分支的内容，我们需要修改后重新合并

### 文件管理
```
git mv a.text b.text //更改文件名称 a.text: 原来的文件名 b.text: 更改的文件名
git mv a.text mydir //移动a.text文件到 mydir文件夹 mydir: 文件夹名称
git rm a.text       //移除文件，将文件提交到分支后需要这样删除
git rm -r mydir     //删除文件夹, mydir:文件名称
rm a.text           //移除本地文件
```
>做任何文件操作时，需要先cd到相对应的目录，然后再执行命令，不然会找不到文件

### 标签管理
```
git tag         //查看标签     
git tag v1.0    //创建标签，v1.0: 标签名称
git tag v0.9 b957a5f    // 在对应的的版本打标签  b957a5f:提交过的版本号
git show v0.9   //查看标签信息
git tag -d v0.1 //删除本地标签
git push origin :refs/tags/v0.9 //删除远程仓库的标签，如果标签已经推送到远程仓库需要用此命令
git push origin --tags  //一次推送所有尚未推送到远程的本地标签
```

### 忽略文件
系统自动生成的文件或者你觉得不需要上传的文件，可以使用.gitignore文件进行忽略，github上面有对应的忽略配置[点此查看](https://github.com/github/gitignore)

```
touch .gitignore    //创建.gitigonre文件
vim .gitignore      //编辑，直接将上面配置好的忽略文件copy过来就可以了
```

### 参考网站
**[廖雪峰的Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/)** *如果想全面了解Git，强烈推荐去看一下廖老师的教程*

**[易百Git教程](https://www.yiibai.com/git/)**

