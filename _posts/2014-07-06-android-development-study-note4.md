---
layout: post
title: "安卓开发学习笔记4-Android应用四大组件初探"
description: ""
category: [Android]
---

### Android应用及其特点

#### [官网关于Android应用][1]是这样说的：
> Android apps are built as a combination of distinct components that can be invoked individually. For instance, an individual __activity__ provides a single screen for a user interface, and a __service__ independently performs work in the background.

> From one component you can start another component using an __intent__. You can even start a component in a different app, such an __activity__ in a maps app to show an address. This model provides multiple entry points for a single app and allows any app to behave as a user's "default" for an action that other apps may invoke.

#### 可以这样理解：Android应用是由可以被单独调用的相互独立的组件构成的，可以有多个入口，一个应用可以通过合理的方式方便地调用另一个应用中的组件。

#### [官网关于Android应用的运行方式][2]是这样说的：
> Once installed on a device, each Android app lives in its own security sandbox:

> The Android operating system is a multi-user Linux system in which each app is a different user.

> By default, the system assigns each app a unique Linux user ID (the ID is used only by the system and is unknown to the app). The system sets permissions for all the files in an app so that only the user ID assigned to that app can access them.

> Each process has its own virtual machine (VM), so an app's code runs in isolation from other apps.

> By default, every app runs in its own Linux process. Android starts the process when any of the app's components need to be executed, then shuts down the process when it's no longer needed or when the system must recover memory for other apps.

#### 也就是说，Android系统基于多用户Linux操作系统，默认情况下，每个应用有自己的UserID，运行于独立的Linux进程中，在自己的某个组件被启动的时候，Android系统会启动该应用的进程，并在应用退出后或者资源紧张的时候根据需要销魂该进程。Android应用是运行在隔离的安全沙盒模式下的。

#### [官网关于Android应用的组件][3]是这样说的：
> App components are the essential building blocks of an Android app. Each component is a different point through which the system can enter your app. Not all components are actual entry points for the user and some depend on each other, but each one exists as its own entity and plays a specific role—each one is a unique building block that helps define your app's overall behavior.

#### 应用组件是Android应用的基本编译单元，每个组件都为Android系统提供了一个启动应用的入口，一些应用组件只能被其他组件启动，而非直接可以被用户启动。但是每个应用组件都作为各自的入口提供一个特定的功能。所有这些组件组合在一起定义了该应用能做什么和怎么做。

### Android应用的四大组件

#### Android应用有四个不同的组件，每种组件提供了不同的功能并有各自不同的生命周期，定义了如何被创建、使用和销毁。
* __Activities__ - 每个Activity都是一个独立的可以和用户交互的界面，一个或者多个Activity组合起来以提供一个特定的功能，应用可以允许自己的那些Activity可以被其他应用所调用。
* __Service__ - 一个Service是没有界面的、运行在后台来执行需要长时间操作的任务或者和其他进程进行通信的组件。Service组件可以被其他组件如Activity所启动来完成特定的任务或者被绑定后进行数据交互。
* __Broadcast receiver__ - 一个Broadcast receiver是用来响应系统范围广播通知的组件。系统自身会发出很多广播通知，比如点亮或者熄灭屏幕等，应用自己也可以发送一个自定义的广播通知，并定义广播方式。广播接收器可以有自己的filter来过滤自己想处理的通知，虽然没有界面，但接收器勀在通知栏中发一个新的状态来通知用户某一个通知事件已经发生。广播接收器只应该做少量的工作，其定位就是一个类似网关的角色，决定当某个事件发生时应该如何响应；对于需要较长时间的操作，可以启动一个Service，或者在需要用户参与的场景中，启动一个Activity。
* __Content provider__ - 内容提供者管理者应用数据的共享状态，通过它提供的统一接口，我们可以查询，修改，添加或者删除存放在各种媒介的相应的数据。当然，内容提供者也可以读写只供该应用访问的数据。

#### 由于前面介绍的Android应用的安全性限制，一个应用无法直接启动另一个应用的组件，只能通过发送相应的Intent给Android系统，Android系统再根据Intent的内容启动特定的应用组件。

#### 本文记录了Android应用及其运行方式，还有Android应用的四大组件，在后续的blog文章中会对每一个组件的使用方式及其生命周期进行讨论。

[1]: http://developer.android.com/guide/index.html
[2]: http://developer.android.com/guide/components/fundamentals.html
[3]: http://developer.android.com/guide/components/fundamentals.html


