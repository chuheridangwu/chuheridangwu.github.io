---
title: Service
date: 2018-08-30 17:28:42
tags: Service
cotegories: 安卓
---

Service 是一个可以在后台执行长时间运行操作而不提供用户界面的应用组件。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。 此外，组件可以绑定到服务，以与之进行交互，甚至是执行进程间通信 (IPC)。 例如，服务可以处理网络事务、播放音乐，执行文件 I/O 或与内容提供程序交互，而所有这一切均可在后台进行。

## 创建Service
New -> Service -> Service,创建一个`MyService`类,继承自`Service`,Android Studio会主动在`AndroidManifest.xml`文件进行注册。

```
<service
   android:name=".MyService"
   android:enabled="true"
   android:exported="true">
</service>
```
`name`: 类名
`enabled`: 是否启用这个服务
`exported`: 是否允许当前程序之外的其他程序访问这个服务
<!--more-->
### Service中对应的重要回调方法:
**onStartCommand()**：当另一个组件通过 `startService()` 请求启动服务时，系统将调用此方法。一旦执行此方法，服务即会启动并可在后台无限期运行。 如果您实现此方法，则在服务工作完成后，需要由您通过调用 `stopSelf()` 或 `stopService()` 来停止服务，绑定服务不会调用这个方法
**onBind()**：当另一个组件想通过调用 `bindService()` 与服务绑定（例如执行 RPC）时，系统将调用此方法。在此方法的实现中，您必须通过返回 `IBinder` 提供一个接口，供客户端用来与服务进行通信
**onCreate()**：首次创建服务时，系统将调用此方法来执行一次性设置程序（在调用 `onStartCommand()` 或 `onBind()` 之前）。如果服务已在运行，则不会调用此方法。
**onDestroy()**：当服务不再使用且将被销毁时，系统将调用此方法。服务应该实现此方法来清理所有资源，如线程、注册的侦听器、接收器等。 这是服务接收的最后一个调用。

## Server的使用方式
服务有两种方式，启动`startService() `和绑定`bindService()`，对应的回调方法`onStartCommand()`（允许组件启动服务）和 `onBind()`（允许绑定服务）
启动和绑定通过`Intent`来调用服务（即使此服务来自另一应用),也可以通过清单文件将服务声明为私有服务
**默认情况下，服务是在应用的主线程中运行，因此，如果服务执行的是密集型或阻止性操作，则您仍应在服务内创建新线程。**
### 启动服务开启、关闭服务
```
Intent intent = new Intent(this,MyService.class);
startService(intent);

// 停止服务
stopService(intent);
```
`startService()` 启动服务（同时会对`onStartCommand()` 调用），服务将一直运行，直到服务使用 `stopSelf()` 自行停止运行，或由其他组件通过调用`stopService() `停止它为止。多个服务启动请求会导致多次对`onStartCommand()`方法 进行相应的调用,
开启后的服务可以在 设置-> 应用 -> 正在运行的状态中找到该程序


#### 开启服务之后的数据传递
假设某 {% post_link Activity Activity %} 需要将一些数据保存到在线数据库中。该 Activity 可以启动一个协同服务，并通过向 startService() 传递一个 {% post_link Intent Intent %}，为该服务提供要保存的数据。服务通过 onStartCommand() 接收 Intent，连接到互联网并执行数据库事务。事务完成之后，服务会自行停止运行并随即被销毁

### 绑定、解绑服务
如果组件是通过调用 `bindService()` 来创建服务（且未调用 `onStartCommand()`，则服务只会在该组件与其绑定时运行。一旦该服务与所有客户端之间的绑定全部取消，系统便会销毁它。**绑定服务通常只在为其他应用组件服务时处于活动状态，不会无限期在后台运行**

创建一个`MyService`类继承自`Service`,在`onBind()`方法中返回一个`Binder`的实例，实现组件和service之间的绑定。

```
public class MyService extends Service {
    public MyService() {
    }

    public  MyBinder binder = new MyBinder();

    // 通过Mybinder进行通讯
    class  MyBinder extends Binder{
       
    }

    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }

    @Override
    public boolean onUnbind(Intent intent) {
        System.out.println("解除绑定 - onUnbind");
        return super.onUnbind(intent);
    }

    @Override
    public void onCreate() {
        super.onCreate();
        System.out.println("创建服务 - onCreate");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        System.out.println("开启服务 - onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        System.out.println("销毁服务 - onDestroy");
    }
    
}
```

其他组件调用绑定服务，例子中是`Activity`组件的调用

```
// 创建服务链接
   private ServiceConnection connection = new ServiceConnection() {
       @Override
       public void onServiceConnected(ComponentName name, IBinder service) {
           System.out.println("绑定服务成功");
       }

       @Override
       public void onServiceDisconnected(ComponentName name) {

       }
   };
// 绑定服务
bindService(intent, connection,BIND_AUTO_CREATE);
```

调用`bindService()`绑定服务，需要传入三个参数,`Intent`、`ServiceConnection`、`BIND_AUTO_CREATE`。
`ServiceConnection`:是一个接口类，需要重写里面的`onServiceConnected()`和`onServiceDisconnected()`方法。`onServiceConnected()`
`BIND_AUTO_CREATE`:表示在活动和服务进行绑定后自动创建服务，会调用`onCreate()`方法

```
// 解绑服务
unbindService(MainActivity.this);
```

**这里`unbindService`方法只有在绑定之后才能调用，未绑定调用服务调用此方法造成崩溃**

## Service的生命周期
![](https://developer.android.com/images/service_lifecycle.png?hl=zh-cn)
左图显示了使用 `startService()` 所创建的服务的生命周期，右图显示了使用` bindService()` 所创建的服务的生命周期。

* `onCreate`方法只会创建一次
* 除非所有客户端均取消绑定，否则 `stopService()` 或 `stopSelf()` 不会实际停止服务。**如果使用`startService()`方法开启服务&&使用`bindService()`方法绑定服务，需要先调用`unbindService()`解绑，之后调用`stopSelf()`或`stopService()`服务才会注销**

## 参考内容
[Service官方文档](https://developer.android.com/guide/components/services?hl=zh-cn)
[Service学习笔记](https://www.jianshu.com/p/e3b7954b9c00)



