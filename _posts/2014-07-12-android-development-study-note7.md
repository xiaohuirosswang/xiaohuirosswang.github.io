---
layout: post
title: "安卓开发学习笔记7-Android应用四大组件之ContentProvider"
description: ""
category: [Android]
---

#### __Content Provider__ 是从一个应用程序访问另一个应用程序中结构化数据集合的标准接口，并提供了一套安全机制。Android系统已经为常用的数据，比如通信录数据，图片，音频视频数据等提供了Content Provider。另外，Content Provider隐藏了数据的实际存储形式和读写方式，通过其提供的标准接口，可以方便其他应用程序进行增删查改。

#### Content Provider以类似关系型数据库中数据表的方式对其他Android应用程序提供数据。其中一行代表一组数据类型集合，一列代表某种相同的数据类型。每一行记录都有一个唯一的long类型的_ID字段来唯一标识。

### Content Provider 的权限控制

#### 当一个Android应用程序通过 Content Provider 对其他程序共享数据时，可以指定相应的权限，而需要访问它的应用程序就必须在manifest文件中申请该权限以便能够访问该 Content Provider。 这种授权可以让用户明确知道有个应用程序正在试图访问那些数据，因为用户在安装需要访问该 Content Provider 并申请了相应权限的应用程序时会被告知这一点。比如，当一个应用程序需要访问 User Dictionary Provider 的时候，必须在自己的 manifest 文件中做如下的声明：

{% highlight xml %}
 <uses-permission android:name="android.permission.READ_USER_DICTIONARY">
{% endhighlight %}

### Content Provider 的定义

#### 自定义一个 Content Provider 的时候，大致可以分为下面这些步骤：
1. 定义 CONTENT_URI 常量
2. 定义一个继承自 ContentProvider 类型的 Provider 类。
3. 在自定义的 Provider 类型中，实现下面的方法：
	* public boolean onCreate() - Android系统会在 Content Provider 被创建后调用该方法，一般来说，Content Provider 实例在其被第一次调用的时候创建。
	* public Uri insert(Uri uri, ContentValues values) - 其他应用程序可以通过该方法在 Content Provider 中添加记录数据。
	* public int delete(Uri uri, String selection, String[] selectionArgs) - 其他应用程序可以通过该方法在 Content Provider 中删除记录数据。
	* public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) - 其他应用程序可以通过该方法在 Content Provider 中更新记录数据。
	* public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) - 其他应用程序可以通过该方法在 Content Provider 中查询数据。
	* public String getType(Uri uri) - 获得当前URI所代表的数据的 MIME 类型。如果数据属于集合类型，则返回 vnd.android.cursor.dir/ 开头的类型字符串；如果数据属于非集合类型，返回 vnd.android.cursor.item/ 开头的类型字符串。
4. 将该 Provider 类型在 AndroidManifest.xml 文件中进行声明，使用下面的格式：

{% highlight xml %}
<application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <provider android:name=".CustomerizedProvider" android:authorities="com.xhrwang.provider.myprovider"/>
         ...
</application>
{% endhighlight %}

5. URI对象，Uri对象主要由两部分数据构成：
	* 将要操作的 Content Provider
	* 将要对该 Content Provider 中的什么数据进行操作

> content://com.xhrwang.provider.myprovider/mydata/1，其中“content”表示scheme，“com.xhrwang.provider.myprovider”表示主机名，或者说authority，“mydata”表示路径，“1”表示_ID.

### Content Resolver

### 当一个Android应用程序需要操作另一个应用程序共享的Provider中的数据时，其可以使用ContentResolver类型对象来进行。我们可以通过调用一个 Activity 对象的 getContentResolver() 方法来获得当前 contentResolver 对象。和 Content Provider 类型相对应，Content Resolver 对象也提供了下面相应的方法：

{% highlight java %}
public Uri insert(Uri uri, ContentValues values)：该方法用于往ContentProvider添加数据。
public int delete(Uri uri, String selection, String[] selectionArgs)：该方法用于从ContentProvider删除数据。
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)：该方法用于更新ContentProvider中的数据。
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)：该方法用于从ContentProvider中获取数据。
{% endhighlight %}

### 获取 Content Provider 中的数据变化通知

#### Content Provider 可以通过调用 getContext().getContentResolver().notifyChange(uri, null); 方法来通知其他对其感兴趣的应用程序它的数据发生了变化。

#### 当一个Android应用程序需要对另一个应用程序中的 Content Provider 数据变化做出反应时，可以：
1. 定义一个继承自 ContentObserver 的类型，实现其 onChange() 方法。
2. 调用 getContentResolver().registerContentObserver() 方法来和其注册。

#### 这样，就可以在 Content Provider 共享的数据发生变化的时候，通过 onChange() 函数被自动调用而做出相应回应。

#### 和之前一样，还是做一个应用程序来练习一下自定义 Content Provider 的使用以及如何处理数据变化后的通知。例子规模不大，注释充分，就不贴代码了，工程在[这里][1]

[1]: https://github.com/xhrwang/ContentProviderDemo


