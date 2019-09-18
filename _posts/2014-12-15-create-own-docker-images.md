---
layout: post
title: "Ubuntu 14.04 上使用 Docker 创建新的 Images"
description: ""
category: [Docker]
---

## 环境准备

[Ubuntu 14.04.1 LTS][1] 发布后，我欢快地去官网下载并创建了一个虚拟机，但是在在执行下面命令的时候：

	sudo apt-get update

一直会遇到下面的错误

	W: 无法下载 http://extras.ubuntu.com/ubuntu/dists/trusty/main/binary-amd64/Packages  Hash 校验和不符
	W: 无法下载 http://extras.ubuntu.com/ubuntu/dists/trusty/main/binary-i386/Packages  Hash 校验和不符

Google 找解决方案，尝试了个遍，问题依旧。最让人不解的是 Ubuntu 官方源里面这个地址是[存在的][2]，里面的内容也是存在的，在浏览器中也可以访问但就是无法 Update。最后进行了如下尝试：

- 将下面的内容添加入 /etc/resolv.conf，使用 Google 的 DNS 解析：

{% highlight text%}
nameserver 8.8.8.8
nameserver 8.8.4.4	
{% endhighlight %}

- 依次执行下面的命令：

{% highlight sh%}
sudo rm -rf /var/lib/apt/lists/*
sudo mkdir /var/lib/apt/lists/partial
sudo apt-get clean
sudo apt-get update
{% endhighlight %}


遗憾的是问题还是存在，而且由于使用的是官方源，速度很慢。换国内源吧，幸好 Ubuntu 的更新管理器可以检测当前网络环境中最快的第三方源，进行检测后，换成对我的网络来说最快的 __yun-idc.com__ 的镜像。再次清理缓存并重新更新，然后。。。

-------------

> 重复地做相同的事情，但是却期待不一样的结果 ————爱因斯坦

-------------

结果还是一样的，就像更新速度快了些。虽然已经不影响正常安装软件和日常工作，但是想到上面那个 Warning 还是很郁闷。更关键的是，在使用 Dockerfile 来 Build 新的 Image 的时候，由于 Base Image 都是官方的干净系统，所以都需要 apt-get update 的步骤，上面提到的错误会导致这条命令返回 100 而不是 0，Docker 的 Build 过程也会因此中断，实在郁闷。

由于还是没有有效的解决方案，就想想 Workaround 吧。既然 extras.ubuntu 这个地址的包不一定用得上，那就暂时不更新这个地址了，之后如果需要可以通过添加 ppa 来解决。于是从 /etc/apt/sources.list 里面去掉 extras.ubuntu 地址相关的内容，再次更新终于没有错误了。

## 创建 Nginx 和 Python 镜像

基于上面提到的解决方案，我创建了两个 Dockerfiles，分别用来创建自己的 Nginx 和 Python 的开发环境镜像：

- [xhrwang/ubuntu-nginx][3]

{% highlight text%}
# Pull base image.
FROM dockerfile/ubuntu

# Patch the proper apt-get sources
ADD assert/sources.list /etc/apt/sources.list

# Install Nginx.
RUN \
  add-apt-repository -y ppa:nginx/stable && \
  apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 16126D3A3E5C1192 && \
  apt-get update && \
  apt-get install -y nginx && \
  rm -rf /var/lib/apt/lists/* && \
  chown -R www-data:www-data /var/lib/nginx

# Define mountable directories.
VOLUME ["/etc/nginx/sites-enabled", "/etc/nginx/conf.d", "/var/log/nginx"]

# Expose ports.
EXPOSE 80

{% endhighlight %}

- [xhrwang/ubuntu-python][4]

{% highlight text%}
# Pull base image.
FROM dockerfile/ubuntu

# Patch the proper apt-get sources
ADD assert/sources.list /etc/apt/sources.list

# Install Python Dev Env.
RUN \
  apt-get update && \
  apt-get install -y python python-dev python-virtualenv

# Expose ports.
EXPOSE 8080

{% endhighlight %}


这两个 repositories 和我在 Docker 的官方镜像库进行了关联：

- [ubuntu-nginx][5]
- [ubuntu-python][6]

都已经在 Docker registry Hub 中 build 成功。之所以选择 dockerfile/ubuntu 是由于其是官方提供的，而且预先安装了 Git，curl 等常用工具。

__注意__: 本文主要的目的是如何创建自己的 Docker Image，使用 ubuntu-nginx Image 创建的 Container 是无法运行在 daemon 模式下的。生产环境中的 Nginx 需要在 Docker 容器中使用 __-d__ 选项运行，这样 Nginx 本身就不能运行在 daemon 状态，可以参考 [Github 上的这个方案][7] 进行部署。

[1]: http://www.ubuntu.com/download/desktop
[2]: http://extras.ubuntu.com/ubuntu/dists/trusty/main/
[3]: https://github.com/xhrwang/ubuntu-nginx
[4]: https://github.com/xhrwang/ubuntu-python
[5]: https://registry.hub.docker.com/u/xhrwang/ubuntu-nginx/
[6]: https://registry.hub.docker.com/u/xhrwang/ubuntu-python/
[7]: https://github.com/dockerfile/nginx


