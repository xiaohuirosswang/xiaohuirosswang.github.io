---
layout: post
title: "HyperLapse的强大拍摄能力以及 video-js"
description: ""
category: [App]
---

### 这篇文章的目的有两个

- 展示 HyperLapse 的强大拍摄能力。
- [video-js][1] 的 HTML5 视频播放框架使用。

> An HTML5 Video Player is a JavaScript library that builds a custom set of controls over top of the HTML5 video element to provide a consistent look between HTML5 browsers. Video.js builds on this by fixing many cross browser bugs or inconsistencies, adding new features that haven't been implemented by all browsers (like fullscreen and subtitles), as well as providing one consistent JavaScript API for both HTML5, Flash, and other playback technologies.

video-js 的使用非常简单，支持常见的视频格式，可以自定义播放器样式，并且是个开源项目，是 HTML5 视频播放方案的很好选择。使用非常简单：

{% highlight html %}
<link href="/images/video-js/video-js.css" rel="stylesheet" type="text/css">
<script src="/images/video-js/video.js"></script>
  <video id="example_video_1" class="video-js vjs-default-skin" controls preload="none" width="540" height="960"
      poster="/images/awesome.jpg"
      data-setup="{}">
    <source src="/images/video-js/cars.webm" type='video/webm'/>
    <source src="/images/video-js/cars.mp4" type='video/mp4'/>
    <p class="vjs-no-js">To view this video please enable JavaScript, and consider upgrading to a web browser that <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a></p>
  </video>
{% endhighlight %}

<link href="/images/video-js/video-js.css" rel="stylesheet" type="text/css">
<script src="/images/video-js/video.js"></script>

#### 这个是在北京朝阳门拍的，可以看到防抖的效果非常好。

  <video id="example_video_1" class="video-js vjs-default-skin" controls preload="none" width="540" height="960"
      poster="/images/awesome.jpg"
      data-setup="{}">
    <source src="http://rwang-video.oss-cn-beijing.aliyuncs.com/cars.webm" type='video/webm'/>
    <source src="http://rwang-video.oss-cn-beijing.aliyuncs.com/cars.mp4" type='video/mp4'/>
        <object id="flash_fallback_1" class="vjs-flash-fallback" width="540" height="960" type="application/x-shockwave-flash" data="http://rwang-video.oss-cn-beijing.aliyuncs.com/flowplayer-3.2.1.swf">
        <param name="movie" value="http://rwang-video.oss-cn-beijing.aliyuncs.com/flowplayer-3.2.1.swf" />
        <param name="allowfullscreen" value="true" />
        <param name="flashvars" value='config={"playlist":["/images/video-js/awesome.jpg", {"url": "http://rwang-video.oss-cn-beijing.aliyuncs.com/cars.flv","autoPlay":false,"autoBuffering":true}]}' />
        <img src="/images/video-js/awesome.jpg" width="280" height="210" alt="Poster Image"  title="No video playback capabilities." />
      </object>
    <p class="vjs-no-js">To view this video please enable JavaScript, and consider upgrading to a web browser that <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a></p>
  </video>

#### 这个是在万达广场里面拍的，背景虚化也很好。

  <video id="example_video_1" class="video-js vjs-default-skin" controls preload="none" width="540" height="960"
      poster="/images/awesome.jpg"
      data-setup="{}">
    <source src="http://rwang-video.oss-cn-beijing.aliyuncs.com/diandian.webm" type='video/webm'/>
    <source src="http://rwang-video.oss-cn-beijing.aliyuncs.com/diandian.mp4" type='video/mp4'/>
        <object id="flash_fallback_1" class="vjs-flash-fallback" width="540" height="960" type="application/x-shockwave-flash" data="http://rwang-video.oss-cn-beijing.aliyuncs.com/flowplayer-3.2.1.swf">
        <param name="movie" value="http://rwang-video.oss-cn-beijing.aliyuncs.com/flowplayer-3.2.1.swf" />
        <param name="allowfullscreen" value="true" />
        <param name="flashvars" value='config={"playlist":["/images/video-js/awesome.jpg", {"url": "http://rwang-video.oss-cn-beijing.aliyuncs.com/diandian.flv","autoPlay":false,"autoBuffering":true}]}' />
        <img src="/images/video-js/awesome.jpg" width="280" height="210" alt="Poster Image"  title="No video playback capabilities." />
      </object>
    <p class="vjs-no-js">To view this video please enable JavaScript, and consider upgrading to a web browser that <a href="http://videojs.com/html5-video-support/" target="_blank">supports HTML5 video</a></p>
  </video>

[1]: http://www.videojs.com/


