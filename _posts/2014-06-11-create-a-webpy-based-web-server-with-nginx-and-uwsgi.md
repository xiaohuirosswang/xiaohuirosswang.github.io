---
layout: post
title: "建立基于web.py，Nginx and uWsgi的RESTful网站"
description: ""
category: [Python]
---

## web.py

[web.py是一个开源、轻量级的 python web框架][1]，使用其可以非常方便地创建网站，特别是RESTful的API

## uWsgi

常用的部署Python projects的方式有FastCGI和uWsgi：

* FastCGI: 每个Python app都有一个进程，分别和http服务器进行通信
* [uWsgi][2]: 让Python apps部署具有和php-cgi类似的便利性，性能优秀，同时进行多个app的管理，具有详尽的日志记录能力，并提供web server相关的高度可定制性。

![][uwsgi picture1]

## Nginx

体积小，近年市场占有率迅速增加的web server，不多介绍。

## 部署环境

* OS: Ubuntu 14.04LTS
* Python Version: 2.7

#### 安装virtualenv

	sudo apt-get install -y python-virtualenv

#### 安装python开发环境

	sudo apt-get install -y python-dev

#### 安装uWsgi

	sudo apt-get install -y uwsgi

#### 安装uwsgi-python插件	

	sudo apt-get install -y uwsgi-plugin-python

#### 安装Nginx

	sudo apt-get install -y nginx-full

#### 安装nginx的附件

	sudo apt-get install -y nginx-extras

#### 使用下面的命令新建并打开uWsgi的配置文件

	sudo vi /etc/uwsgi/apps-available/vhosts.ini

#### 将下面的内容拷贝入vhosts.ini, 替换YOURUSERNAME为web server的用户

	[uwsgi]
	plugins = python
	gid = YOURUSERNAME
	uid = YOURUSERNAME
	vhost = true
	logdate
	socket = 127.0.0.1:9980
	master = true
	processes = 1
	harakiri = 20
	limit-as = 128
	memory-report
	no-orphans

#### 使用下面的命令新建并打开Nginx的站点配置文件

	sudo vi /etc/nginx/sites-available/siteinpython

#### 将下面的内容拷贝入vhosts.ini

	server {
  	listen 9981;
  	server_name 127.0.0.1;
  	location / {
    	include uwsgi_params;
    	uwsgi_param UWSGI_CHDIR /home/rwang/python/api;
    	uwsgi_param UWSGI_PYHOME /home/rwang/python/api;
    	uwsgi_param UWSGI_SCRIPT api;
    	uwsgi_pass 127.0.0.1:9980;
  		}
	}

#### 分别创建刚才创建的两个配置文件的软链接

	sudo ln -s /etc/uwsgi/apps-available/vhosts.ini /etc/uwsgi/apps-enabled/
	sudo ln -s /etc/nginx/sites-available/siteinpython /etc/nginx/sites-enabled/

#### 修改Nginx的配置文件/etc/nginx/nginx.conf,替换YOURUSERNAME为web server的用户

	user YOURUSERNAME;

#### 重启uWsgi和Nginx服务

	sudo service uwsgi restart
	sudo service nginx restart

#### 使用virtualenv创建一个python app沙盒，名为api

	virtualenv api
	cd api

#### 进入沙盒环境，使用沙盒的原因详见[这里][3],这篇文章还介绍了一些使用python创建网站，特别是Django网站的一些**Best practice**

	source bin/activate

#### 在沙盒环境中安装web.py

	pip install web.py

#### 在api目录下，新建一个python源码文件api.py, 将下面的代码放入并保存

	#!/usr/bin/env python
	import web

	urls = (
    	      '/(.*)', 'index'
    	   )

	class index:
    	def GET(self, name):
      		return "Hello, World"


	app = web.application(urls, globals())
	application = app.wsgifunc()

#### 重启 uWsgi 服务

	sudo service uwsgi restart

## 测试

Visit http://127.0.0.1:9981, the page is now online in local domain

That is it! Enjoy it, and code python with web.py with fun.

## 参考资料

1. [Nginx and uWSGI](http://lars.la/nginx_uwsgi.html)
2. [web.py and virtualenv](http://lars.la/webpy_and_virtualenv.html)

[1]: http://webpy.org/
[2]: http://uwsgi-docs.readthedocs.org/en/latest/index.html
[3]: http://www.jeffknupp.com/blog/2013/12/18/starting-a-django-16-project-the-right-way/
[uwsgi picture1]: http://down.chinaz.com/upload/2011/12/8/201112817335078221.jpg


