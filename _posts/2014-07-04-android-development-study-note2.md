---
layout: post
title: "安卓开发学习笔记2-Android应用工程及其对localization，compatibility的支持"
description: ""
category: [Android]
---

#### Android是Google开源的智能设备操作系统，使用Android的厂商非常多，推出的机型更是数不胜数，不同的硬件配置带来了多样的屏幕尺寸和分辨率，全球范围的大规模使用带来了localization和globalization的要求。Android开发环境对这些提供了足够的支持，让开发者有能力设计开发出兼容不同设备，不同安卓版本的应用。

#### 本文要记录的是Android应用工程目录结构及其对localization和compatiility的支持。

### Android工程目录结构

#### Android工程将布局，菜单，字符串，图片资源等界面元素从代码中剥离，放入特定的目录中，从而提供对localization和compatibility的支持。关于如何搭建Android开发环境以及如何创建一个安卓工程，[官网提供了详细的步骤和说明][3]，本文就不介绍了，这里以官网提供的[Sample project][1]为例。下面是Android Studio（左）和Eclipse ADT（右）中Android工程的目录结构：
![][androidproj]

* Android Studio用Gradle来build安卓工程，Eclipse ADT用Apache Ant来build安卓工程，所以二者的目录结构有不同的配置文件。
* 安卓应用的代码，资源，依赖库以及manifest文件两个IDE都有，只是组织形式不一样。
	1. libs目录中是安卓应用工程的依赖库，NDK开发时编译的C/C++动态库也会放在这里。
	2. Android Studio中的 app->src->main->java目录中是应用程序java代码，这些代码在Eclipse ADT中存在于src目录中。
	3. res目录中存放着安卓工程的资源文件，包括布局，矢量图，样式等。
	4. Android Studio的build目录，Eclipse ADT的bin目录中是自动生成的java代码。
	5. AnroidManifest.xml文件中定义了安卓应用的配置信息，包括支持的安卓版本，运行所需权限，样式，用到的组件，入口activity等。
	6. 更多详细解释可以参考[官网说明][2]


### Android开发中的localization

#### 当Android应用定位于不同国家和地区的用户时，我们需要确定有哪些语言需要支持，并提供不同的货币单位，时间日期格式等以方便不同地区的用户使用。应用界面上显示的字符串和style定义在res->values目录中，为了支持localization，Android工程允许我们建立不同的values目录，名称为“values-_ISO language code_”，ISO language code为ISO定义的不同语言的代号，比如中文为zh，西班牙语为es，不加的时候默认为English。在应用安装到安卓设备后运行时，安卓系统会根据当前的语言设置使用相应的目录中定义的值，如果没有定义则使用valuse目录中的定义。

#### [官网说明][4]如下：
![][localization]

### Android开发中对不同Android系统版本的支持

#### 在应用的manifest配置文件AndroidManifest.xml中有如下的定义：

{% highlight xml %}
<uses-sdk android:minSdkVersion="14" android:targetSdkVersion="19" />
{% endhighlight %}

#### 这段配置信息说明该应用最低支持运行在BuildAPI为14，也就是4.0系统上，使用BuildAPI为19的SDK进行编译。Google建议开发者尽量使用最新版本SDK进行编译，这让开发者能使用最新的api，并尽可能支持多个安卓系统版本，因为用户群体中可能会有很多人还在使用较低的安卓版本。

### Android开发中对不同硬件设备的优化

#### 针对不同的硬件设备（屏幕尺寸和分辨率不同）提供相应的用户界面能让我们的应用在不同的设备上获得最好的用户体验。Android将屏幕大致分为small,normal，large和xlarge，相应的分辨率分为low dpi, medium dpi, high dpi以及extra high dpi。工程目录下的res目录中针对不同分辨率有不同的矢量图文件夹：
![][drawable]

#### 为了获得更好的体验，应用在安卓设备处于竖屏和横屏时也应该有不同的布局来适应，所以安卓工程允许开发者定义不同的layout，具体可参考下面的图片：
![][layout]

##### 上面的两张图片来自[Android开发者官网页面][5]

#### 本文主要记录了安卓应用工程的目录结构以及其对localization，compatibility的支持，在开发一个安卓应用前，对这些知识有所了解是必要的，有助于在应用开发的早期设计中考虑到这些因素。

[1]: http://developer.android.com/shareables/training/ActivityLifecycle.zip
[2]: http://developer.android.com/training/basics/firstapp/running-app.html
[3]: http://developer.android.com/training/basics/firstapp/index.html
[4]: http://developer.android.com/training/basics/supporting-devices/languages.html
[5]: http://developer.android.com/training/basics/supporting-devices/screens.html
[androidproj]: /images/androidproj.png
[localization]: /images/localization.png
[drawable]: /images/pictures.png
[layout]: /images/layout.png


