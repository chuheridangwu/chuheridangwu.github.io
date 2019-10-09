---
title: Intent
date: 2018-08-10 10:10:54
tags: [Intent]
categories: 安卓
---
Intent 是一个消息传递对象，可以使用它从其他应用组件请求操作。Intent 其基本用例主要包括以下三个：

* **启动Activity :**
    Activity 表示应用中的一个屏幕。通过将 Intent 传递给 startActivity()，您可以启动新的 Activity 实例。Intent 描述了要启动的 Activity，并携带了任何必要的数据。
    
如果希望在 Activity 完成后收到结果，请调用 startActivityForResult()。在上个 Activity 的 onActivityResult() 回调中，Intent会在这个方法中传递过来。

* **启动服务 :**
    Service 是一个不使用用户界面而在后台执行操作的组件。通过将 Intent 传递给 startService()，您可以启动服务执行一次性操作（例如，下载文件）。Intent 描述了要启动的服务，并携带了任何必要的数据。

如果服务旨在使用客户端-服务器接口，则通过将 Intent 传递给 bindService()，您可以从其他组件绑定到此服务
 
* **传递广播 :**
    广播是任何应用均可接收的消息。系统将针对系统事件（例如：系统启动或设备开始充电时）传递各种广播。通过将 Intent 传递给 sendBroadcast()、sendOrderedBroadcast() 或 sendStickyBroadcast()，您可以将广播传递给其他应用。
     <!--more-->
## 显式Intent
按名称（限定类名）指定要启动的组件

```
startActivity(new Intent(MainActivity.this,SendActivity.class));
```
## 隐式Intent
创建隐式Intent时，系统会通过Intent的内容与设备上其他应用的[清单文件](https://developer.android.com/guide/topics/manifest/manifest-intro)中声明的Intent过滤器进行比较，从而找到相应的组件。如果Intent 与 Intent过滤器匹配，系统将启动该组件，并向其传递Intent对象。
如果有多个Intent过滤器兼容，系统会显示一个对话框，支持用户选择要使用的应用。
如果没有为Activity声明任何Intent过滤器，Activity只能通过显式intent启动
`AndroidManifest.xml`文件就是清单文件
### 发送隐式Intent
通过隐式Intent调用 `<aciton>` 格式为`com.test.android.intent.action.Sendactivity`的Activity,**如果没有应用可以处理隐式Inetnt，应用会崩溃，所以调用前需要使用`resolveActivity()`方法验证手机是否有程序可以处理该Intent**

```
 Intent intent = new Intent("com.test.android.intent.action.Sendactivity");
            if (intent.resolveActivity(getPackageManager()) != null){
                    startActivity(intent);
                }
```

### 接收隐式Intent
为了接收隐式Intent，必须在Intent过滤器中将一个`<category>`标签设置为`DEFAULT`类别，这样方法`startActivity()`和`startActivityForResult()`才会按照已申明的类别方式处理所有的Intent,**如果未在Intent过滤器中声明此类别`DEFAULT`，则隐式Intent不会解析。**
`<action>`标签格式一般为：包名.intent.action.类名
```
<activity android:name=".SendActivity">
    <intent-filter>
        <action android:name="com.test.android.intent.action.Sendactivity"></action>
        <category android:name="android.intent.category.DEFAULT"></category>
    </intent-filter>
</activity>
```
### Intent过滤器 - IntentFilter
 intent 过滤器在清单中被指定为 <intent-filter> 元素。一个组件可有任意数量的过滤器，其中每个过滤器描述一种不同的功能。`category`和`action`标签可以有多个
 intent过滤器可以直接在`AndroidManifest.xml`文件中添加，也可以直接使用**IntentFilter**进行创建，文件中添加为静态，使用**IntentFilter**动态创建更加灵活
 
 ```
<intent-filter>
    <category android:name="android.intent.category.DEFAULT"></category>
    <action android:name="com.example.dym.locholandroid.intent.action.Sendaction"></action>
</intent-filter>
 ```
如果有多个应用都可以接收隐式Intent,会出现选项框让你选择要调用的应用。有两个选择，仅此一次或者永远。
选择仅此一次，下次点击按钮的时候弹窗还会出现，
选择永远，下次点击按钮直接调用，如果想可以重新选择，需要进入设置，清除**选择应用**的默认设置

### 通过URL打开app
需要在网页中植入一个点击链接，对应下面 `<data>`标签的**scheme**内容`<a href="app://">启动应用程序</a> `

```
<activity android:name=".SendActivity">
    <intent-filter>
        <category android:name="android.intent.category.BROWSABLE"></category>
        <category android:name="android.intent.category.DEFAULT"></category>
        <action android:name="android.intent.action.VIEW"></action>
        <data android:scheme="app"></data>
    </intent-filter>
</activity>
```
`android.intent.category.BROWSABLE`: 可以被浏览器启动


### 隐式Intent的使用技巧
可以直接通过隐式Intent调用其他app的活动
如果Activity只允许在当前程序内部才可以访问，需要对Activity配置`android:exported="false"`，这样其他程序就无法调用当前活动了

### 注意点
>为了确保应用的安全性，启动 Service 时，请始终使用显式 Intent，且不要为服务声明 Intent 过滤器。使用隐式 Intent 启动服务存在安全隐患，因为您无法确定哪些服务将响应 Intent，且用户无法看到哪些服务已启动。从 Android 5.0（API 级别 21）开始，如果使用隐式 Intent 调用 bindService()，系统会引发异常。

## 隐式 Intent 如何通过系统传递以启动其他 Activity 的
![](https://developer.android.com/images/components/intent-filters@2x.png)
图解：
[1] Activity A 创建包含操作描述的 Intent，并将其传递给 startActivity()。
[2] Android 系统搜索所有应用中与 Intent 匹配的 Intent 过滤器。 找到匹配项之后。
[3] 该系统通过调用匹配 Activity（Activity B）的 onCreate() 方法并将其传递给 Intent，以此启动匹配 Activity。

## 参考网址
[谷歌Intent文档](https://developer.android.com/guide/components/intents-filters)





