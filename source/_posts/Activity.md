---
title: Activity
date: 2018-08-08 17:48:47
tags: [Activity]
categories: 安卓 
---
Android 是使用任务(Task)来管理活动的，一个任务就是一组存放在栈里的活动集合，这个栈也称为返回栈。**栈是一种后进先出的数据结构**，在默认情况下，当我们启动一个新的Activity，它会在返回栈中入栈，处于栈顶。当我们按下Back键或者调用`finsih()`方法销毁一个活动时，处于栈顶的活动会出栈，前一个入栈的活动会处于栈顶的位置。系统总是显示栈顶的活动给用户。
<!--more-->
## Activity的状态
### 运行状态
当活动处于栈顶的时候属于运行状态，系统最不愿意回收的就是处于运行状态的活动。
### 暂停状态
当一个活动不再处于栈顶，但仍然可见时，这时活动进入暂停状态。处于暂停状态的活动仍然是完全存活的，只有在内存极低的情况下，系统才会去考虑回收这种活动
### 停止活动
当活动不处于栈顶并且完全不可见的时候，就进入停止状态，系统仍然会为这种活动保存相应的状态和成员变量，这不是完全可靠的，当其他地方需要内存时，处于停止状态的活动可能会被系统回收
### 销毁状态
当活动从返回栈中移除后属于销毁状态，系统会回收这种状态的活动

