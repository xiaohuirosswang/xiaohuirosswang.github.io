---
layout: post
title: "Ubuntu 14.04 基于最新的 openssl 编译 curl"
description: ""
category: [Linux, C++]
---

## 为什么需要编译基于最新版本 openssl 的 curl？

项目开发中，有时候需要用到支持 HTTPS 协议的网络请求，使用 curl 能很方便地实现这些功能，所以我们需要编译 curl。通常情况下，我们使用静态编译，所以需要从源码编译 curl，如果需要支持 SSL，常用的就是基于 openssl 进行编译，而且我们知道使用 SSL 的服务都非常重要，在一些情况下（比如 openssl 的安全漏洞被修复），我们需要升级 openssl，并重新编译 curl，从而使我们的业务保持安全。

## 编译步骤

#### Ubuntu编译环境配置

{% highlight sh %}
# 更新 apt-get, 如果速度慢，可以考虑将更新源变更为 aliyun。
sudo rm -fR /var/lib/apt/lists/*
sudo apt-get update
sudo apt-get upgrade

# 安装 build-essential vim git subversion
sudo apt-get install -y build-essential vim git subversion
{% endhighlight %}

#### 编译安装

{% highlight sh %}
# 下载 openssl 和 curl 的最新版本源码，这里将 tarball 放在 ~/Download 目录下，解压它们。
# 安装 ia23-libs
sudo apt-get install -y lib32z1 lib32ncurses5 lib32bz2-1.0
# 参考 https://launchpad.net/ubuntu/trusty/+source/curl，安装curl的依赖开发库：
sudo apt-get install -y autoconf automake ca-certificates debhelper groff-base libgcrypt11-dev libdb-dev libgnutls-dev libidn11-dev libkrb5-dev libldap2-dev libnss3-dev librtmp-dev libssl-dev libtool openssh-server python quilt zlib1g-dev libssh2-1-dev
# 参考 https://launchpad.net/ubuntu/trusty/+source/openssl，安装openssl的依赖开发库：
sudo apt-get install -y bc debhelper dpkg-dev m4

# 编译安装 openssl
# 必须有-fPIC, 否则之后配置 curl 的时候不会成功，就算在 configure 的时候加上 “LIBS=-ldl” 使得配置不出错，编译的时候也会有下面的错误：
# /usr/bin/ld: /usr/local/lib/libssl.a(s2_clnt.o): relocation R_X86_64_32 against `.rodata' can not be used when making a shared object; recompile with -fPIC
cd openssl-1.0.1j
./configure --prefix=/usr/local shared -fPIC
make && sudo make install

# 编译安装 curl
cd ..
cd curl-7.39.0
./configure --prefix=/usr/local --with-zlib --with-ssl
make -s && sudo make install
{% endhighlight %}

最后如果不需要使用最新版本 openssl 编译 curl 的时候，编译安装按照下面的步骤就可以了：

{% highlight sh %}
# 安装 ia23-libs
sudo apt-get install -y lib32z1 lib32ncurses5 lib32bz2-1.0
# 参考 https://launchpad.net/ubuntu/trusty/+source/curl，安装curl的依赖开发库：
sudo apt-get install -y autoconf automake ca-certificates debhelper groff-base libgcrypt11-dev libdb-dev libgnutls-dev libidn11-dev libkrb5-dev libldap2-dev libnss3-dev librtmp-dev libssl-dev libtool openssh-server python quilt zlib1g-dev libssh2-1-dev

# 编译安装 curl
cd ..
cd curl-7.39.0
./configure --prefix=/usr/local --with-zlib --with-ssl
make -s && sudo make install
{% endhighlight %}

[1]: http://www.unix.com/man-page/opensolaris/3lib/libdl/


