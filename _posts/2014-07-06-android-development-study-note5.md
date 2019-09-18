---
layout: post
title: "安卓开发学习笔记5-Android应用四大组件之Activity"
description: ""
category: [Android]
---

#### Activity组件是Android应用程序中直接同用户进行可视化交互的组件。一个Android应用可以包含一个或者一组Activity，其中有一个Activity是用户通过点击应用Icon首次启动应用时呈现出的用户界面。每个Activity都被分配一个窗口来绘制自己的界面，一般情况下这个窗口会充满整个屏幕，开发者也可以控制该窗口在屏幕上以小窗体的形式显示或者浮动在其他窗体上面。

#### 一个Activity可以启动其他的Activity，这时，该Activity会被存入Activity栈中（通过点击“Back”按钮可以回到原来的Activity），然后新创建的Activity也会被放入该栈中，处于栈顶的位置并在屏幕顶层显示。这里的栈代表着Task的概念。

### 创建Activity
1. 通过在Java代码中定义一个Activity类型的子类就可以创建一个Activity，这个Activity的子类可以通过override Activity类型的回调方法来实现Activity在自己生命周期中状态变化时执行的操作。其中最重要的两个回调函数为：
	* __onCreate()__ - 这个方法必须被实现，因为在创建Activity实例时，Android系统会调用这个方法来初始化Activity的基础元素，方法体中，setContentView方法必须被调用从而使为该Activity设置的用户界面以定义好的layout呈现在屏幕上。
	* __onPause()__ - Android系统在用户离开当前Activity实例（该Activity实例被其他Activity实例完全覆盖不可见或者有其他Activity实例float在其上或者用户退出了该Activity实例）或者销毁其进程之前会调用该方法。在该方法中，我们应该做状态和数据的保存，因为用户可能不会再回到该Activity实例或者之后该Activity实例或被Android系统销毁。
	* 其他重要的回调函数在后面关于Activity生命周期的讨论中会继续讨论。
2. Activity的用户界面是由一系列继承自View类型的View对象呈现，用来响应用户操作以及展示信息，接受用户输入等。
3. 程序设计中，将界面和逻辑分离是被证明很有价值的做法，在之前的blog文章中已经介绍了独立定义Activity的布局对Android应用程序支持localization和compatibility方面的方式和意义。

### 声明Activity
* 为了让Android系统能够访问到我们创建的Activity，必须将其在Android应用的manifest文件中声明，[格式][1]如下：

{% highlight xml %}
<manifest ... >
  <application ... >
      <activity android:name=".ExampleActivity" />
      ...
  </application ... >
  ...
</manifest >
{% endhighlight %}

* 在声明Activity的时候，我们可以设置很多该Activity的属性，比如label，icon，theme等，下面是一个实际中的例子：

{% highlight xml %}
<activity android:name=".ExampleActivity" android:icon="@drawable/app_icon">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
{% endhighlight %}

* 在上面的实例中，可以看到__intent_filter__，该属性声明了其他应用组件如何激活该Activity。上面例子中的action元素声明了这个Activity是包含它的Android应用程序的主要（main）入口，首次点击应用图标启动应用时激活的Activity；category元素声明了该Activity应该被放入Android系统的应用启动器中，以允许用户通过点击应用图标进行启动。
* 当Android应用不允许其他应用程序激活自己的某个Activity时，可以不声明任何intent-filter，否则可以声明包含action，category以及data（后两者非必需）元素的intent-filter，像下面的例子一样：

{% highlight xml %}
<activity android:name="ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
{% endhighlight %}

### 启动Activity
* 通过调用startActivity()方法传递一个Intent类型对象参数来启动一个Activity，这里的Intent类型对象或者指定了要启动的Activity的名称，或者声明了要启动的Activity的action元素类型。对于后者，Android系统会选择一个符合条件的Activity来启动，如果有多个符合条件的，Android系统会列出来让用户选择。这里的Intent类型对象也可以携带少量的数据用来传递。下面有[两个官网的例子][2]，分别实例了这里所说的两种情况：

{% highlight java %}
Intent intent = new Intent(this, SignInActivity.class);
startActivity(intent);
{% endhighlight %}

{% highlight java %}
Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_EMAIL, recipientArray);
startActivity(intent);
{% endhighlight %}

* 通过调用startActivityForResult()方法传递一个Intent类型对象参数来启动一个Activity并处理其返回的结果。这种场景下，onActivityResult()方法应该被override，以便处理被启动的Activity放入Intent类型对象中的数据，[官网的例子][3]如下：

