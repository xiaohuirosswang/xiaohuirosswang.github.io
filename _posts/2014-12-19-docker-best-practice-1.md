---
layout: post
title: "Dockerfile 最佳实践"
description: ""
category: [Docker]
---

## Why

Docker 作为非常优秀的轻量级 PaaS 解决方案，得到了主流云服务平台的先后支持，配合 Docker Registry Hub 提供高质量的 Docker Image 以及 Fig 进行 Containers 的管理，吸引了全球的开发者将自己的服务迁移到 Docker Containers 上面，从而提高开发和部署效率。作为快速发展的新技术，很多高质量的技术文章和使用经验都是英文的，我自己在学习 Docker 的过程中，在不同的网站和个人 Blog 就看到过很多，所以，就想着把我看到的高质量文章翻译过来放在这个 Blog，一方面翻译的过程是加深理解的过程，另一方面把散落在各处的好文章集中在一个地方并翻译成中文，便于之后的分享以及回顾。

## What

本文译文部分翻译自 [Dockerfile Best Practices][1]，英文版权归 [原作者][2] 所有，译文转载请注明出处。

## 译文

__Dockerfile__ 对创建 Docker Image 提供了简单的语法，本文记录一些可以让我们真正用好 __Dockerfile__ 的一些技巧和经验。

### 在 Dockerfile 的最开始部分保持通用创建指示内容以利用缓存

Dockerfile 中的每一条指示执行后的结果都会提交到新创建的 Image 中，并作为下一条指示执行的基础。如果已经存在一个拥有相同父 Image 而且该 Image 的 Dockerfile 有相同的指示（除了 __ADD__）的另一个 Image，Docker 将会使用这个已存在的 Image 而不是重新通过执行每一条 Dockrfile 中的指示来创建另一个新的 Image，这就是缓存机制。

为了能够能有效地利用缓存，我们需要尽量保持 Dockerfiles 的一致性，并且将不同的指示放在 Dockerfile 最后的部分，所有我自己的 Dockerfiles 都以下面 5 行开始：

{% highlight text %}
FROM ubuntu
MAINTAINER Michael Crosby <michael@crosbymichael.com>

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y
{% endhighlight %}

> 注意，改变 MAINTAINER 指示将会迫使 Docker 放弃使用缓存并重新通过执行 RUN 指示的操作。

### 在创建 Docker Image 的时候使用 __-t__ 参数对 Image 打上 Tag

如果我们不是仅仅对 Docker 尝尝鲜试用一下，那我们应该在创建 Docker Image 的时候使用 __-t__ 参数对 Image 打上 Tag，因为一个有意义的 Tag 名可以帮助我们明确创建这个 Image 的目的。

docker build -t="crosbymichael/sentry" .

### 不要在 Dockerfile 中进行端口映射

Docker 的两个最核心的特性就是可重复性和可移植性。Image 应该可以被根据需要用来在任何 host 上创建多个 Containers，考虑到这点，虽然我们在 Dockerfile 中可以映射 Container 端口到 host 端口，但是我们永远不应该在 Dockerfile 中这么做，否则我们就只能使用该 Dockerfile 创建的 Image 创建一个 Container。

{% highlight text %}
# private and public mapping
EXPOSE 80:8080

# private only
EXPOSE 80
{% endhighlight %}

> 如果 Image 的使用者关心 Container 应该将自己的什么端口和 host 的端口进行映射，那就会显式地在创建 Container 的时候使用 __-p__ 参数进行设置，否则，Docker 会自动给 Container 分配一个 host 上的端口进行映射。

### 在使用 __CMD__ 和 __ENTRYPOINT__ 指示的时候使用数据传入参数

__CMD__ 和 __ENTRYPOINT__ 指示的作用是容易理解的，但是它们有个隐藏的坑，如果不注意就可能会导致错误发生。这两个指示都支持下面的语法：

{% highlight text %}
CMD /bin/echo
# or
CMD ["/bin/echo"]
{% endhighlight %}

看上去好像没什么，但是隐藏在细节中的坑却可能真的会坑人。如果我们使用下面以数组方式传递参数的方式，最后的结果会和我们预期的相同；但是当我们使用第一种语法时，由于 Docker 会自动在需要执行的命令前加上 __/bin/sh -c__，这种情况下可能会导致一些未预料的错误和不好理解的事情发生，所以总是使用数组传递参数是个好主意，这能保证命令按照我们预期的方式被执行。

### __CMD__ 和 __ENTRYPOINT__ 最好配合使用

