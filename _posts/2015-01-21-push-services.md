---
layout: post
title: "iOS, Android 和 Windows 的 Push Services"
description: ""
category: [App]
---

## Why

iOS，Android 和 Windwos 做为现在比较流行的移动操作系统，都提供了自己的 Push Services：

- [ANS][1] Apple Notification Service
- [GCM][2] Google Cloud Messaging
- [MPNS][3]/[WNS][4] Microsoft Push Notification Service/Windows Notification Service

由系统提供商建立统一的 Push Service 机制， 可以获得下面这些好处：

- 更好的安全性。所有的 Push 都来自于认证过的官方 Push Server
- 更稳定，更可靠。Apple，Google 和 Microsoft 提供统一的 Notification 的 “网关”服务器，大公司做这些更可靠。
- 更省电。不需要每个应用自己轮询去获取 Notification，保持后台进程运行。
- 更高效。很显然这样的机制减少了开发者的负担，开发者只需要调用系统提供商的 Notification 接口就好。

这里需要提一下的是，由于 Android 系统本身不对应用自己管理 Push Notifiation 做强制要求，很多应用为了更灵活地接收通知，都自己直接和自己的 Push Server 通信来获取 Notification，这样自然保持了很多后台常驻进程，造成了系统资源浪费以及耗电更快等问题。国内由于 Google 的服务无法稳定访问，而且国内 App 开发商基本都对应用权限做了超过应用本身需求的申请，再加上通过对 Broadcast Receiver 机制的滥用，结合在一起造成了 Android 设备的很多使用过程中的问题。

## What

下面我们主要针对 [ANS][1] 和 [MPNS][3]/[WNS][4] 来看看系统提供商控制的 Push Notification Service 的实现。

## How 

### ANS

所有 Apple Store 上应用的 Remote Notification 都需要在 __ANS__ 上注册获取 Device Token，自建 Push Service 作为 Provider 角色，应用将从 __ANS__ 获取到的 Device Token 发送到自家的 Push Server，Push Server 根据需要发送包含 Device Token 的 Notification 给 __ANS__，最后 __ANS__ 根据 Device Token 将 Notification 发送到正确设备上的正确应用。

[ANS 的工作示意图][1]：

![](/images/remote_notif_simple_2x.png)

> The remote-notification data flows in one direction. The provider composes a notification package that includes the device token for a client app and the payload. The provider sends the notification to APNs which in turn pushes the notification to the device.

当然，__ANS__ 面向所有 iOS 设备的所有来自 Apple Store 的应用提供 Push Service，所以就有下面的更符合实际情况的示意图：

![](/images/remote_notif_multiple_2x.png)

>  The device-facing and provider-facing sides of APNs both have multiple points of connection; on the provider-facing side, these are called gateways. There are typically multiple providers, each making one or more persistent and secure connections with APNs through these gateways. And these providers are sending notifications through APNs to many devices on which their client apps are installed.

为了避免轮询以节省电量并提供更高效的 Push Service，iOS 系统有一个基于 [XMPP][5] 的后台进程来和 __ANS__ 进行通信。很多 IM 应用自己实现即时消息也使用了 XMPP 协议，比如微信。知乎上面有两个关于 __ANS__ 实现机制的问题，下面有很多非常好的答案值得参考：

- [iOS 和 Android 的后台推送原理各是什么？有什么区别？][6]
- [iOS 和 Windows Phone 上推送通知的实现原理是什么？][7]

### MPNS

![](/images/mpns.png)

1. App 从 Windows Phone 系统上的 Push Client 申请用于 Push Notification 的 URI。
2. Push Client 和 MPNS 进行通信后，MPNS 返回一个 URI 给 Push Client。
3. Push Client 将从 MPNS 获取到的 URI 返回给 App。
4. App 将申请到的 URI 发送到 App 提供商自建的 Cloud Server。
5. Cloud Server 在需要发送 Notification 给 App 的时候，会将收到的 URI 和通知体数据发送到 MPNS。
6. MPNS 根据 URI 将 Notification 路由到正确设备的正确 App。

### WNS

![](/images/wns.png)

1. App 从 Windows Phone 系统上的 Notification Client 申请用于 Push Notification 的 channel。
2. Notification Client 请求 WNS 建立一个 channel，WNS 返回一个 URI 格式的 channel 给 Notification Client。
3. Notification Client 将从 WNS 获取到的 URI 返回给 App。
4. App 将申请到的 URI 发送到 App 提供商自建的 Cloud Server，这中间完全有 App 提供商负责通信安全和稳定性。
5. Cloud Server 在需要发送 Notification 给 App 的时候，会将收到的 URI 和通知体数据发送到 MPNS，通信采用经过 SSL 认证的 HTTP Post 方式。
6. MPNS 根据 URI 将 Notification 路由到正确设备的正确 App。

注意：

- WNS 是微软在 MPNS 之后推出的 Push Notification Service，主要提供对 Windows Universal App 的支持，增加了一些 Windows RT 的接口，关于开发者如何选择 WNS 和 MPNS，微软在 [Choosing MPNS or WNS for a Windows Phone Silverlight 8.1 app][8] 这篇文档中给出了一些建议。在 Windows 8.1 之后，推荐使用 WNS。
- 有时候通知发送可能无法及时收到，可以参考 [Understanding the Temp Disconnected State][9] 查找原因。
- [StackOverflow 上的问题：How to test Windows Phone 8.1 push notifications?][10] 里面提供了 Windows 8.1 之后版本使用模拟器中自带的 Additional Tools 测试 Push Notification 的很好方式。
- [windows 8 push client+server sample及部署][10] 


[1]: https://developer.apple.com/library/mac/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html#//apple_ref/doc/uid/TP40008194-CH100-SW9
[2]: http://developer.android.com/google/gcm/index.html
[3]: https://msdn.microsoft.com/en-us/library/windows/apps/ff402558(v=vs.105).aspx
[4]: http://msdn.microsoft.com/en-us/library/windows/apps/hh913756.aspx
[5]: http://en.wikipedia.org/wiki/XMPP
[6]: http://www.zhihu.com/question/20667886
[7]: http://www.zhihu.com/question/20047884
[8]: http://msdn.microsoft.com/en-us/library/windows/apps/dn642085(v=vs.105).aspx
[9]: https://msdn.microsoft.com/en-us/library/windows/apps/ff941100(v=vs.105).aspx#BKMK_UTDS
[10]: http://stackoverflow.com/questions/25011504/how-to-test-windows-phone-8-1-push-notifications
[11]: http://silverlightchina.net/html/windows8/study/2012/1017/19537.html