{% highlight java %}
private void pickContact() {
    // Create an intent to "pick" a contact, as defined by the content provider URI
    Intent intent = new Intent(Intent.ACTION_PICK, Contacts.CONTENT_URI);
    startActivityForResult(intent, PICK_CONTACT_REQUEST);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // If the request went well (OK) and the request was PICK_CONTACT_REQUEST
    if (resultCode == Activity.RESULT_OK && requestCode == PICK_CONTACT_REQUEST) {
        // Perform a query to the contact's content provider for the contact's name
        Cursor cursor = getContentResolver().query(data.getData(),
        new String[] {Contacts.DISPLAY_NAME}, null, null, null);
        if (cursor.moveToFirst()) { // True if the cursor is not empty
            int columnIndex = cursor.getColumnIndex(Contacts.DISPLAY_NAME);
            String name = cursor.getString(columnIndex);
            // Do something with the selected contact's name...
        }
    }
}
{% endhighlight %}

### 关闭Activity
1. 通过调用finish()方法可以关闭Activity自身。
2. 通过调用finishActivity()方法可以关闭自己启动的其他Activity

### Activity生命周期

#### 通过合理得override Activity类型中的回调函数可以控制Android应用中的Activity来提供优异的用户体验。文本前面提到过onCreate()和onPause()方法，下面我们结合Activity组件生命周期的不同状态继续讨论。

#### Activity的状态主要有下面三个：
1. Resumed - 也就是Running状态，表示该Activity正处于屏幕顶端，被用户使用。
2. Paused - 该Activity被其他Activity部分覆盖，它已不再有焦点，但是仍能被用户看到。这种状态下，该Activity的所有成员都存在于内存中，并还和Window manager保持联系。系统资源紧张的时候该Activity会被系统自动销毁。
3. Stopped - 该Activity处于后台，不再能被用户看到。这时该Activity的成员也还存在于内存中，但已经不再和Window manager保持联系。系统资源紧张的时候该Activity会被系统自动销毁。

##### 系统在销毁一个Activity实例的时候，会尝试要求该Activity调用自己的finish方法，也可能会直接kill进程，之后该Activity被重新启动时，需要从头开始初始化。

#### Activity生命周期示意图(来自[官网][4])：
![][activitylifecycle]

#### 下面是包含基本生命周期方法的Activity的例子：

{% highlight java %}
public class ExampleActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // The activity is being created.
    }
    @Override
    protected void onStart() {
        super.onStart();
        // The activity is about to become visible.
    }
    @Override
    protected void onResume() {
        super.onResume();
        // The activity has become visible (it is now "resumed").
    }
    @Override
    protected void onPause() {
        super.onPause();
        // Another activity is taking focus (this activity is about to be "paused").
    }
    @Override
    protected void onStop() {
        super.onStop();
        // The activity is no longer visible (it is now "stopped")
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // The activity is about to be destroyed.
    }
}
{% endhighlight %}

#### Activity生命周期中体现了下面三个重要的阶段：
1. 完整生命周期 - 从onCreate初始化到onDestroyed销毁。
2. 可见阶段 - 从onStart启动到onStop停止转入后台。
3. 前台阶段 - 从onResume运行到onPause暂停。

### 保存Activity状态

#### 当Activity处于非Resumed状态并且系统资源紧张时，Android系统可能会销毁该Activity实例，这时有必要保存该Activity实例的数据以便用户重新回到该Activity的时候能继续操作。另一种情况是当设备由竖屏变为横屏的时候，如果定义了不同的layout，Activity会被销毁然后根据定义的layout初始化，这时也需要保存和还原之前的状态数据。看一张官网的示意图：
![][save]

#### 通过调用onSaveInstanceState()方法和onSaveInstanceState()方法可以保存和还原Activity的状态。这里需要注意的是，Android系统不保证onSaveInstanceState()方法一定会在被销毁前调用，所以对于一些重要的数据，最好在onPause方法中保存。

#### [官网提供了一个非常好的Demo以及对其的说明][5]来帮助我们学习和理解Activity的生命周期，可以下载编译后运行看看。

[1]: http://developer.android.com/guide/components/activities.html
[2]: http://developer.android.com/guide/components/activities.html
[3]: http://developer.android.com/guide/components/activities.html
[4]: http://developer.android.com/guide/components/activities.html
[5]: http://developer.android.com/training/basics/activity-lifecycle/index.html
[activitylifecycle]: /images/activity_lifecycle.png
[save]: /images/restore_instance.png