## Activity的生命周期
* `onCreate()`: 在活动第一次被创建的时候调用，加载布局、绑定时间等操作在这个方法中进行
* `onStart()`:  活动由不可见变为可见的时候调用
* `onResume()`: 活动准备好和用户进行交互的时候调用，此时活动一定位于返回栈的栈顶，并处于运行状态  
* `onPause()`: 系统准备去启动或者恢复另一个活动的时候调用，通常会在这个方法将消耗CPU的资源释放掉，以及保存一些关键数据，**这个方法的执行速度一定要快，不然会影响新的栈顶活动的使用** 
* `onStop()`: 活动完全不可见的时候调用，它和`onPause()`方法的区别在于，如果启动的新活动是一个对话框式的活动，那么`onPause()`方法会得到执行，而`onStop()`方法并不会执行 
* `onDestroy()`: 活动被销毁之前调用，之后的活动状态将变为销毁状态 
* `onRestart()`: 活动由停止状态变为运行状态之前调用，也就是活动被重新启动了 
 ![](https://developer.android.com/guide/components/images/activity_lifecycle.png?hl=zh-cn)
 
### Example：
当前有两个活动`MainActivity`和`SendaActivity`,MainActivity为主入口活动，在生命周期的每个方法上都打上log。
在启动程序的时候，运行顺序： 
`MainActivity: onCreate()`、
`MainActivity: onStart()`、
`MainActivity: onResume()`
从MainActivity跳转到SendaActivity活动时，调用顺序：
`MainActivity: onPause()`、
`SendActivity: onCreate()`、
`SendActivity: onStart()`、
`SendActivity: onResume()`、
`MainActivity: onStop()`
从SendaActivity返回MainActivity时的调用顺序：
`SendActivity: onPause()`、
`MainActivity: onRestart()`、
`MainActivity: onStart()`、
`MainActivity: onResume()`、
`SendActivity: onStop()`、
`SendActivity: onDestroy()`

## Activity之间的数据传递
`Activity`之间可以直接使用`Intent`进行传递数据，或者把数据包装成`Bundle`，再用`Intent`进行传递。可以传递简单的数据，或者传递对象，对象需要进行序列化。

### 传递普通数据
```
Intent i = new Intent(MainActivity.this,TestActivity.class);
i.putExtra("data","helloWordxiaojeijei");
startActivity(i);
```

### 使用`Bundle`传递数据

#### 传递数据
```
Intent i = new Intent(MainActivity.this,TestActivity.class);
Bundle bundle = new Bundle();
bundle.putInt("age",17);
bundle.putString("name","dongyumao");
i.putExtras(bundle);
startActivity(i);        
```
#### 获取数据

```
Intent i  = getIntent();
Bundle b = i.getExtras();
TextView textView = findViewById(R.id.textView);
textView.setText(String.format("age=%d,name=%s，name1=%s",b.get("age"),b.get("name"),b.getString("name1","test")));
```
`b.getString("name1","test")`: 如果name1字段没有数据，使用默认数据"test"

### 使用对象进行传递数据
需要对类进行序列化，使用对象传递数据有两种方式`implements Serializable`或者 `implements Parcelable`。`Parcelable`需要重写两个方法

#### 使用`implements Serializable`进行传递创建User对象，

```
public class User  implements Serializable{
    private String name;
    private int age;

    // 重写 get set 方法
    
   public User(String name,int age){
        this.name = name;
        this.age = age;
   }
}
```
##### 发送消息

```
Intent i = new Intent(MainActivity.this,TestActivity.class);
i.putExtra("data", new User("test111",18));
startActivity(i);
```

##### 接受消息

```
 Intent i  = getIntent();
 User u = (User) i.getSerializableExtra("data");
 TextView textView = findViewById(R.id.textView);
 textView.setText(String.format("age=%d,name=%s",u.getAge(),u.getName()));
```
#### 使用`Parcelable`进行传递对象

创建User类，需要重写Parcelable的`describeContents()` 和 `writeToParcel()` 方法，在`writeToParcel()`方法中调用 `dest`的`writeXXX()`方法，将User字段一一写出来。
同时，需要在User类中提供一个名为 **CREATOR** 的常量，创建Parcelable.Creator 接口的一个实现，并将泛型指定为User，需要重写 `createFromParcel()` 和 `newArray()`方法，在 `newArray()`方法中需要返回一个User数组，并使用方法传入size作为数组大小， `createFromParcel()` 方法需要使用`Parcel`的 `readXXX()` 方法读取刚才写入的字段，并创建一个User对象进行返回.
##### user类
```
public class User  implements Parcelable{
    private String name;
    private int age;

     // 重写 get set 方法

   public User(String name,int age){
        this.name = name;
        this.age = age;
   }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(age);
    }

    public static final Parcelable.Creator<User> CREATOR = new Creator<User>() {
        @Override
        public User createFromParcel(Parcel source) {
            return new User(source.readString(),source.readInt());
        }

        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };

}
```

##### 发送消息

```
Intent i = new Intent(MainActivity.this,TestActivity.class);
i.putExtra("data", new User("test111",18));
startActivity(i);
```

##### 接收消息

```
 Intent i  = getIntent();
 User u = (User) i.getParcelableExtra("data");
 TextView textView = findViewById(R.id.textView);
 textView.setText(String.format("age=%d,name=%s",u.getAge(),u.getName()));
```

### 回调方法，从B返回A的时候传递数据

A到B,在启动B  Activity时，需要调用`startActivityForResult(Intent,int)` 方法，并且重写`onActivityResult()`方法，B的返回结果都会在该方法中展示
```
 @Override
protected void onActivityResult(int requestCode, int resultCode, Intent data){
    super.onActivityResult(requestCode, resultCode, data);
    String str = data.getStringExtra("key");
}
```


从B返回A

```
 Intent i = new Intent();
 i.putExtra("key", "测试结果");
 setResult(1,i);
 finish();
```

## Activity的启动模式
活动的启动模式有4种，`standard` `singleTop` `singleTask` `singleInstance`,可以在 **AndroidManifest.xml** 中通过<activity>标签指定 `android:launchMode`属性来选择启动模式

```
 <activity android:name=".MainActivity"
            android:launchMode="standard">
```

### standard
`standard`是活动默认使用的启动模式，在不进行显示指定的情况下，所有活动都会自动使用这种启动模式。使用`standard`模式的活动，系统不会在乎这个活动是否已经在栈中存在，每次启动都会创建一个新的活动实例，并置于栈顶

### singleTop
`singleTop`模式可以解决重复创建栈顶活动的问题，如果当前活动处于栈顶，则不会继续创建新的活动，但是如果当前活动不在栈顶，则会创建新的活动

### singleTask
`singleTask`模式在整个app运行期间只会创建一个实例

### singleInstance
`singleInstance`会启动一个新的返回栈来管理这个活动（如果`singleTask`模式指定了不同的`taskAffinity`,也会启动一个新的返回栈）

## Activity的小知识

### 设置入口活动
在`AndroidManifest.xml`中的`<intent-filter>`标签设置主活动

```
<activity android:name=".MainActivity"
        android:launchMode="singleInstance">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />

            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
```

### Activity的全路径
`<activity android:name=".SendActivity"></activity>` 
加上包的路径`package="com.example.dym.testactivity"` 
是活动的全路径 `com.example.dym.testactivity.SendActivity`

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.dym.testactivity">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity"
            android:launchMode="singleInstance">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name=".SendActivity"></activity>
    </application>

</manifest>
```

## 参考文档
[谷歌官方文档](https://developer.android.com/guide/components/activities/intro-activities)
[第一行代码第2版]


