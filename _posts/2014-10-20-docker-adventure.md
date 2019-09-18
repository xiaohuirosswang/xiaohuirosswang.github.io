---
layout: post
title: "Docker - 小李飞刀般的虚拟化解决方案"
description: ""
category: [Linux,Docker]
---

## Docker 介绍

#### [Docker][1] 是一个可以让开发者、系统管理员创建、分发并最终运行应用程序的开放平台。它使用 Google 的 Go 语言开发，并在 [Github 上开源][2]。作为轻量级的操作系统虚拟化解决方案，Docker 现在受到了越来越多的关注和使用，特别是RedHat，Ubuntu以及Google 都宣布官方支持 Docker，更让 Docker 走上了快车道，吸引了非常多的开发者。

#### Docker 和 传统的虚拟机解决方案相比有哪些优势呢，可以看看官网的一张示意图：
![][dockervm]

#### Docker 是基于 LXC 的，使用操作系统上的容器，所以提高了操作系统的复用程度，甚至可以在一台物理机上部署成百上千个 Docker 容器。当然，由于运行依赖于操作系统，Docker 本身也受当前操作系统的限制。具体可以看看 [知乎的一个讨论][3]。

## Docker 基本概念

### Docker 镜像（Image）

#### Docker 镜像是用来创建 Docker 容器的只读模板。Docker 官方仓库提供了非常多从 [操作系统][4] 到 [各种编程语言环境][5] 的官方镜像，我们可以基于这些镜像创建我们自己的镜像，也可以使用 “docker build” 命令配合 dockerfile 生成新的镜像。Docker 镜像可以发布到公共仓库（比如 Docker 官方维护的 [Docker Hub][6]），也可以存放在自己的私有仓库。通过创建 Docker 镜像可以非常方便地分享和部署应用。

### Docker 容器（Container）

#### Docker 容器是基于 Docker 镜像创建的 Docker 运行实例。同一系统之上的容器相互隔离，互不影响，各自有独立的进程空间和网络空间。在一个容器退出后，我们可以基于该容器创建新的 Docker 镜像。

### Docker 仓库（Repository）

#### Docker 仓库是存放各种 Docker 镜像的库。比如 Ubuntu 系统的仓库提供了不同版本的 Docker 镜像。

## 安装 Docker

#### Docker 支持 Mac OS 和很多主流的 Linux 操作系统，[官方文档介绍得很详细][7]，这里只提一下在 Ubuntu14.04 64bit上的安装：

	curl -sSL https://get.docker.com/ubuntu/ | sudo sh

#### 如果安装完成后使用 docker 时出现下面的错误：

> Cannot connect to the Docker daemon. Is 'docker -d' running on this host?

#### 那么参考[这里的内容][11]，我们需要执行下面的命令来安装 apparmor：

    sudo apt-get install apparmor

## Docker 基本操作

### 查看本地 Docker 镜像

	sudo docker images

### 获取 Docker 镜像

	sudo docker pull ubuntu:14.04

#### 上面的命令从 Docker 官方仓库注册服务器下载 Ubuntu 1404，这条命令等价于：

	sudo docker pull registry.hub.docker.com/ubuntu:14.04

#### 可以更清楚得看到下载地址是 Docker Hub 的 registry 下 ubuntu 仓库中 tag 为 14.04 的镜像。

### 创建镜像

#### 前面提到过我们可以通过修改现有镜像来创建新镜像，也可以通过创建命令和配置文件来从头创建新镜像，下面分别介绍。

- 通过修改已有镜像方式创建新镜像

{% highlight sh %}
# 基于 tag 为 14.04 的 ubuntu 镜像创建一个容器，并在容器中运行 /bin/bash
# -t 表示在容器中开启一个伪终端
# -i 表示连接到容器的标准输入
rwang@ubuntu140464:~$ sudo docker run -i -t ubuntu:14.04 /bin/bash

# 更新软件源
root@cd2107af89ae:/# apt-get update

# 安装 python 开发环境
root@cd2107af89ae:/# apt-get install -y build-essential python-dev libevent-dev python-pip liblzma-dev

# 退出容器
root@cd2107af89ae:/# exit

# 查看当前镜像信息
rwang@ubuntu140464:~$ sudo docker images

# 基于刚才的容器创建新镜像
rwang@ubuntu140464:~$ sudo docker commit -m "Ubuntu1404 Python2.7 Env" -a "xhrwang@gmail.com" cd2107af89ae ubuntu-python27:v1

# 再次查看本地镜像信息，会发现多了一条下面的信息：
# REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
# ubuntu-python27     v1                  a6c20de08034        7 minutes ago       392.7 MB

# 基于新创建的镜像创建容器
rwang@ubuntu140464:~$ sudo docker run -it ubuntu-python27:v1 /bin/bash

# python 开发环境已就绪
root@ef66d647855b:/# python

# Python 2.7.6 (default, Mar 22 2014, 22:59:56) 
# [GCC 4.8.2] on linux2
# Type "help", "copyright", "credits" or "license" for more information.
# >>>

{% endhighlight%}

- 通过创建命令和配置文件创建新镜像

