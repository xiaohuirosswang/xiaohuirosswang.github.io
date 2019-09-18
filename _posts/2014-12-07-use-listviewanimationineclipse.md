---
layout: post
title: "Use ListViewAnimation in eclipse"
description: ""
category: [Android]
---

## This article is about my repository [ListViewAnimationDemoForEclipse][6]

Based on ListViewAnimation repository, target Lollipop and for Eclipse ADT, all libraries in source

Credits go to below repositories:

- [nhaarman/ListViewAnimations][1]
- [JakeWharton/NineOldAndroids][2]

## Why

Official ListViewAnimation repository provides gradle based projects for Android Studio, but after being upgraded to Version 1 RC 4, Android Studio cannot properly build the projects in the repository. This one is for Eclipse ADT for old time sake. Instead of using jar files according to [official doc][5], I imported the sources of the libs and the example app.

These projects are tested in Eclipse Luna with ADT, target build API 21.

Luckly, I succeeded when trying to import my eclipse version to Android Studio 1 RC4, building and running on an Android phone is smooth, so I also create [a repository for Android Studio 1 RC 4][7] .

## How to use

Just clone this repository and use your Eclipse to import them all.

### Depedences:

- lib-core
	- appcompat_v7
- lib-core-slh
	- appcompat_v7
	- lib-core
	- lib-stickylistheaders
- lib-manipulation
	- appcompat_v7
	- lib-core
- lib-stickylistheaders
	- appcompat_v7
- ListViewAnimationDemo
	- appcompat_v7
	- lib-core
	- lib-core-slh
	- lib-manipulation
	- lib-stickylistheaders

#### Notice: Google billing module required by the demo app needs to be imported manually, which has been taken care of in this repository, so just enjoy it.

### The Demo App

The Demo App is actually from [the example of ListViewAnimations][3], which exellently demostrated how great ListViewAnimations is.

![][4]

[1]: https://github.com/nhaarman/ListViewAnimations
[2]: https://github.com/JakeWharton/NineOldAndroids/downloads
[3]: https://github.com/nhaarman/ListViewAnimations/tree/master/example
[4]: https://raw.githubusercontent.com/nhaarman/ListViewAnimations/gh-pages/images/dynamiclistview.gif
[5]: http://nhaarman.github.io/ListViewAnimations/#getting-started
[6]: https://github.com/xhrwang/ListViewAnimationDemoForEclipse
[7]: https://github.com/xhrwang/ListViewAnimationDemoForAndroidStudio1RC4


