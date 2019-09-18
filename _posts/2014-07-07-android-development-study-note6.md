---
layout: post
title: "安卓开发学习笔记6-Android应用四大组件之Service"
description: ""
category: [Android]
---


#### __Service__ 是Android应用程序中用来执行较长时间操作的没有用户界面的后台组件。Service组件实例可以被其他Android应用程序组件实例启动，并在用户切换到其他应用程序后仍然在后台运行。另外，其他Android应用程序组件实例可以bind到一个Service实例进行数据交互，甚至进行进程间的通信。

#### 一个Service组件实例主要有下面两种形式：
1. __Started__ - 一个Service实例被其他组件实例通过startService()方法启动后，就认为其处于 __Started__ 状态。此时该Service组件实例可以在启动它的其他组件实例被销毁后继续在后台运行。一般情况下，Service实例用于执行某个特定操作，不需要返回数据。
2. __Bound__ - 一个Service实例被其他组件实例通过 bindService() 方法激活后，就认为其处于 __Bound__ 状态。此时该Service提供一个C/S接口来和其他组件实例进行数据交互，此时甚至可以进行进程间的通信。处于 __Bound__ 状态的 Service 实例在bind到它的所有其他组件实例结束 bind 后被销毁。

#### Service实例可以同时处于 __Started__ 和 __Bound__ 状态，这个取决于override了那些Service类型的回调函数。通过实现onStartCommand()允许其他组件实例启动该Service，而实现onBind()可以允许其他组件实例对其实例进行bind。默认情况下，Service实例运行在主进程的主线程中，所以如果该Service需要执行CPU密集型或者较长时间的操作，最好在Service实例中新建一个线程来执行操作，从而降低出现ANR的几率，避免影响到用户体验。

#### 一般情况下，由于一个Service实例可以在启动它的组件被销毁后仍然运行在后台，所以当有这种业务需求时，Service是个好选择，否则可以使用线程。

### 创建Service

* 通过在Java代码中定义一个Service类型的子类就可以创建一个自定义的Service。这个Service的子类可以通过override Service类型的回调方法来实现Activity在自己生命周期中状态变化时执行的操作。其中最重要的四个回调函数为：
  1. onStartCommand() - Android系统会在其他应用程序组件实例通过startService()方法启动一个Service实例的时候调用onStartCommand()方法，在执行完操作后，调用stopSelf()方法结束自己，或者在调用它的其他组件实例中调用startService()方法来结束它。如果该service被设计为会被其他组件实例bind，那么可以不用实现onStartCommand()方法。
  2. onBind() - Android系统会在其他应用程序组件实例通过bindService()方法启动一个Service实例的时候调用onBind()方法。在该方法中，一个可以和其他组件实例通信的IBinder接口必须被返回，如果不希望该Service被bind，则返回null，但onBind()方法都必须被实现。
  3. onCreate() - Android系统在Service实例被创建的时候调用该方法，在方法体中，会调用onStartCommand()或者onBind()方法，如果Service实例已经存在，则onCreate()不会被再次执行。
  4. onDestroy() - Android系统在销毁Service实例的时候调用该方法。在该方法中，Service运行过程中申请的资源应该被正确地释放，这是一个Service最后一个被调用的方法。

#### 由于Android系统在资源紧张的时候会强制结束一些Service实例，所以在这种情况发生后，是否需要重启该Service，重启后是否需要继续处理Intent需要根据业务需求来确定。Android提供了相关机制，文本后面会讨论到。

### 声明Service

#### 所有Android应用程序中用到的Service都需要在应用的manifest文件中声明，[官网给出了格式][1]:

{% highlight xml %}
<manifest ... >
  ...
  <application ... >
      <service android:name=".ExampleService" />
      ...
  </application>
</manifest>
{% endhighlight %}

#### Service元素可以有很多属性，下面实际应用中的例子中有个intent-filter：

{% highlight xml %}
        <service android:name=".ServiceB">
            <intent-filter >
                <action android:name="com.xhrwang.servicedemo.actionB"/>
            </intent-filter>
        </service>
{% endhighlight %}

#### 关于如何平衡Android应用程序的安全和应用所提供功能的要求，官网给出了一些建议：

> To ensure your app is secure, always use an explicit intent when starting or binding your Service and do not declare intent filters for the service. If it's critical that you allow for some amount of ambiguity as to which service starts, you can supply intent filters for your services and exclude the component name from the Intent, but you then must set the package for the intent with setPackage(), which provides sufficient disambiguation for the target service.

> Additionally, you can ensure that your service is available to only your app by including the android:exported attribute and setting it to "false". This effectively stops other apps from starting your service, even when using an explicit intent.

### “ __Started__ ” Service

#### Android应用程序的其他组件实例可以通过startService()方法启动一个Service实例，这时Android系统会调用onStartCommand()方法。该Service实例甚至可以在启动它的其他组件实例被销毁后继续在后台运行，在完成特定操作后，该Service实例可以调用stopSelf()方法自己退出，其他组件实例也可以通过调用stopService()方法主动退出该Service实例。

