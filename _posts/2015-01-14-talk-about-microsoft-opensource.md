---
layout: post
title: "谈谈微软的 2014 开源之路"
description: ""
category: [.Net, C#]
---

2014 年 11 月 12 日，微软官方宣布 [.Net Core 开源][1]，并将代码[托管在 Github][2]，在提到从自己的 CodePlex 迁移到 Github 的原因时，原博文这样说：

> As a principle, we don’t want to ask the community to _come to where we are_. Instead, we want to _go where the community already is_.

可以看出，微软对开源社区释放了很大的善意，并且同时，推出了免费的针对个人的 [Visual Studio 2013 Community][3] 版本，相比于之前免费的 Express 版本，提供了全功能，只是在授权协议上做了针对个人用户的限制：

> Visual Studio Community 2013 is a new edition that enables you to unleash the full power of Visual Studio to develop cross-platform solutions. Create apps in one unified IDE. Get Visual Studio extensions that incorporate new languages, features, and development tools into this IDE. 

如果联系 2014 年 4 月初微软提出的 [.Net Native][4]，可以将 C# 开发的代码编译为原生机器码，提供接近于 C++ 程序的运行性能。这个功能仅限于针对 Windows Store 的应用，但考虑到 Windows 8 的普及以及即将发布的 Windows 10，应用程序发布的官方渠道就是 Windows Store，而且结合基于 MVVM 的 WPF 框架，随着硬盘空间不断增大以及带宽增大带来的网速提高，用户对应用程序大小敏感度下降，那我们可以想象这样的场景：

- Windows 平台应用程序使用 WPF 开发，提供优秀的界面交互。
- 使用 C# 进行开发，提高开发效率。
- 使用 .Net Native 进行编译并发布，保证运行效率。
- .Net Core 开源后，可以预期可以出现很多第三方的优秀库，解决特定领域的问题，开发者的选择更多，社区更活跃。

而且，从微软在宣布 .Net Core 开源的同时发布的 [Visual Studio 2015 Preview][5] 来看，.Net 已经大步迈向跨平台，VS2015 已经集成了 LLVM 和 Clang 编译器支持开发跨平台的应用。微软推荐使用 [Xamarin][6]，可以使用 C# 在 Visual Studio 中开发 iOS，Android 以及 Windows 平台的应用，那么需要支持多个移动平台的 App 开发者来说，有下面的优势：

- App 核心逻辑可以使用相同的代码模块，针对不同的平台设计不同的界面，提高代码重用性、可维护性以及可扩展性，缩短开发周期。

至于 .Net 框架上的 ASP.NET vNext 以及 MVC，还需要看微软将 .Net Framework 移植到 Linux 系统之后的性能。作为开发者，我们肯定愿意面对问题的时候有多个优秀可选项，所以对于微软 2014 年的一系列调整，我们应该欢迎和赞赏，并期待接下来更多的惊喜。


[1]: http://blogs.msdn.com/b/dotnet/archive/2014/11/12/net-core-is-open-source.aspx
[2]: http://microsoft.github.io/
[3]: http://www.visualstudio.com/en-us/news/vs2013-community-vs.aspx
[4]: http://msdn.microsoft.com/en-us/vstudio/dotnetnative.aspx
[5]: http://www.visualstudio.com/en-us/news/vs2015-preview-vs.aspx
[6]: http://xamarin.com/


