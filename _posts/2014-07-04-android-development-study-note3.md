---
layout: post
title: "安卓开发学习笔记3-Android应用开发中的布局"
description: ""
category: 安卓开发
tags: [Android]
---

### Android应用开发中的布局

#### LinearLayout - 线性布局，根据设置的horizontal或者vetical依次摆放控件。下面的例子展示了使用android:weight定义的比例来定义控件布局的demo，包括水平和垂直方式的布局。

{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".LinearLayoutActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal">
        <View
            android:background="#aa0000"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"/>
        <View
            android:background="#00aa00"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="2"/>
        <View
            android:background="#0000aa"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="3"/>
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="2"
        android:orientation="vertical">
        <View
            android:background="#550000"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"/>
        <View
            android:background="#005500"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="2"/>
        <View
            android:background="#000055"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="3"/>

    </LinearLayout>

</LinearLayout>
{% endhighlight %}

* 在整体布局采用垂直分布，其中中嵌套了两个LinearLayout。第一个采用水平分布，第二个采用垂直分布。
* 这两个子LinearLayout按照layout_weight定义的比例获得屏幕高度分配。
* 第一个子LinearLayout中每个View设置了“android:layout_width="0dp"”和“android:layout_weight="1"”两个属性，也就是在屏幕宽度范围内，按照ayout_weight定义的比例为每个View分配宽度。
* 第二个子LinearLayout中每个View设置了“android:layout_height="0dp"”和“android:layout_weight="2"”两个属性，也就是说在自己的屏幕高度范围内，按照layout_weight定义的比例为每个View分配高度。
* 效果图如下：

* ![][linearlayout]

#### RelativeLayout - 相对布局，控件可以根据之前定义控件的位置采相对地确定自己的位置。下面的例子展示了使用android:layout_below和android:layout_toRightOf定义控件相对位置的demo。

{% highlight xml %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".RelativeLayoutActivity">

    <View
        android:id="@+id/v1_View"
        android:layout_width="200dp"
        android:layout_height="100dp"
        android:background="#aa0000"/>
    <View
        android:id="@+id/v2_View"
        android:layout_width="wrap_content"
        android:layout_height="100dp"
        android:background="#00aa00"
        android:layout_toRightOf="@id/v1_View"/>
    <View
        android:id="@+id/v3_View"
        android:layout_width="wrap_content"
        android:layout_height="100dp"
        android:background="#0000aa"
        android:layout_below="@id/v1_View"  />

</RelativeLayout>
{% endhighlight %}

* layout_below属性定义了该控件在那个控件的下面。
* layout_toRightOf属性定义了该控件在那个控件的右边。
* 效果图如下：

* ![][relativelayout]

#### TableLayout - 表格布局，使用表格的形式组织和展示控件。下面的例子展示了一个简单的表格形式。

{% highlight xml %}
<TableLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:stretchColumns="1"
    tools:context=".TableLayoutActivity">
    <TableRow>
        <View
            android:layout_height="100dp"
            android:layout_width="200dp"
            android:background="#aa0000"
            />
        <TextView
            android:layout_height="100dp"
            android:text="#aa0000"
            android:gravity="center"/>
    </TableRow>
    <View
        android:layout_height="2dip"
        android:background="#000000" />
    <TableRow>
        <View
            android:layout_height="100dp"
            android:layout_width="200dp"
            android:background="#00aa00"
            />
        <TextView
            android:layout_height="100dp"
            android:text="#00aa00"
            android:gravity="center"/>
    </TableRow>
    <View
        android:layout_height="2dip"
        android:background="#000000" />
    <TableRow>
        <View
            android:layout_height="100dp"
            android:layout_width="200dp"
            android:background="#0000aa"
            />
        <TextView
            android:layout_height="100dp"
            android:text="#0000aa"
            android:gravity="center"/>
    </TableRow>
</TableLayout>
{% endhighlight %}

* 定义了三行，每行有两个控件，各行之间有一个分割View
* 效果图如下：

* ![][tablelayout]

#### FrameLayout - 单帧布局，申请一块空白区域，控件可以按照定义层次展示出来，下面的例子展示了如何在应用使用单帧布局。

{% highlight xml %}
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:background="#000000"
    tools:context=".FrameLayoutActivity">
    >
    <ImageView
        android:id="@+id/one_imageview"
        android:src="@drawable/ic_action_search"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        />
    <ImageView
        android:id="@+id/two_imageview"
        android:src="@drawable/ic_action_favorite"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="left"
        />
</FrameLayout>  
{% endhighlight %}

* 在屏幕的左边和右边分别定义了一个图片View，以层叠方式显示。
* 效果图如下：

* ![][framelayout]


#### 为了熟悉和理解这些布局模式，最好的办法是了解他们之间的区别之后，写个应用来巩固一下。完整的应用工程可以从[这里下载][1]，在这个工程中，也应用到了本系列第2篇博文中讨论的localization和compatibility方面的支持。

[1]: https://github.com/xhrwang/AndroidLayoutDemo
[linearlayout]: /images/linearlayout.png
[relativelayout]: /images/relativelayout.png
[tablelayout]: /images/tablelayout.png
[framelayout]: /images/framelayout.png

