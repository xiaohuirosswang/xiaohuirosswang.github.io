---
layout: post
title: "Apple Watch!"
description: ""
category: [Apple Watch]
---

![](/images/apple_watch_size_fixed.0.jpeg)

公司在 Apple Watch 发售后很快搞到一块用来测试适配，故而有机会第一时间适用这款无论从工业设计还是软件系统方面为 Smart Watch 树立全新标杆的产品。这篇文章就通过实际使用经验结合各路测评报告谈谈 Apple Watch。

### Apple Watch 首次启动配置

1. 充电，全新 Apple Watch 开箱后电池电量可能会比较低。
2. 将 Apple Watch 和 iPhone （iPhone 5 及更新）进行配对。
	- 选择系统语言
	- 提示打开 iPhone 上的 Apple Watch 应用进行配对
	- 选择佩戴在左手还是右手（可能和判断手腕抬起时电量屏幕时传感器数据的处理有关）
	- 条款和条件确认
	- 输入 Apple Id
	- 定位和 Siri 服务确认，无法不选择，但是定位服务在之后可以从 iPhone 中禁用
	- 设置 Apple Watch 密码，至此，无法退回之前设置，只能恢复出厂设置
	- 根据已经安装在 iPhone 上的应用程序列表自动在 Apple Watch 上安装 Watch App，可以跳过第三方应用
3. Force Touch，也就是用力按屏幕将进入 Watch Face 选择界面，有很多选择。
4. 在 iPhone 上的 Watch App 中设置安装哪些第三方程序的 Watch 版本。
5. 在 iPhone 上的 Watch App 的 “健康” 选项中设置性别、年龄、体重以及预期的卡路里消耗量。
6. 在 iPhone 上的 Watch App 中浏览并安装第三方支持 Apple Watch 的应用。
7. 在 iPhone 上的 Watch App 中的 “朋友” 选项中最多添加 12 个人可以从 Apple Watch 上打电话。

图文并茂的文章：[How to set up the Apple Watch in 16 steps](http://www.theverge.com/2015/4/24/8489459/apple-watch-how-to-pair-setup-pay-bluetooth-contacts)

### Apple Watch 是如何工作的

1. 首先，需要配合一部 iPhone （iPhone5 及更新）使用，代码在运行在 iPhone 上的 WatchKit Extension 上执行，Watch Apps 包含界面设计部分。[Apple 声称](https://developer.apple.com/watchkit/)：

	> WatchKit apps have two parts: A WatchKit extension that runs on iPhone and a set of user interface resources that are installed on Apple Watch. When your app is launched on Apple Watch, the WatchKit extension on iPhone runs in the background to update the user interface and respond to user interactions. 

2. 当前 Watch Apps 提供三种形式：
	- Apple Watch 独有交互风格的 App界面
	- Glances，时效性只读 Message
	- Actionable Notifications，用户可以交互的通知
3. Apple 声称 Watch App 让 iOS 应用更完整，一些几秒钟就可以完成的交互适合在 Watch App 中实现：

	> A Watch app complements your iOS app; it does not replace it. If you measure interactions with your iOS app in minutes, you can expect interactions with your Watch app to be measured in seconds.

4. 完全独立在 Apple Watch 上运行的应用可能会在明年晚些时候支持，考虑到电量和屏幕大小，这种 App 可以预期会有很多限制，但是总归可以独立于 iPhone 运行了：

	> Starting later next year, developers will be able to create fully native apps for Apple Watch.

5. 目前提供两种分辨率（Retina）的 Watch，但是 Apple 提到了 Watch 界面渲染方式和 iPhone 上的不同之处, WatchKit 界面元素渲染方式类似于响应式网页，可能苹果以后会推出不同分辨率的 Watch：

	> Unlike iOS, where you place views at a coordinate on the screen,with WatchKit, objects automatically flow downward from the top left corner of the screen, filling the available space.  
	
	- 38mm，w：272px，h：340px
	- 42mm，w：312px，h：390px

6. 对于通知来说，有两种：`Short Look` and `Long Look`，前者只显示应用图标、名称以及摘要信息；后者在检测用户阅读 `Short Look` 一段时间后自动出现，应用图标和名称将自动滑向顶部，用户可以下滑看到更多信息，并通过按钮进行交互：
7. Glance 只能显示单个屏幕能够显示的只读信息，不过用户可以通过点击启动 iPhone 上的 App。
8. `Force Touch` （用力长按）可以弹出最多 4 个 Action 的按钮
9. 地图界面是静态的，无法放大缩小，只能显示最多 5 个图钉的注解，点击会启动 iPhone 上的 Apple Map，但是 Watch 上可以进行导航。
10. Watch 上每个应用可以最多缓存 20MB 的图片资源，其他的只能放在 Extension 中，视频不支持。
11. Watch 上的相机应用可以控制快门，但是无法选择前置摄像头，估计觉得没必要。
12. 微信朋友圈也可以在 Watch 上浏览，图片可以全屏显示，不支持微视频。可以评论朋友圈其他同学的分享，但是 Siri 的中文语音识别真的不敢恭维，而且识别后就自动发出去了，没有机会修改。
13. 应用排列可以在 iPhone 的 Watch App 中调整。
14. 来电或者 Facetime 视频通话被叫时，手表会震动提醒，并且可以从 Watch 上确认接还是不接，在选择不接时很方便无需拿出手机。
15. 运行健康的优先级很高，在抬起手腕的默认显示中就有，随时提醒，而且制定卡路里消耗计划后，Watch 会按照自己的算法在合适的时间提醒用户做做运动，比如，每隔一个小时就会提醒用户站一会。


图文并茂的文章：[11 things we just learned about how the Apple Watch works](http://www.theverge.com/2014/11/18/7243085/most-important-apple-watchkit-discoveries)

[其实这里有张图片总结的就很好](http://www.mutualmobile.com/posts/apple-watch-design-overview-infographic)：

![](/images/3077359519662398946.jpg)

开发者的一些经验：

- [Apple Watch Programming Guide](https://developer.apple.com/library/ios/documentation/General/Conceptual/WatchKitProgrammingGuide/iOSSupport.html#//apple_ref/doc/uid/TP40014969-CH21-SW1) 
- [Apple Watch 应用开发有哪些注意事项？](http://www.zhihu.com/question/29446492)
- [Apple Watch两个月开发的一些收获总结](http://www.infoq.com/cn/articles/watch-app-development)

美国市场对于可穿戴设备的调查结果：[Do Consumers Even Know What They Want from Wearables?](http://www.emarketer.com/Article/Do-Consumers-Even-Know-What-They-Want-Wearables/1012403/1)

当然，华强北山寨版早已就绪，而且很逆天：[“Apple Watch”评测——华强北版](http://www.qdaily.com/display/articles/8240.html)


