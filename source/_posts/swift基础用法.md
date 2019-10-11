---
title: swift基础用法
date: 2019-04-16 15:31:29
tags: [swift]
categories: iOS
---

## 标签
在编程过程中需要用到的一些备注，或者说明，使用MARK TODO FIXME （必须大写）
* // MARK: 说明文字
* // TODO: 需要提醒的文字
* // FIXME: 需要修改bug的相关说明

## 常见错误
### Expected expression after operator
代码中有使用 index++  需要更改为index += 1

### 如何对浮点数进行取余（取模）

```
let value1 = 5.5
let value2 = 2.2
let result = value1.truncatingRemainder(dividingBy: value2)
```

# 代理
## 扩展使用代理
```
extension ViewController: UITextFieldDelegate{

}
```

# swift 和oc之间的交互
swift和oc中可以互相调用，需要用到 MyGame-bridging-Header.h文件，MyGame是项目名称，swift中需要的头文件在里面导入。
链接文件有两种创建方式，一种是系统自动创建，一种手动创建，手动新建.h文件，名称改成 项目名`-brifging-Header.h ,Bunild Setting -> Objective Bridging Header` 填入本地路径 `$(SRCROOT)/MyGame/MyGame-bridging-Header.h, $(SRCROOT)`代表本地项目路径。
