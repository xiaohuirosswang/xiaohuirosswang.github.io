---
layout: post
title: "安卓开发学习笔记1-Android系统架构和启动流程"
description: ""
category: [Android]
---

#### 开始着手系统学习安卓开发，决定记录一下，一方面通过blog的形式自我督促，另一方面也通过整理思路来巩固所学到的知识点。

#### 之前收集学习资料的时候，看到一张[安卓开发学习路线图][1]，很有启发：
1. 基础 - Java编程语言，XML解析，Linux基础，数据库基础等。
2. 入门 - Android系统架构，开发环境搭建，Android应用程序工程结构，应用布局，四大组件特性以及数据访问和存储等。
3. 进阶 - Activity，Service，Broadcast receiver和content provider的熟练合理运用，多线程控制，多媒体管理，进程间通信以及高级控件的使用。
4. 高级 - 自定义样式和主题，animation，2D/3D绘图，传感器应用，NDK开发，localization，不同设备不同版本的compatibility等


#### 在学习一个平台的开发之前，对这个平台本身做一些了解是必要的，不仅有助于理解该平台开发的特点，也有利于在之后的开发过程中针对平台本身的特点进行设计。

### Android系统架构

#### [Android官网的系统架构图][2]如下：
![][sysarch]


#### 参考官网的解释，安卓系统可以分为下面几层：
1. Linux内核 - Android系统运行的基础，提供基本文件系统，硬件驱动以及内存和进程管理。
2. 硬件抽象层（HAL） - 一个允许Android系统调用设备硬件的标准接口，硬件厂商可以在保证满足HAL的基础上针对设备硬件进行适合自己的优化。
3. 系统服务层 - 由系统类服务（处理Activity manager，Window manager以及Notification manager等）和媒体类服务（提供多媒体的记录和播放等）组成，这些服务都是专注于某一个特定功能的模块化组件，便于Application framework API调用。
4. 实现进程间通信IPC的Binder - 通过进程间通信封装了application framework和各类系统服务通信的细节，开发者可以专注在应用功能的开发或者说如何用好应用框架层提供的API。
5. 应用框架层 - 应用框架层通过API提供了对HAL的控制，开发者开发的应用大部分基于这些API，并通过他们和设备硬件进行交互。

##### Google在 I/O 2014大会上推出Android L Release之后宣布，新版本系统将会完全用ART取代Dalvik，以获得更好的性能和电池寿命，同时提供更有效率的垃圾回收功能。同时这张官网的系统架构图也没有了Dalvik VM的介绍。

### Android系统启动流程

#### Android L Release使用ART的启动流程应该和现在大家熟悉的流程有所不同，相信不久后专注Android第三方ROM开发的朋友们会对其有系统的讨论，现在，对于学习安卓应用开发而言，先了解一下基于Dalvik VM的安卓系统启动流程。

#### Android系统是基于Linux内核的，其启动前期的流程和Linux系统启动流程是类似的，借用[小米英文官网的一张图][3]：
![][bootprocess]

1. 按下电源开机后，安卓设备会从ROM中特定的分区加载系统引导程序Bootloader。
2. 系统引导程序由硬件厂商提供，首先针对CPU和主板进行初始化，检测外部RAM并为下一步做一些准备工作；然后引导程序会初始化网络连接，内存等加载Linux内核时需要的任务，这时可以准备加载内核时的一些配置和参数。
3. 准备工作完成后，引导程序开始加载系统内核，并启动第一个安卓系统进程init
4. init启动后会挂载文件系统目录，度init.rc配置脚本，该脚本由安卓初始化语言定义，包含Actions，Commands，Services以及Options。根据配置会启动管理USB连接的usbd服务，我们熟悉的adbd，以及最后会启动两个重要的进程，zygote和runtime进程，runtime将初始化service manager，用来提供Binder services的注册和查找，完成后runtime将请求zygote启动system service，这时zygote将会孵化出第一个Dalvik实例并启动system service，system service会启动两个native服务Surface flinger和Audio Flinger，这两个服务会在service manger注册提供IPC服务；之后，system service会启动application framework API与之通信的安卓系统管理服务。
![][codeboot]
5. 安卓系统服务启动后，安卓系统启动流程就结束了，系统将广播“ACTION_BOOT_COMPLETED”并启动Home activity。

#### 这里需要引用几篇关于安卓启动流程的好文章，这些文章从不同的角度讲解了这一流程，对我们理解安卓系统从加电启动到系统初始化完成所作的事情很有帮助：

* [Android启动过程深入解析][4]
* [Android系统启动流程][5]
* [Android系统启动过程剖析][6]
* [Understanding Android Boot Process][7]

#### 本文主要整理了安卓系统架构和安卓系统启动流程，如文章最前面所说，了解这些有助于我们更好得在安卓平台开发应用程序，这些也是理解安卓应用程序中关键组件比如activity，service，broadcast receiver以及content provider如何工作以及各自生命周期的基础。


[1]: http://androidtoast.iteye.com/blog/1151917
[2]: https://source.android.com/devices/index.html
[3]: http://en.miui.com/thread-15659-1-1.html
[4]: http://blog.jobbole.com/67931/
[5]: http://www.cnblogs.com/wiikii-/archive/2013/04/14/wiikii.html
[6]: http://mobile.51cto.com/hot-285155.htm
[7]: http://en.miui.com/thread-15659-1-1.html
[sysarch]: /images/system-architecture.png
[bootprocess]: /images/bootprocess.jpg
[codeboot]: /images/init.gif

