---
layout: post
title: "使用 Fig 管理 Flask 和 Nginx 的开发环境"
description: ""
category: [Docker, Flask]
---

### Why

[Fig][1] 是 Docker 官方推出的非常易用却相当 Powerful 的 Containers 部署管理工具，其和 Docker 一样是个开源项目，开发背景可以参考 [WELCOMING THE ORCHARD AND FIG TEAM][2] 这篇文章。

当我们的服务由多个 Container 共同协作提供时，使用 __Fig__ 可以提高部署效率，同时，在开发环境中，__Fig__也能够帮助我们省去环境配置的时间，提高开发测试效率。

Docker 官方博客有一篇文章介绍了如何使用 __Fig__ 创建自己的业务流程（Orchestration，如果有 Microsoft BizTalk 的使用经验，看到这个词会非常亲切）：
[GETTING STARTED WITH DOCKER ORCHESTRATION USING FIG][3]。

## What

本文记录在 Ubuntu 14.04 上如何使用 __Fig__ 来管理基于 Flask 并使用 Nginx 代理的 Web Service，这个 Service 提供北京市公交实时到站信息查询服务。由于使用了 `Gunicorn`，所以无法将 Nginx 作为独立的 Container 来运行，所以整个服务只用到一个 Container，但是不影响我们使用 __Fig__，如果需要联系使用 __Fig__ 以及 links 管理多个 Container，可以将服务中的 sqlite 数据库换位 Mongodb 等第三方数据库单独运行在另一个 Container 中，然后像官方 Demo 的做法一样 fig up 就可以了。 

北京公交实时信息来自于 [Github 上的这个项目][4]，我添加了 Web Service 模块，并扩展了其本身提供的微信公众账号接口的查询功能。

如果对部署 Flask 应用和 Nginx 不熟悉，可以看看我的另一篇文章：[Ubuntu 14.04 系统基于 Gunicorn 和 Nginx 部署 Flask 应用][9]

## How

### 环境准备

首先，从 [Trusted Automated Docker Builds][5] 获取下面的 Docker Image：

- [dockerfile/python][7]，注意，这里的 Python 版本是 2.7


其次，在 ～/docker/ 下按照下面的目录结构创建项目:

{% highlight text %}
└── beijing-bus
    └── python
{% endhighlight %}

然后，进入 ～/docker/beijing-bus/python 目录，使用 Git 同步 Python 代码：

	git clone https://github.com/xhrwang/Beijing-Bus-Weixin

最后，还在 ～/docker/beijing-bus/python 目录创建 beijing-bus 文件，并将下面的 Nginx 配置保存在里面：

{% highlight text %}
server {
    listen 80;
    server_name 127.0.0.1;
 
    root ～/flask-hello/;
 
    access_log ～/flask-hello/logs/access.log;
    error_log ～/flask-hello/logs/error.log;
 
    location / {
	    try_files $uri @gunicorn_proxy;
    }

    location @gunicorn_proxy {
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header Host $http_host;
	    proxy_redirect off;
	    proxy_pass http://127.0.0.1:8000;
	}
}
{% endhighlight %}

### 部署 Fig

按照 [官方文档安装 Docker 和 Fig][8]，如果已经安装了 __1.3__ 版本或者更新的 Docker，使用下面的命令安装 Fig：

	sudo curl -L https://github.com/docker/fig/releases/download/1.0.1/fig-`uname -s`-`uname -m` > /usr/local/bin/fig; chmod +x /usr/local/bin/fig

如果提示 SSL 连接错误，可以从 [官方下载地址][10] 手动下载并按照上面的命令部署好 Fig。按照下面的方式验证一下是否安装部署成功：

{% highlight text %}
rwang@Ubuntu1404:~/docker/beijing-bus/python$ fig version
No such command: version

Commands:
  build     Build or rebuild services
  help      Get help on a command
  kill      Kill containers
  logs      View output from containers
  port      Print the public port for a port binding
  ps        List containers
  pull      Pulls service images
  rm        Remove stopped containers
  run       Run a one-off command
  scale     Set number of containers for a service
  start     Start services
  stop      Stop services
  restart   Restart services
  up        Create and start containers
{% endhighlight %}

如果看到类似上面的结果就说明 Fig 已经安装好了。

### 创建公交信息本地缓存

在 ～/beijing-bus/python 目录下，执行下面的命令建立本地公交信息缓存：

* `sudo apt-get install -y python python-dev python-pip python virtualenv` 安装 pip 和 virtualenv。
* `virtualenv flask` 创建名为 flask 的 virtualenv
* `source flask/bin/activate` 进入这个 virtualenv 环境
* `pip install -r requirements.txt` 安装依赖
* `python manage.py build_cache` 获取离线数据，建立本地缓存
* `deactivate` 退出 virtualenv 环境

