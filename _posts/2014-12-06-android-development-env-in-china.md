---
layout: post
title: "安卓开发之在国内给 Eclipse 加速"
description: ""
category: [Android]
---

## 缘由

在中国大陆的安卓开发者是郁闷的，因为 GFW 的存在，我们无法顺畅地访问 Google Android Development 相关站点的资源，遇到问题也无法求教于 Google，StackOverflow 也间歇式抽风。基本上自备梯子已经是成熟安卓开发者的标配了。但是，当我们更新 Android SDK 的时候，经过代理的网速有时候慢得完全能毁掉一天的好心情。这篇文章主要整理一下如何通过设置 DNS 缓存以及对 Eclipse 的设置来在国内加速更新。同时介绍一下如何解决 Google 提供的 Eclipse ADT 带来的问题。

## Eclipse ADT Windows 版本的问题

之前遇到一个问题，从 [Android 官网][1] 下载 Eclipse ADT 后，下载最新的 Android Build Tools 和 SDK，新建一个安卓应用程序并选择 blank Activity 的时候会发现没有任何 Activity 被创建，在工程中新建 Activity 也没有效果。这件事情发生在 Google I/O 2014 发布 Android L 的时候，让满心欢喜准备体验新版安卓应用开发的我非常郁闷。最后在 [StackOverflow][2] 上得到这是 Google 的问题的确认，非常纳闷 Google 怎能如此大意，而且在得到开发者反馈后也没有及时更新让我更是不解，只能转而使用 Beta 版的 Android Studio，由于其尚未支持 NDK 开发，所以刚开始的时候还是两个配合着用的，没过多久实在郁闷，决心解决这个问题，上网找方案，[发现不只我一个遇到这个问题][3]，解决办法最终采用安装完整 Eclipse Luna 加 ADT plugin。

__注意：这里默认JDK环境已经安装配置完成，这部分不是本文目的所在。请参考 [这篇文章][4]__

## 下载并安装 Eclipse Luna

从 [Eclipse 官网][5] 下载 [Luna 版本][6] 并安装。

## 修改 Windows 平台的 hosts 文件为 Google Android Developer 站点进行 DNS 缓存，参考 [这篇文章][10]

- 找到并打开 C:\Windows\System32\drivers\etc\hosts 文件。
- 将下面的内容添加到文件末尾

{% highlight sh %}
# 这行是为了方便打开Android开发官网
74.125.113.121 developer.android.com
# 更新的内容从以下地址下载
203.208.46.146 dl.google.com
203.208.46.146 dl-ssl.google.com
{% endhighlight %}

- 保存并关闭 hosts 文件，可能需要进行授权。

## 安装 ADT plugin

现在打开 Eclipse Luna，打开 Help -> Install new software，在 Work With 里面输入“https://dl-ssl.google.com/android/eclipse/”，等刷新完成后，安装所有项。详细步骤可以参考 [官方文档][7]

安装完成后重启 Eclipse Luna，从 Windows -> Android SDK Manager，在弹出窗口中，点击 Tools -> Option，在 settings 窗口里面选中 “Force https://... sources to be fetched using http://...” 的 CheckBox，然后关闭窗口。现在刷新获取 SDK 资源速度会非常快，我用的是 20MB 的宽带，下载速度接近 3MB/s。

## 新建 Android Application 工程

现在重新去创建一个包含 blank Activity 的 Android Application 工程，一切正常，终于可以欢乐地搬砖了。

## 一些对于 Eclipse 的配置

事情到这里还没结束，当我们尝试导出 APK 的时候，会出现一些错误，因为这个 Eclipse 版本是针对 JAVA 开发者的，有些设置对 Android 开发这来说需要修改。

问题1，出现下面的错误：

> "app_name" is not translated in af, am, ar, be, bg, ca, cs, da, de, el, en-rGB, en-rIN, es, es-rUS, et, et-rEE, fa, fi, fr, fr-rCA, hi, hr, hu, hy-rAM, in, it, iw, ja, ka-rGE, km-rKH, ko, lo-rLA, lt, lv, mn-rMN, ms, ms-rMY, nb, nl, pl, pt, pt-rBR, pt-rPT, ro, ru, sk, sl, sr, sv, sw, th, tl, tr, uk, vi, zh-rCN, zh-rHK, zh-rTW, zu

[解决方案][8] 如下：

> In your ADT go to window->Preferences->Android->Lint Error Checking, Find there Missing Translation and change its Severity to Warning. That's it :)

问题2，出现下面的错误：

> This class should be public (android.support.v7.internal.widget.ActionBarView.HomeView)

[解决方案][9] 如下：

> On the "v7-appcompat" library: preferences -> Android Lint Preferences, Search for "Instantiatable" and set to Warning.

好了，现在可以正常导出安装包了。如果遇到其他类似奇怪的问题，需要我们在网上确认不会有潜在风险后用类似的方式解决。

## 最后

本文不是 Windows 平台 Android 开发环境搭建的教程，只是一些让我的生活更舒服的经验。


[1]: http://developer.android.com/sdk/index.html
[2]: http://stackoverflow.com/questions/24595811/unable-to-create-activity-in-eclipse
[3]: https://code.google.com/p/android/issues/detail?id=66647
[4]: http://stackoverflow.com/questions/8578441/can-the-android-sdk-work-with-jdk-1-7
[5]: http://www.eclipse.org/
[6]: http://124.205.69.131/files/924300000000D7E0/mirror.neu.edu.cn/eclipse/technology/epp/downloads/release/luna/SR1/eclipse-java-luna-SR1-win32-x86_64.zip
[7]: http://developer.android.com/sdk/installing/installing-adt.html
[8]: http://stackoverflow.com/questions/21118725/error-app-name-is-not-translated-in-af
[9]: http://stackoverflow.com/questions/22821420/this-class-should-be-public-android-support-v7-internal-widget-actionbarview-ho
[10]: http://blog.sina.com.cn/s/blog_4a94a0db0100y4h7.html