Docker 镜像配置文件内容实例如下：

{% highlight sh %}
# 注释的形式是这样的。
# This is a comment

# Docker 创建镜像的时候基于哪一个镜像
FROM ubuntu:14.04

# 镜像维护者的信息
MAINTAINER Docker Newbee <newbee@docker.com>

# 创建过程中执行的命令，每条 RUN 命令就是镜像的一个 layer 
RUN apt-get -qq update
RUN apt-get -qqy install ruby ruby-dev
RUN gem install sinatra
{% endhighlight%}

使用下面的命令进行 build

	rwang@ubuntu140464:~$ sudo docker build -t="IMAGENAME:IMAGETAG" DockerfilePath

结束后我们就可以通过 “Docker images” 来查看本地镜像列表。

### 导出和载入镜像

#### 使用 “docker save” 可以把本地镜像到处为一个tarball，使用方法如下：

{% highlight sh %}
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              14.04               c4ff7513909d        5 weeks ago         225.4 MB
...
$sudo docker save -o ubuntu_14.04.tar ubuntu:14.04
{% endhighlight%}

#### 使用 “docker load” 可以将之前导出的tarball导入到本地镜像库，使用方法如下：

	$ sudo docker load --input ubuntu_14.04.tar

或者：

	$ sudo docker load < ubuntu_14.04.tar

### 移除镜像

#### 删除一个镜像之前，需要先使用下面的命令删除基于这个镜像的所有容器

	$ sudo docker rm ubuntu:14.04

#### 使用 “docker rmi” 可以将从本地镜像库中删除一个镜像，使用方法如下：

	$ sudo docker rmi ubuntu:14.04

## 容器

### 新建一个容器并启动

#### 在不需要和容器进行交互，只执行某个操作的时候，使用下面的命令基于一个镜像新建并启动一个容器：

	$ sudo docker run ubuntu:14.04 /bin/echo 'Hello world'

#### 如果需要和容器进行交互，可以让 Docker 给容器分配一个伪终端，并将当前 shell 和容器的标准输入进行绑定，我们之前使用过类似的用法：

	$ sudo docker run -t -i ubuntu:14.04 /bin/bash

#### 然后就可以在容器的环境中进行操作了。

#### 使用 “docker run” 时，[发生了下面这些事情][9]：

1. 检查本地是否存在指定的镜像，不存在就从公有仓库下载
2. 利用镜像创建并启动一个容器
3. 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
4. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
5. 从地址池配置一个 ip 地址给容器
6. 执行用户指定的应用程序
7. 执行完毕后容器被终止

#### 启动一个已经终止的容器

	$ sudo docker start CONTAINERID

#### 重新启动一个正在运行的容器，对于后面马上说到的 daemon 方式运行的容器很有用

	$ sudo docker restart CONTAINERID

### 后台运行容器

	$ sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"

#### 使用 “docker ps” 可以看到正在运行的容器信息

#### 如果需要关注后台容器的运行状态，可以使用 “docker logs” 来查看运行日志

### 终止容器运行

#### 如果 shell 连接到容器的标准输入，可以使用 exit 命令或者 Ctrl + D 来终止容器运行。对于后台运行的容器，可以使用 “docker stop” 命令来做同样的事情。

### 连接容器

#### 对于打开了伪终端的后台容器来说，可以通过下面的命令创建并运行：

	$ sudo docker -idt ubuntu:14.04

#### 然后，可以通过下面的命令连接到这个容器：

{% highlight sh %}
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia
$sudo docker attach nostalgic_hypatia
root@243c32535da7:/#
{% endhighlight%}

### 导出和载入容器

{% highlight sh %}
$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:14.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ sudo docker export 7691a814370e > ubuntu.tar
{% endhighlight%}

{% highlight sh %}
$ cat ubuntu.tar | sudo docker import - test/buntu:v1.0
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
test/ubuntu         v1.0                9d37a6082e97        About a minute ago   171.3 MB
{% endhighlight%}

#### 导出的容器只是容器当前状态的快照，不包含该容器之前的历史记录和元数据信息。

## 仓库

#### 我们可以把允许公开的镜像上传到 Docker 官方维护的 Docker Hub 来共享，也可以下载安装 docker-registry 创建私有仓库来保存自己的镜像，具体的文档可以参考[这里][10]


#### [Docker 资源][8]

[1]: https://www.docker.com/whatisdocker/
[2]: https://github.com/docker/docker
[3]: http://www.zhihu.com/question/25394149
[4]: https://registry.hub.docker.com/search?q=library&f=official
[5]: https://blog.docker.com/2014/09/docker-hub-official-repos-announcing-language-stacks/
[6]: https://registry.hub.docker.com/
[7]: http://docs.docker.com/installation/
[8]: http://www.oschina.net/question/2012764_174713
[9]: http://yeasy.gitbooks.io/docker_practice/content/container/run.html
[10]: http://yeasy.gitbooks.io/docker_practice/content/repository/local_repo.html
[11]: https://www.digitalocean.com/community/questions/start-docker-error-message

[dockervm]: /images/docker.png