#### 一般情况下，下面两种类型的Service可以按这种方式启动：
1. __Service__ - 这是Service的基类，当我们自己创建一个继承自它的Service类型后，需要新建一个线程来执行操作，否则该Service的实例在主进程的主线程中运行可能会造成应用程序ANR，影响用户体验。[官网提供了一个使用该类型的例子][3]，这个例子显示了如何手动进行多线程处理每个请求。这里有两点需要注意的：
  * 在Service实例完成所有工作后，需要自己调用 stopSelf() 方法退出或者在其他组件实例中调用 stopService() 退出该Service实例。
  * onStartCommand() 方法需要返回一个int类型的对象，这个整型数值告诉Android系统当该Service实例在资源紧张的时候被系统杀掉之后如何继续，其取值情况分为：
    1. START\_NOT\_STICKY - 当系统在 onStartCommand() 方法执行完后杀掉了Service实例，如果没有等待处理的传递给该Service的Intent，系统不需要重启该Service实例。
    2. START\_STICKY - 当系统在 onStartCommand() 方法执行完后杀掉了Service实例，重启该Service实例，并调用 onStartCommand() 方法，并传递待处理的Intent对象，如果有的话，否则传递一个空Intent对象，不自动重发最近一次的Intent对象。
    3. START\_REDELIVER\_INTENT - 类似于 START\_STICKY ，区别在于会在调用 onStartCommand() 方法的时候自动传递最近的一次发给该Service的Intent对象。
2. __IntentService__ - 继承自Service类型，该类型有一个单独的worker线程依次处理每个请求的操作。当发给Service的请求不需要并行处理时，可以使用该类型，我们可以实现 onHandleIntent() 方法来处理每个请求。具体的运行情况如下：
  * 启动一个单独的worker线程处理所有传递过来的Intent对象。
  * 提供一个默认的 onStartCommand() 方法中为所有Intent对象创建一个工作队列，依次将它们传递给 onHandleIntent() 方法执行。
  * 在所有Intent对象被处理完毕后调用 stopSelf() 方法自己退出，我们不需要自己去关心该Service实例的生命周期。
  * 提供一个默认的 onBind() 方法并返回null。

  所以我们唯一需要做的就是实现 onHandleIntent() 方法。[官网提供了一个例子][4]:

{% highlight java %}
public class HelloIntentService extends IntentService {

  /**
   * A constructor is required, and must call the super IntentService(String)
   * constructor with a name for the worker thread.
   */
  public HelloIntentService() {
      super("HelloIntentService");
  }

  /**
   * The IntentService calls this method from the default worker thread with
   * the intent that started the service. When this method returns, IntentService
   * stops the service, as appropriate.
   */
  @Override
  protected void onHandleIntent(Intent intent) {
      // Normally we would do some work here, like download a file.
      // For our sample, we just sleep for 5 seconds.
      long endTime = System.currentTimeMillis() + 5*1000;
      while (System.currentTimeMillis() < endTime) {
          synchronized (this) {
              try {
                  wait(endTime - System.currentTimeMillis());
              } catch (Exception e) {
              }
          }
      }
  }
}
{% endhighlight %}

### “__Bound__” Service

#### 如同前面提到的，其他Android应用程序组件实例通过 bindService() 方法激活的Service实例就是 Bound Service，以建立一个同该Service实例的长连接进行数据交互。一般情况下，这种情况下我们不会实现 onStartCommand() 方法，也不允许其他组件实例通过  startService() 方法启动该Service实例。

#### 这种情况下，我们需要实现 onBind() 方法并返回一个IBind对象，该对象定义了和该Service实例进行通信的接口，其他组件实例可以通过 bindService() 方法来获得该IBind对象并调用该Service提供的方法。当所有bind到该Service实例的其他组件实例都结束bind之后，Android系统会销毁该Service实例，我们不需要主动去停止该Service实例。

### 给用户发通知

#### 正在运行的Service实例可以通过 Toast Notifications 和 Status Bar Notifications 来发通知给用户：
1. __Toast Notifications__ - 一个出现在当前窗口最顶层的消息窗口，在呈现指定时间后自动消失。
2. __Status Bar Notifications__ - 在状态通知栏显示一条由icon和消息组成的通知，用户点击后可以执行一个Action，比如启动一个Activity。

### 前台服务

#### 前台服务被认为是和用户当前操作相关的正在运行的服务，该服务即使在资源紧张的时候也不会被系统杀掉，而且必须在状态通知栏放置一条通知。该通知无法被取消，直到该服务被停止后者该服务不再是前台服务。

#### 可以通过下面的方式启动一个前台服务：

{% highlight java %}
Notification notification = new Notification(R.drawable.icon, getText(R.string.ticker_text),
        System.currentTimeMillis());
Intent notificationIntent = new Intent(this, ExampleActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent, 0);
notification.setLatestEventInfo(this, getText(R.string.notification_title),
        getText(R.string.notification_message), pendingIntent);
startForeground(ONGOING_NOTIFICATION_ID, notification);
{% endhighlight %}

### Service的生命周期

#### Service的生命周期示意图：
![][lifecycle]

#### 本文前面已经详细说明了 Started Service 和 Bound Service 在使用的时候调用Service类型回调函数的情况，这里就不多说了，需要注意的是Service的完整生命周期从 onCreate() 方法被调用开始，到 onDestroy() 方法被调用结束。 Started Service 的 active 生命期从 onStartCommand() 被调用开始，到 onDestroy() 被调用结束，而 Bound Service，其 active 生命期从 onBind() 被调用开始，到 onUnbind() 被调用结束。


#### 参考官网Activity Demo而写的[关于Service的Demo][2]

[1]: http://developer.android.com/guide/components/services.html
[2]: https://github.com/xhrwang/AndroidServiceDemo
[3]: http://developer.android.com/guide/components/services.html
[4]: http://developer.android.com/guide/components/services.html
[lifecycle]: /images/service_lifecycle.png


