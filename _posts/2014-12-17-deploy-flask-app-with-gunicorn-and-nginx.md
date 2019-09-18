---
layout: post
title: "Ubuntu 14.04 系统基于 Gunicorn 和 Nginx 部署 Flask 应用"
description: ""
category: [Python, Flask, Gunicorn]
---

## Why

我在 [建立基于web.py，Nginx and uWsgi的RESTful网站][4] 中记录过使用 uWsgi 和 Nginx 部署基于 Web.py 的 Python Web 应用的流程，这种方式也适用于部署基于 Flask 的应用。我们可以从中感受到配置 uWsgi 还是比较麻烦的，对于访问量不大的 Web 应用来说，有没有更方便的部署方式呢？答案就是 [Gunicorn][5]。

[UWSGI VS. GUNICORN, OR HOW TO MAKE PYTHON GO FASTER THAN NODE][6] 对 uWsgi 和 Gunicorn 两种部署方式下的性能做了比较，可以参考。

关于 __Gunicorn__，[How to Deploy Python WSGI Apps Using Gunicorn HTTP Server Behind Nginx][3] 这篇文章中提到：

-------------

> Gunicorn is a stand-alone web server which offers quite a bit of functionality in a significantly easy to operate fashion. It uses the pre-fork model -- meaning that a central master process (Gunicorn) is tasked with managing the initiated worker processes (of differing types), which then handle and deal with the requests directly. And all this can be configured and adapted to suit your needs and diverse production scenarios.

-------------

也就是说 __Gunicorn__ 是一个独立的（stand-alone）、提供丰富功能的 WSGI Web 应用服务器。它基于自己的适配器原声支持多种框架，能够很容易地用于替代我们开发的时候使用的 Web 服务器。从技术角度来说，__Gunicorn__ 的工作方式与 Ruby 应用常用的 Unicorn 服务器非常相似。它们都使用一种叫做 [pre-fork 的模型][2]，也就是使用一个 Master 进程来管理所有的工作进程、创建 sockets 和 bindings 等。

[DigitalOcean 上面有有一篇非常好文章][1] 对常用的 Python Web 应用部署方案进行了比较，并对各个方案的适用场景做了描述，如果我们在使用何种方案部署 Python Web 应用上面面对多种方案无所适从的时候，这篇文章可以为我们提供决策的参考。关于什么时候我们应该考虑使用 __Gunicorn__ 时，文章是这么说的：

-------------

> - It supports WSGI and can be used with any WSGI running Python application and framework.
> - It can also be used as a drop-in replacement for Paster (ex: Pyramid), Django's Development Server, web2py, et alia.
> - Offers the choice of various worker types/configurations and automatic worker process management.
> - HTTP/1.0 and HTTP/1.1 (Keep-Alive) support through synchronous and asynchronous workers.
> - Comes with SSL support
> - Extensible with hooks.
> - It is transparent and has a clear architecture.
> - Supports Python versions 2.6, 2.7, 3, 3.2, 3.3

-------------

所以，__Gunicorn__ + __Nginx__ 的 Pyhon Web 部署方案是值得了解的。

## What

下文中，我们将记录如何在 Ubuntu 14.04 + Gunicorn + Nginx 上面部署基于 Flask 的 Python Web 应用。

## 部署

这部分参考 [HOW TO RUN FLASK APPLICATIONS WITH NGINX USING GUNICORN][7]，我自己在 Ubuntu 14.04 上进行了验证。

### 环境准备

安装 Python 开发环境：

	sudo apt-get install -y python python-dev python-pip python-virtualenv

建立 Virtualenv 开发目录：

	cd ~
	mkdir flask-hello
	cd flask-hello
	virtualenv hello
	source hello/bin/activate

运行完上面的命令后，命令提示符最前面的括号中应该显示了 virtualenv 的名字 “hello”，如果没有请检查上面的命令运行结果确保执行成功。然后，在这个 virutalenv 中安装 Flask：

	pip install flask

### 创建 Flask 应用

新建 hello.py 文件并将下面的内容保存在该文件中：

{% highlight python %}
from flask import Flask
app = Flask(__name__)
 
@app.route('/')
def hello():
    return "Hello world!"
 
if __name__ == '__main__':
    app.run()
{% endhighlight%}

关于 Flask，这里就不多介绍了，感兴趣的可以参考我的另一篇文章：[基于 Flask 实现 RESTful API][8]

### 引入 Gunicorn

在之前的 virutalenv 环境中，安装 gunicorn：

	pip install gunicorn

为了使我们的 Flask 应用能够使用 gunicorn，需要修改上面的 hello.py 内容如下：

{% highlight python %}
from flask import Flask
from werkzeug.contrib.fixers import ProxyFix
app = Flask(__name__)
 
@app.route('/')
def hello():
    return "Hello world!"
 
app.wsgi_app = ProxyFix(app.wsgi_app)
 
if __name__ == '__main__':
    app.run()
{% endhighlight%}

注意这里和上面内容的不同就是使用 gunicorn 作为 这个应用服务器的关键。

### Nginx

使用下面的命令退出 virutalenv：

	deactivate

安装 nginx：

	sudo apt-get install -y nginx-full

安装后 nginx 的站点配置文件存在于 /etc/nginx/sites-enabled/default，而这个文件时 /etc/nginx/sites-available/default 的软链接，所以我们可以先给 /etc/nginx/sites-available/default 文件做个备份，然后直接修改它就好：

	sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default-bak

然后用下面的内容替换 /etc/nginx/sites-available/default 里面的内容：

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
{% endhighlight%}

重启 nginx 的服务：

	sudo service nginx restart

重新进入刚才应用的 virtualenv 环境并启动 gunicorn：

	cd ~/flask-hello
	source hello/bin/activate
	gunicorn hello:app

默认情况下，gunicorn 会使用 8000 端口。

### 测试

好了，打开浏览器访问 127.0.0.1 或者使用 curl 执行下面的命令：

	curl 127.0.0.1

看到 “Hello world!” 就说明部署成功了。


[1]: https://www.digitalocean.com/community/tutorials/a-comparison-of-web-servers-for-python-based-web-applications
[2]: http://stackoverflow.com/questions/25834333/what-exactly-is-a-pre-fork-web-server-model
[3]: https://www.digitalocean.com/community/tutorials/how-to-deploy-python-wsgi-apps-using-gunicorn-http-server-behind-nginx
[4]: http://xhrwang.me/2014/06/11/create-a-webpy-based-web-server-with-nginx-and-uwsgi.html
[5]: http://gunicorn.org/
[6]: http://blog.kgriffs.com/2012/12/18/uwsgi-vs-gunicorn-vs-node-benchmarks.html
[7]: http://www.onurguzel.com/how-to-run-flask-applications-with-nginx-using-gunicorn/#comment-816040378
[8]: http://xhrwang.me/2014/12/13/restful-api-by-flask.html