__ENTRYPOINT__ 会让我们的 Container 使用起来像个可执行文件，我们可以在使用 __docker run__ 创建 Container 的时候传递参数给 __ENTRYPOINT__ 执行并且不用担心操作被覆盖（就像使用 __CMD__ 后的情况）。在和 __CMD__ 一起配合使用时 __ENTRYPOINT__ 甚至能提供更好的体验，我们通过一个 [Rethinkdb][3] 的例子来看看怎么用：

{% highlight text %}
# Dockerfile for Rethinkdb 
# http://www.rethinkdb.com/

FROM ubuntu

MAINTAINER Michael Crosby <michael@crosbymichael.com>

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y python-software-properties
RUN add-apt-repository ppa:rethinkdb/ppa
RUN apt-get update
RUN apt-get install -y rethinkdb

# Rethinkdb process
EXPOSE 28015
# Rethinkdb admin console
EXPOSE 8080

# Create the /rethinkdb_data dir structure
RUN /usr/bin/rethinkdb create

ENTRYPOINT ["/usr/bin/rethinkdb"]

CMD ["--help"]
{% endhighlight %}

这就是我们将 Rethinkdb 迁移到 Docker 需要做的全部事情。最前面的 5 行是我自己的 Dockerfile 的标准开头，接下来是一些安装和必要的端口暴露等，最后使用 __ENTRYPOINT__。我们知道当我们使用该 Dockerfile 创建的 Image 来创建一个 Container 并运行的时候，所有通过 __docker run__ 命令传递的参数都会被传递给 __ENTRYPOINT(/usr/bin/rethinkdb)__。但是可以看到 __ENTRYPOINT__ 指示下面还有一条 __CMD__ 指示并拥有 __--help__ 的内容，这样就会在当我们在使用 __docker run__ 创建 Container 的时候没有传递参数的时候显示 rethinkdb 的帮助内容，就像这样：

	docker run crosbymichael/rethinkdb

输出为：

{% highlight text %}
Running 'rethinkdb' will create a new data directory or use an existing one,
  and serve as a RethinkDB cluster node.
File path options:
  -d [ --directory ] path           specify directory to store data and metadata
  --io-threads n                    how many simultaneous I/O operations can happen
                                    at the same time

Machine name options:
  -n [ --machine-name ] arg         the name for this machine (as will appear in
                                    the metadata).  If not specified, it will be
                                    randomly chosen from a short list of names.

Network options:
  --bind {all | addr}               add the address of a local interface to listen
                                    on when accepting connections; loopback
                                    addresses are enabled by default
  --cluster-port port               port for receiving connections from other nodes
  --driver-port port                port for rethinkdb protocol client drivers
  -o [ --port-offset ] offset       all ports used locally will have this value
                                    added
  -j [ --join ] host:port           host and port of a rethinkdb node to connect to
  .................
{% endhighlight %}

现在我们再看看如果传递了 “--bind all” 的参数后发生什么：

	docker run crosbymichael/rethinkdb --bind all

输出为：

{% highlight text %}
info: Running rethinkdb 1.7.1-0ubuntu1~precise (GCC 4.6.3)...
info: Running on Linux 3.2.0-45-virtual x86_64
info: Loading data from directory /rethinkdb_data
warn: Could not turn off filesystem caching for database file: "/rethinkdb_data/metadata" (Is the file located on a filesystem that doesn't support direct I/O (e.g. some encrypted or journaled file systems)?) This can cause performance problems.
warn: Could not turn off filesystem caching for database file: "/rethinkdb_data/auth_metadata" (Is the file located on a filesystem that doesn't support direct I/O (e.g. some encrypted or journaled file systems)?) This can cause performance problems.
info: Listening for intracluster connections on port 29015
info: Listening for client driver connections on port 28015
info: Listening for administrative HTTP connections on port 8080
info: Listening on addresses: 127.0.0.1, 172.16.42.13
info: Server ready
info: Someone asked for the nonwhitelisted file /js/handlebars.runtime-1.0.0.beta.6.js, if this should be accessible add it to the whitelist.
{% endhighlight %}

就是这样，一个完整的 Rethinkdb 实例，可以像在 host 上使用 Rethinkdb 的可执行文件一样来进行交互，很简单。

我希望这篇文章能帮助大家善用 Dockerfile 创建自己的 Image 并分享出来。我相信 Dockerfile 是 Docker 之所以非常简单易用的一个重要原因。

[Dockerfile 最佳实践 续][4]


[1]: http://crosbymichael.com/dockerfile-best-practices.html
[2]: http://crosbymichael.com/index.html
[3]: http://www.rethinkdb.com/
[4]: http://crosbymichael.com/dockerfile-best-practices-take-2.html


