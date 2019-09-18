---
layout: post
title: "安卓开发学习笔记8-Android应用四大组件之Broadcast Receiver"
description: ""
category: [Android]
---

### 什么是Broadcast Receiver

#### Broadcast Receiver 是允许我们对Android系统或者Android应用程序的事件进行注册的组件。当一个事件发生时，Android Runtime 会通知所有注册到该事件的receiver，这些 receivers 事先定义的针对该事件的响应逻辑将被执行。比如，应用程序可以注册 ACTION_BOOT_COMPLETED 系统事件，当Android系统完成启动流程后，该应用程序就可以做出相应的响应。

### Broadcast Receiver的实现
1. 可以在 AndroidManifestxml 文件中静态注册 receiver
2. 可以在代码中使用 Context.registerReceiver() 方法动态注册 receiver
3. 实现一个继承自 BroadcastReceiver 类型的类，并实现其 onReceive() 方法
4. 当该receiver注册的事件发生时，Android 系统会调用该 receiver 的 onReceive() 方法
5. 当一个 receiver 的 onReceive() 方法执行完后，Android系统就可以回收该 receiver 的资源。

#### 在 API lever 11 之前，onReceive() 方法中不能执行异步操作，当其执行完后，该 receiver 实例也会被 Android 系统销毁，所以为了避免ANR，如果需要执行需要较长时间运行的逻辑，最好在 onReceive() 方法中启动一个 Service 实例来工作。从 API level 11 之后，我们可以使用 goAsyc() 方法来执行异步操作，该方法返回一个 PendingResult 类型对象，Android系统不会销毁该 receiver 实例直到PendingResult.finish() 方法被调用。 

#### 从 Android 3.1 开始，如果用户显示地停止了某个应用程序的运行，默认情况下，则该应用的所有 receiver 注册的事件发生时它们都不会收到通知。

### 使用 Broadcast Receiver

#### 我们通过一个例子来看静态注册 action.BOOT_COMPLETED 事件的 receiver 如何实现。
1. 在 AndroidManifest.xml 文件中注册该 receiver，并申请 android.permission.RECEIVE_BOOT_COMPLETED 的权限：

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    ...
        <receiver android:name="MyScheduleReceiver" >
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>
        <receiver android:name="MyStartServiceReceiver" >
        </receiver>
{% endhighlight %}

2. 定义一个继承自 BroadcastReceiver 类型的类，并实现其 onReceive() 方法，本例中启动了一个Service：

{% highlight java %}
public class MyReceiver extends BroadcastReceiver {

  @Override
  public void onReceive(Context context, Intent intent) {
    // assumes WordService is a registered service
    Intent intent = new Intent(context, WordService.class);
    context.startService(intent);
  }
} 
{% endhighlight %}

##### 注意，测试的时候，包含本例的应用程序必须状态机身而不是SDCard中，否则当 action.BOOT_COMPLETED 事件发生时SDCard还没有初始化并挂载，该应用程序就无法响应该事件。

#### 关于 receiver 动态注册事件，还是用例子来说明。
1. 使用 Context.registerReceiver() 可以动态注册一个 receiver 到一个事件，之后，一定要注意在合适的时候调用 Context.unregisterReceiver() 来取消注册，否则会发生 leaked broadcast receiver error
2. 使用 package manager，可以在运行时 enable 或者 disable 一个静态注册的 receiver：

{% highlight java %}
ComponentName receiver = new ComponentName(context, myReceiver.class);
PackageManager pm = context.getPackageManager();

pm.setComponentEnabledSetting(receiver, 
  PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
  PackageManager.DONT_KILL_APP); 
{% endhighlight %}

#### 关于 Sticky broadcast intents

#### 一般情况下，当一个用来将 receiver 和事件动态注册的 Intent 对象被 Android 系统传递后就不能被使用了，但是使用 sendStickyBroadcast(Intent) 方法后，该 Intent 对象会一直存在，即使事件发生，receiver 的 onReceive() 也被执行完毕。这种情况主要用在我们想在 Intent 对象中保存相关系统信息的时候。

### Broadcast 的类型
1. __Normal broadcast__  - 通过 Context.sendBroadcast 异步发送。所有该广播的 receiver 会以未定义的顺序执行，一般情况下是同时进行的。
2. __Ordered broadcast__ - 通过 Context.sendOrderedBroadcast 顺序发送到每个 receiver，我们可以把一个 receiver 的执行结果传递给下一个 receiver，也可以在某个 receiver 里面放弃该广播的继续传递。receiver 被执行的数序可以通过 android:priority 定义，当多个 receiver 具有相同的 priority 的时候，它们的执行顺序是随机的。动态注册的 receiver 比静态注册的 receiver 优先级要高。

### 参考资料：

1. [官网文档][1]
2. [Android中Broadcast Receiver组件详解][2]
3. [Android BroadcastReceiver - Tutorial][3]

[1]: http://developer.android.com/reference/android/content/BroadcastReceiver.html
[2]: http://blog.csdn.net/zuolongsnail/article/details/6450156
[3]: http://www.vogella.com/tutorials/AndroidBroadcastReceiver/article.html

