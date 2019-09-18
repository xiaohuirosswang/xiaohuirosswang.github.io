---
layout: post
title: "Android 开发中的 Event Driven"
description: ""
category: [Android]
---

# Why

[Decoupling your Android code][1] 这篇文章提到：

> Android as a platform offers a lot of different components to be used around. In a single Activity you might find yourself with a stack of Fragments, Views, Model data, adapters, Action bar actions, View actions (click), context menus and more.   
Aiming to combine them all in an elegant way that will guarantee you with flexible and modular code will require you to “decouple” them from each other. That way, you will have the opportunity to re-use them, and make future changes easier.

[Decoupling Android App Communication with Otto][2] 这篇文章中同样提到：

> As Android applications increase in complexity, ensuring effective communication between different parts becomes more and more difficult.

> In order to find an elegant solution to this problem, a technique is borrowed from an unexpected place: Swing applications. The event bus pattern—also known as message bus or publisher/subscriber model—allows for the communication of two components to occur without either of them being immediately aware of the other.

这两篇文章都指出在 Android 应用开发中，随着应用复杂度提高，模块增多，如果使用 `Listener`、`BroadcastReceiver` 以及只在应用内部传递信息的 `LocalBroadcastReceiver` 等组件间的通信方式，耦合度也会随之增大。为了降低代码的耦合度，使用 __Event Bus__ 是很好的选择。

__Event Bus__ 模式，也被称为 `Message Bus` 或者 `publisher/subscriber` 模型，相互通信的组件间无需知道对方的存在：

![](/images/eventbus.png)

# What

针对 Android 平台的 Event Bus 实现比较热门的有以下两个开源项目：

- [square/otto][3] 
- [greenrobot/EventBus][4]

__otto__ 在 [Google guava][5] 的基础上，适配了 Android 平台，提供了基于反射和 Annotation 的事件发布/订阅机制，默认工作在 `Main UI` 线程。

__EventBus__ 基于反射和方法命名约定也提供了事件发布/订阅机制，但是针对 Android 平台提供了深度定制，支持 `Sticky Event`，并提供了多种工作线程模式，不再限制在 `Main UI` 线程。[这篇文档][5] 给出了详细的解释。__Event Bus__ 官方提供了一个 [实例 App][6] 对 __otto__ 和 __Event Bus__ 的性能进行了比较，得出性能更好的结论，考虑到其提供了针对 Android 平台的更多功能，得到了更多开发者的青睐。

[关于二者功能和性能的对比表][7]：

![](/images/eventbug_compare1.png)

# How

## Otto

如果应用规模较小，且无需复杂线程间通信，__otto__ 是个很好的选择，其使用 Annotation 让发布和订阅事件更方便，相比 __Event Bus__ 依靠命名约定更不易出错。

将 __otto__ 和传感器数据收集模块集成到 SDK 的工程已经提交到：[xiaohui.wang/SensorDemo][8]

由于 __otto__ 的代码非常精简，规模很小，很容易读懂，[EventBus vs Otto vs Guava][9] 这篇文章参考其实现了一个更轻量级的 __Event Bus__，是很好的参考。

[Vogella 网站有一个 End to End 的教程][10] 全方位介绍了 otto 在项目中的使用

## Event Bus

如果应用功能较多，组件间通信比较复杂，那么 __Event Bus__ 是更好的选择，其提供的工作在不同线程的支持，并且有 `Sticky Event`，可以相当大程度高效替换 BroadcastReceiver，提供更安全和耦合度更低实现。（[这篇文章][11] 详细介绍了 `Intent` 的 `extra` 可能存在的运行时类型安全隐患，特别是在项目规模较大，需要多人协作时可能造成的维护成本增加）。

下面两个系列博文对本文讨论的 Android 平台上的 __Event Driven__ 模式都进行了深度讨论并提供了使用 __Event Bus__ 进行解耦的实例项目，并且都给出了实例代码的 Github 链接：

- Using an EventBus in Android
	- [Why an EventBus?](http://blog.cainwong.com/using-an-eventbus-in-android-pt-1-why-an-eventbus/)
	- [Sticking Your Config](http://blog.cainwong.com/using-an-eventbus-in-android-pt-2-sticking-your-config/)
	- [Multi-Threaded Event Handling](http://blog.cainwong.com/using-an-eventbus-in-android-pt-3-threading/)

- Event-driven programming for Android 
	- [Part 1](https://medium.com/google-developer-experts/event-driven-programming-for-android-part-i-f5ea4a3c4eab)
	- [Part 2](https://medium.com/google-developer-experts/event-driven-programming-for-android-part-ii-b1e05698e440)
	- [Part 3](https://medium.com/google-developer-experts/event-driven-programming-for-android-part-iii-3a2e68c3faa4)

## Event Driven Architecture

[Google Developer Experts Group 的这篇文章][12] 用一个项目实例介绍了如何在 Android 应用程序开发中应用 __Event Driven__ 模式，对应用功能进行模块化组织，使用 __Event Bus__ 管理模块间信息传递从而达到解耦提高可扩展和可维护性的目的。



[1]: http://blog.android-develop.com/2014/03/decoupling-your-android-code.html
[2]: http://corner.squareup.com/2012/07/otto.html
[3]: https://github.com/square/otto
[4]: https://github.com/greenrobot/EventBus
[5]: https://github.com/greenrobot/EventBus/blob/master/HOWTO.md
[6]: https://github.com/greenrobot/EventBus/tree/master/EventBusPerformance
[7]: https://raw.githubusercontent.com/greenrobot/EventBus/master/COMPARISON.md
[8]: http://gitlab.tenddata.com/xiaohui.wang/sensordemo
[9]: http://avenwu.net/ioc/2015/01/29/custom_eventbus/
[10]: http://www.vogella.com/tutorials/JavaLibrary-EventBusOtto/article.html
[11]: http://blog.cainwong.com/using-an-eventbus-in-android-pt-1-why-an-eventbus/
[12]: https://medium.com/google-developer-experts/event-driven-programming-for-android-part-iii-3a2e68c3faa4