由于我们将在 Docker 的 Container 里面运行 Flask 应用，本身就是独立的容器，所以不需要 virtualenv 了，python 目录下的 flask 目录可以删除。这里如果使用 virtualenvwrapper 就没有这个麻烦了，因为 virtualenv 会保存在其他地方。更多信息请参考我的另一篇文章： [Windows 平台上的 Python 开发][11] 中关于 virtualenvwrapper 的部分。

### 创建应用镜像

在 ～/beijing-bus/python 目录下，使用下面的 Dockerfile 创建 Flask + Nginx 的镜像：

{% highlight text %}
# Use base image
FROM dockerfile/python

# Add all items in current dir into /data dir of  Image
ADD . /data

# Install Nginx and configure flask application runtime dependencies
RUN \
          cp /data/sources.list /etc/apt/sources.list && \
          apt-get update && \
          apt-get install -y nginx-full && \
          cp /data/beijing-bus /etc/nginx/sites-enabled/default && \
          pip install -r /data/requirements.txt

WORKDIR /data

EXPOSE 80
EXPOSE 8000
{% endhighlight %}

### 创建 fig.yml 文件

在 ～/docker/beijing-bus 目录下，创建 fig.yml 文件：

{% highlight text %}
python:  
  image: python:bj-bus
  command: ./run.sh
  ports:
    - "80:80"
{% endhighlight %}

这里使用 `run.sh` 的原因是 fig.yml 不支持多条 `command` 命令执行，参考 [Multiple command with fig?][15]，`run.sh` 启动了 Nginx 服务以及 gunicorn 进程：

{% highlight sh %}
#! /bin/bash

/etc/init.d/nginx restart
gunicorn myflaskapp:app 
{% endhighlight %}

### 使用 fig 启动服务

好了，终于到最后了，在命令行中输入下面的命令：

{% highlight text %}
rwang@Ubuntu1404:~/docker/beijing-bus$ fig up -d
Creating beijingbus_python_1...
rwang@Ubuntu1404:~/docker/beijing-bus$ fig ps
       Name           Command    State              Ports             
---------------------------------------------------------------------
beijingbus_python_1   ./run.sh   Up      0.0.0.0:80->80/tcp, 8000/tcp 
{% endhighlight %}

如果看到上面的结果就说明服务正常启动了，打开浏览器输入地址 `127.0.0.1` 进行测试，将看到下面的提示：

{% highlight text %}
欢迎查询北京公交线路信息以及实时到站时间。

查询方式：

起点终点查询

查询示例：从西坝河到将台路口西

线路查询

查询示例：102

回复 help 获取帮助信息

回复 all 获取所有支持的公交线路

本接口是微信公众号 giant123 的数据接口，如果需要网络查询，请查询 “wangdiandian.com/web/[123]”
{% endhighlight %}

## Final Word

__Fig__ 的官方 Demo 里面有个看似多余的步骤，首先在创建 python 镜像的时候已经将 /code 拷贝进去了，后面在 fig.yml 里面又定义了 volume 块将 /code 目录挂载到 Container 中，好像没有必要，经过测试，可以从 fig.yml 里面去掉 volume 块也不影响运行。可能的解释就是：在 Dockerfile 中 ADD 进去是为了执行 `pip install -r requirements.txt` 命令，这时没有必要把整个项目代码也放进去，只拷贝 requirements.txt 就可以了；后面我们可能会在 host 上修改代码，然后启动 Container 运行的时候挂载进去使用最新代码。

参考资料：

- [Beautiful Laravel Development with Docker & Fig][11]
- [Docker run reference][12]
- [Linking Containers Together][13]
- [fig.yml reference][14]


[1]: http://www.fig.sh/index.html
[2]: http://blog.docker.com/2014/07/welcoming-the-orchard-and-fig-team/
[3]: http://blog.docker.com/2014/08/getting-started-with-orchestration-using-fig/
[4]: https://github.com/wong2/beijing_bus
[5]: https://github.com/dockerfile
[7]: https://github.com/dockerfile/python
[8]: http://www.fig.sh/install.html
[9]: http://xhrwang.me/2014/12/17/deploy-flask-app-with-gunicorn-and-nginx.html
[10]: https://github.com/docker/fig/releases/
[11]: http://xhrwang.me/2014/12/11/python-on-windows.html
[12]: https://docs.docker.com/reference/run/
[13]: https://docs.docker.com/userguide/dockerlinks/
[14]: http://www.fig.sh/yml.html
[15]: https://groups.google.com/forum/#!topic/docker-user/2gA_Ebt4Q0I


