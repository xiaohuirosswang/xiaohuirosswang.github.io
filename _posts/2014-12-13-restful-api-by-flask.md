---
layout: post
title: "基于 Flask 实现 RESTful API"
description: ""
category: [Python, Flask, RESTful]
---

## Why

2000 年，[Roy T. Fielding][1] 完成了自己的博士论文 [Architectural Styles and
the Design of Network-based Software Architectures][2]（[中文版][3]）,提出了 Web 服务的基础设计准则，并推导出了 [REST][4] 的设计模式。这成为了后来 HTTP 1.1 的设计基础并被大规模应用。进入到移动互联网时代后，在移动应用的服务器接口设计中 [RESTful API][5] 设计准则也广受推崇。

上面提到的论文中文版译者李锟在 [理解本真的REST架构风格][12] 中评价道：

-----------

> 在笔者看来，Fielding这篇博士论文在Web发展史上的价值，不亚于Web之父Tim Berners-Lee关于超文本的那篇经典论文。然而遗憾的是，这篇博士论文在诞生之后的将近5年时间里，一直没有得到足够的重视。例如Web Service相关规范SOAP/WSDL的设计者们，显然不大理解REST是什么，HTTP/1.1究竟是一个什么样的协议、为何要设计成这个样子。

> 这种情况在2005年之后有了很大的改善，随着Ajax、Ruby on Rails等新的Web开发技术的兴起，在Web开发技术社区掀起了一场重归Web架构设计本源的运动，REST架构风格得到了越来越多的关注。在2007年1月，支持REST开发的Ruby on Rails 1.2版正式发布，并且将支持REST开发作为Rails未来发展中的优先内容。Ruby on Rails的创始人DHH做了一个名为“World of Resources”的精彩演讲，DHH在Web开发技术社区中的强大影响力，使得REST一下子处在Web开发技术舞台的聚光灯之下。

-----------------

记得曾经在知乎看到过一个问题，大意是问十年后我们的生活可能会是什么样子，有个答案说 `如果想知道十年后我们的生活是什么样子，就去各个研究院看看现在的博士、博士后们都在研究些什么`。Fielding 博士的这篇论文正好可以作为这个答案的证明。

很显然掌握如何设计并实现一个 RESTful API 是非常有价值的。我喜欢 Python，也喜欢 Flask，因为这个 Microframework 提供了非常好的扩展性，使用 Flask 周边的优秀插件可以提供诸如登录加密认证，第三方数据库连接，表单，Markdown支持等功能。而 [Miguel Grinberg][7] 在自己的 Blog 分享了许多使用 Flask 进行 Web 开发的经验，我在阅读 [Designing a RESTful API with Python and Flask][8] 这篇文章的时候收获很多，现在将其翻译过来，同时巩固一下理解。

如果想对 __RESTful API__ 有全面的了解，阅读上面提到的博士论文中的 [第五章][5] 可以提供非常好的参考，阅读全文可以参考这篇非常好的 [导读][6]。 另外 [Whatisest][11] 也是非常好的关于 REST 的网站，不过需要翻墙。

## What

本文后面的 Article 部分是 [Designing a RESTful API with Python and Flask][8] 这篇文章的翻译，原文外的内容会特别注明。

开发环境：

- Ubuntu 14.04.1 64bit LTS
- Python 2.7
- Vitualenv
- Flask


## Article

------------

### [Designing a RESTful API with Python and Flask][8] 翻译

近年以来 [REST][5] （REpresentational State Transfer）在 Web Service 和 Web APIs 领域作为标准架构设计越来越重要。

在这篇文章中，你将看到使用 [Python][9] 和 [Flask][10] microframework 创建一个 RESTful Web Service 是多么简单。

------------

### 什么是 REST？

下面六条准则定义了一个 REST 系统的特征：

- __客户-服务器（Client-Server）__，提供服务的服务器和使用服务的客户需要被隔离对待。
- __无状态（Stateless）__，来自客户的每一个请求必须包含服务器处理该请求所需的所有信息。换句话说，服务器端不能存储来自某个客户的某个请求中的信息，并在该客户的其他请求中使用。
- __可缓存（Cachable）__，服务器必须让客户知道请求是否可以被缓存。（Ross：更详细解释请参考 [理解本真的REST架构风格][12] 以及 [StackOverflow 的这个问题][13] 中对缓存的解释。）
- __分层系统（Layered System）__，服务器和客户之间的通信必须被这样标准化：允许服务器和客户之间的中间层（Ross：代理，网关等）可以代替服务器对客户的请求进行回应，而且这些对客户来说不需要特别支持。
- __统一接口（Uniform Interface）__，客户和服务器之间通信的方法必须是统一化的。（Ross：GET,POST,PUT.DELETE, etc）
- __支持按需代码（Code-On-Demand，可选）__，服务器可以提供一些代码或者脚本（Ross：Javascrpt，flash，etc）并在客户的运行环境中执行。这条准则是这些准则中唯一不必必须满足的一条。（Ross：比如客户可以在客户端下载脚本生成密码访问服务器。）

------------

### 什么是一个 RESTful 的 Web Service？

__REST__ 架构最初被设计出来用于 World Wide Web 使用的 [HTTP 协议][14]。

RESTful Web Service 的核心概念在于对 Resources 的抽象。Resources 被 [URIs][15] (Uniform Resource Identifier) 表征。客户使用 HTTP 协议定义的方法发送请求给这些 URIs，然后相应的资源的状态就可能会发生变化。

HTTP 请求的方法是被专门设计出来以标准的方式影响给定资源的：

----------

<table>
    <tr>
        <td>HTTP Method</td>
        <td>Action</td>
        <td>Example</td>
    </tr>
    <tr>
        <td>Get</td>
        <td>从某种资源获取信息</td>
        <td>http://example.com/api/orders （获取 order list）</td>
    </tr>
    <tr>
        <td>Get</td>
        <td>从某个资源获取信息</td>
        <td>http://example.com/api/orders/123 （获取 order #123）</td>
    </tr>
    <tr>
        <td>POST</td>
        <td>创建一个新资源</td>
        <td>http://example.com/api/orders （根据请求中的数据创建一个新 order）</td>
    </tr>
    <tr>
        <td>PUT</td>
        <td>更新一个资源</td>
        <td>http://example.com/api/orders/123 （根据请求中的数据更新 #order 为 123 的 order）</td>
    </tr>
    <tr>
        <td>DELETE</td>
        <td>删除一个资源</td>
        <td>http://example.com/api/orders/123 （删除 #order 为 123 的 order）</td>
    </tr>
</table>

--------------

REST 设计对请求中的数据格式没有要求，但是一般情况下，数据在请求中是一个 JSON 串，或者 URL 后面跟着的 [Query String][16]

------------

### 设计一个简单的 Web Service

遵循 REST 的准则设计一个 Web Service 或者 API 可以看作识别一种公开资源并定义他们如何被不同的请求方法所改变。

现在假设我们要实现一个 TO-DO List 应用并为其设计一个 Web Service。首先要做的事情就是访问该 Service 的 ROOT URL 是什么。比如，我们可以这样定义：

	http://[hostname]/todo/api/v1.0/

这里我决定将该应用的名称和 API 的版本号包含在 URL 中。在 URL 中包含应用名称可以将该服务于运行在同一服务器上的其他应用区别开；而 URL 中包含 API 版本信息可以让我们在未来版本升级中更方便，因为可能新版本中可能会加入潜在的与旧版本不兼容的函数，这样升级后不会影响到依赖于旧版本的其他应用。

接下来就需要确定该服务计划向外界公开的资源。本文中的例子是一个极其简单的应用，我们只需要用到 Tasks，所以这里的资源也就是 TO-DO list里面的 Tasks。

我们的 Tasks 资源将会以下面的方式被 HTTP 方法所影响：

-----------------

<table>
    <tr>
        <td>HTTP Method</td>
        <td>URI</td>
        <td>Action</td>
    </tr>
    <tr>
        <td>Get</td>
        <td>http://[hostname]/todo/api/v1.0/tasks</td>
        <td>获取所有 Tasks 的列表</td>
    </tr>
    <tr>
        <td>Get</td>
        <td>http://[hostname]/todo/api/v1.0/tasks/[task_id]</td>
        <td>获取给定 Task Id 的任务内容</td>
    </tr>
    <tr>
        <td>POST</td>
        <td>http://[hostname]/todo/api/v1.0/tasks</td>
        <td>创建一个新的 Task</td>
    </tr>
    <tr>
        <td>PUT</td>
        <td>http://[hostname]/todo/api/v1.0/tasks/[task_id]</td>
        <td>根据 Task Id 更新一个已有的 Task</td>
    </tr>
    <tr>
        <td>DELETE</td>
        <td>http://[hostname]/todo/api/v1.0/tasks/[task_id]</td>
        <td>根据 Task Id 删除一个已有的 Task</td>
    </tr>
</table>

-----------------

我们定义一个 Task 包含下面的内容：

- __id__，一个 Task 的唯一识别号，Numeric 类型
- __title__，一个 Task 的名字，也就是简短描述，String 类型
- __description__，一个 Task 的详细描述，Text 类型
- __done__，一个 Task 的完成状态，Boolean 类型

到这里，我们基本上已经完成了这个 Web Serivce 的设计，接下来我们来实现它。

------------

### 对 Flask Microframework 的简短介绍

如果你阅读过 [这篇文章][17]（Ross：也是原作者写的），那你一定已经了解 Flask 是个非常简洁但是很强大的 Python Web 框架。

在我们一头扎进本文例子中的 Web Service 之前，让我们先来回顾一下一个常规的 Flask Web 应用是如何组织的。这里我假定你了解如何在自己的工作环境中进行 Python 开发。下面我们将看到的例子是运行在类 Unix 平台上的，或者说，它们将兼容 Linux，Mac OS X 以及运行 [Cygwin][18] 的 Windows 平台。如果你使用的是 Windows 上的原生 Python 开发环境，有些命令可能会有些许不同。

好了，我们先在一个虚拟环境中安装 Flask。如果你还没有安装 __virtualenv__，可以从[这里下载][19]。

	$ mkdir todo-api
	$ cd todo-api
	$ virtualenv flask
	New python executable in flask/bin/python
	Installing setuptools............................done.
	Installing pip...................done.
	$ flask/bin/pip install flask

OK, Flask 安装好了，我们来创建一个简单的 Web 应用，将下面的内容放入一个名为 app.py 的文件，并保存：

{% highlight python %}
#!flask/bin/python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return "Hello, World!"

if __name__ == '__main__':
    app.run(debug=True)
{% endhighlight %}

接下来，在命令行中敲入下面的命令来运行这个应用：

	$ chmod a+x app.py
	$ ./app.py
	 * Running on http://127.0.0.1:5000/
	 * Restarting with reloader

最后，你可以在自己喜欢的浏览器中敲入 “http://localhost:5000” 来访问这个应用了。

很简单是不是？我们马上就基于这个应用来实现我们的 RESTful Service

------------

### 使用 Python 和 Flask 实现一个 RESTful Service

使用 Flask 创建一个 Web Service 的确难以想象的简单，比创建一个完全服务器端应用（参考原文作者的 [Mega Tutorial][20]）简单多了。

有很多 Flask 的插件可以帮助我们更方便地使用 Flask 实现 RESTful 的 Service，不过由于本文的例子很简单，我们就不讨论这些插件了。（Ross：原文作者的 [MicroBlog][20] 项目使用了很多 Flask 插件实现了一个全功能的个人博客，可以参考）

使用这个 GTD （Get Things Done）Service 的客户会请求服务器添加、删除以及修改一个 Task，很显然我们需要有一个方式来存储这些 Tasks。创建一个小规模的数据库是个办法，但是和数据库（Ross：Sqlite，MongoDB，etc）的通信不是本文的重点，所以我们将使用一个更简单的做法来实现数据存储。关于如何在使用 Flask 的时候使用第三方数据库，请参考 [MicroBlog][20]

我们将使用一个内存中的数组结构来代替数据库保存所有的 Tasks，这适用于当这个 Service 单进程并且单线程工作的情况。Flask 自带的开发用的 Web Server 就满足这种条件。但是如果在使用了其他 Web Server 的生产环境中就不合适了，这时候我们就必须有个数据库来做数据存储了。

在我们之前的 Flask Demo 应用基础上，让我们来实现 TO-DO List 的第一个 Entry Point：

{% highlight python %}
#!flask/bin/python
from flask import Flask, jsonify

app = Flask(__name__)

tasks = [
    {
        'id': 1,
        'title': u'Buy groceries',
        'description': u'Milk, Cheese, Pizza, Fruit, Tylenol', 
        'done': False
    },
    {
        'id': 2,
        'title': u'Learn Python',
        'description': u'Need to find a good Python tutorial on the web', 
        'done': False
    }
]

@app.route('/todo/api/v1.0/tasks', methods=['GET'])
def get_tasks():
    return jsonify({'tasks': tasks})

if __name__ == '__main__':
    app.run(debug=True)
{% endhighlight %}

可以看出，相比那个 Hello World 例子没有多少变化。我们添加了一个内存中的元素为字典的 List 来保存 Tasks，每个字典元素都包含我们之前定义的 Task 的内容。

区别于之前的 __index__ entry point，我们使用了和 __todo/api/v1.0/tasks__ 关联的 __get_tasks__ 函数，只处理 HTTP 的 GET 方法。这个函数的 response 并不是纯文本格式，而是由 Flask 的 __jsonify__ 函数创建的 JSON 格式的数据。

使用浏览器测试一个 Web Service 通常不是最好的办法，因为浏览器无法非常方便地创建各种类型的 HTTP 请求，所以我们使用 [curl][21]，如果你没有 curl，安装一下吧。（Ross：Ubuntu 上使用 “sudo apt-get install curl” 就可以了，你会爱上 curl 的）

和之前一样，我们通过运行 __app.py__ 来启动这个 Service，然后打开一个命令行终端，运行下面的命令：

{% highlight text %}
$ curl -i http://localhost:5000/todo/api/v1.0/tasks
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 294
Server: Werkzeug/0.8.3 Python/2.7.3
Date: Mon, 20 May 2013 04:53:53 GMT

{
  "tasks": [
    {
      "description": "Milk, Cheese, Pizza, Fruit, Tylenol",
      "done": false,
      "id": 1,
      "title": "Buy groceries"
    },
    {
      "description": "Need to find a good Python tutorial on the web",
      "done": false,
      "id": 2,
      "title": "Learn Python"
    }
  ]
}
{% endhighlight %}

就这样，我们访问了刚刚创建的 RESTful Service 并拿到了 Tasks 的数据。

现在，我们来实现第二个相应 GET 请求并返回特定的 Task 的 API，参考上面 API 的定义表格就好：

{% highlight python %}
from flask import abort

@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['GET'])
def get_task(task_id):
    task = filter(lambda t: t['id'] == task_id, tasks)
    if len(task) == 0:
        abort(404)
    return jsonify({'task': task[0]})
{% endhighlight %}

这里就有点意思了，我们从 URL 中获取到了 Task 的 Id 信息，Flask 将其传递给 __get_task__ 函数的 __task_id__ 变量。使用这个变量的值从我们存储的 Tasks List 里查找具有相同 id 的 Task，如果找不到相应的 Task，返回我们熟知的 404 错误，在 HTTP 规范中这表示 “Resource Not Found”，和我们预期的结果是一致的；如果找到了符合条件的 Task，那我们将这个 Task 的内容用 __jsonify__ 封装为 JSON 数据并响应这个请求，就像我们上一个 API 中处理获取所有 Tasks 的请求一样。

如果我们用 curl 访问这个接口时，会看到：

{% highlight text %}
$ curl -i http://localhost:5000/todo/api/v1.0/tasks/2
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 151
Server: Werkzeug/0.8.3 Python/2.7.3
Date: Mon, 20 May 2013 05:21:50 GMT

{
  "task": {
    "description": "Need to find a good Python tutorial on the web",
    "done": false,
    "id": 2,
    "title": "Learn Python"
  }
}
$ curl -i http://localhost:5000/todo/api/v1.0/tasks/3
HTTP/1.0 404 NOT FOUND
Content-Type: text/html
Content-Length: 238
Server: Werkzeug/0.8.3 Python/2.7.3
Date: Mon, 20 May 2013 05:21:52 GMT

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>404 Not Found</title>
<h1>Not Found</h1>
<p>The requested URL was not found on the server.</p><p>If you     entered the URL manually please check your spelling and try again.</p>
{% endhighlight %}

可以看到，当我们尝试获取 Task2 的时候，我们得到了 Task2 的数据；但当我们尝试获取 Task3 的时候，我们得到了 404 错误提示。比较奇怪的是错误提示是 HTML 格式，而不是 JSON 格式，这其实是因为 Flask 自动处理了 404 错误。但是由于这是一个 Web Service，访问它的客户可能期望总是得到相同格式的数据，这里就是 JSON 格式的数据。所以我们需要改进一下我们对 404 错误的处理逻辑：

{% highlight python %}
from flask import make_response

@app.errorhandler(404)
def not_found(error):
    return make_response(jsonify({'error': 'Not found'}), 404)
{% endhighlight %}

这样我们就可以得到 API 友好的错误提示了：

{% highlight text %}
$ curl -i http://localhost:5000/todo/api/v1.0/tasks/3
HTTP/1.0 404 NOT FOUND
Content-Type: application/json
Content-Length: 26
Server: Werkzeug/0.8.3 Python/2.7.3
Date: Mon, 20 May 2013 05:36:54 GMT

{
  "error": "Not found"
}
{% endhighlight %}

接下来，我们将处理 POST 方法，也就是响应客户请求添加一个新的 Task 到我们的 Tasks List：

{% highlight python %}
from flask import request

@app.route('/todo/api/v1.0/tasks', methods=['POST'])
def create_task():
    if not request.json or not 'title' in request.json:
        abort(400)
    task = {
        'id': tasks[-1]['id'] + 1,
        'title': request.json['title'],
        'description': request.json.get('description', ""),
        'done': False
    }
    tasks.append(task)
    return jsonify({'task': task}), 201
{% endhighlight %}

可以看到，添加一个新的 Task 也是很简单的。__request.json__ 变量保存了请求中的的 JSON 格式的数据。如果请求中没有数据，或者数据中没有 __title__ 的内容，我们将会返回一个表示 “Bad Request” 的 400 错误。如果数据合法，我们会新建一个字典元素，这个字典元素表示的 Task 的 id 就是 TO-DO List 中的最后一个字典元素的 id + 1，这是在我们存储 Tasks 数据的 List 数据结果中创建唯一 id 的简单方式。我们允许 __description__ 字段为空，而且我们假定所有新建 Task 的完成状态都是 False。接下来，我们将新创建的字典元素添加到 TO-DO List 的末尾，最后将新建的 Task 的内容和表示 “Created” 的 HTTP 状态码 201 返回给客户作为响应。

让我们在命令行中输入下面的命令来测试这个 API：

{% highlight text %}
$ curl -i -H "Content-Type: application/json" -X POST -d '{"title":"Read a book"}' http://localhost:5000/todo/api/v1.0/tasks
HTTP/1.0 201 Created
Content-Type: application/json
Content-Length: 104
Server: Werkzeug/0.8.3 Python/2.7.3
Date: Mon, 20 May 2013 05:56:21 GMT

{
  "task": {
    "description": "",
    "done": false,
    "id": 3,
    "title": "Read a book"
  }
}
{% endhighlight %}

#### __注意__ 如果你在 Windows 平台上的 Cygwin Bash 环境中运行相应版本的 curl，上面的命令会正常工作。但是如果你在 Windows 的原声 CMD 中运行 Win32 版本的 curl，需要做一些调整，即给请求的数据加上双引号：

	curl -i -H "Content-Type: application/json" -X POST -d "{"""title""":"""Read a book"""}" http://localhost:5000/todo/api/v1.0/tasks

一般情况下在 Windows 平台上，我们需要给请求中的数据加上双引号，数据体中的双引号转义需要加三个双引号，就像上面的例子。

当然，在上面的操作结束后，我们可以获取更新后的 Tasks 列表数据：

{% highlight text %}
$ curl -i http://localhost:5000/todo/api/v1.0/tasks
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 423
Server: Werkzeug/0.8.3 Python/2.7.3
Date: Mon, 20 May 2013 05:57:44 GMT

{
  "tasks": [
    {
      "description": "Milk, Cheese, Pizza, Fruit, Tylenol",
      "done": false,
      "id": 1,
      "title": "Buy groceries"
    },
    {
      "description": "Need to find a good Python tutorial on the web",
      "done": false,
      "id": 2,
      "title": "Learn Python"
    },
    {
      "description": "",
      "done": false,
      "id": 3,
      "title": "Read a book"
    }
  ]
}
{% endhighlight %}

我们的 RESTful Web Service 里面的剩余两个函数的实现如下：

{% highlight python %}
@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['PUT'])
def update_task(task_id):
    task = filter(lambda t: t['id'] == task_id, tasks)
    if len(task) == 0:
        abort(404)
    if not request.json:
        abort(400)
    if 'title' in request.json and type(request.json['title']) != unicode:
        abort(400)
    if 'description' in request.json and type(request.json['description']) is not unicode:
        abort(400)
    if 'done' in request.json and type(request.json['done']) is not bool:
        abort(400)
    task[0]['title'] = request.json.get('title', task[0]['title'])
    task[0]['description'] = request.json.get('description', task[0]['description'])
    task[0]['done'] = request.json.get('done', task[0]['done'])
    return jsonify({'task': task[0]})

@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
    task = filter(lambda t: t['id'] == task_id, tasks)
    if len(task) == 0:
        abort(404)
    tasks.remove(task[0])
    return jsonify({'result': True})
{% endhighlight %}

__delete_task__ 函数我们应该很好理解就不多说了。__update_task__ 中为了避免可能的 bug，我们对输入的数据进行了全面的检查，我们必须在更新 Tasks List 之前保证所有来自客户请求的数据都有正确的格式和内容。

让我们来看看一个更新 Task2 的例子：

{% highlight text %}
$ curl -i -H "Content-Type: application/json" -X PUT -d '{"done":true}' http://localhost:5000/todo/api/v1.0/tasks/2
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 170
Server: Werkzeug/0.8.3 Python/2.7.3
Date: Mon, 20 May 2013 07:10:16 GMT

{
  "task": [
    {
      "description": "Need to find a good Python tutorial on the web",
      "done": true,
      "id": 2,
      "title": "Learn Python"
    }
  ]
}
{% endhighlight %}

------------

### 改进 Web Service 的接口

目前为止的 API 设计的问题在于，客户必须依赖于我们返回的 Task 的 id 来构建请求的 URIs，这样做的确很简单，但是间接要求客户必须清楚地了解 URIs 的构建逻辑，如果我们想在未来对这个逻辑进行修改就比较麻烦，可能会导致和现有客户应用不兼容。

所以我们放弃返回 Task 的 id，而是返回 Task 完整的 URI。为了达到这样的目的，我们需要实现一个小函数来创建一个公开的 Task 的 URI 并返回给客户：

{% highlight python %}
from flask import url_for

def make_public_task(task):
    new_task = {}
    for field in task:
        if field == 'id':
            new_task['uri'] = url_for('get_task', task_id=task['id'], _external=True)
        else:
            new_task[field] = task[field]
    return new_task
{% endhighlight %}

这个函数所做的事情就是从存储 Tasks 的 List 里面拿出一个 Task 的数据，并将其 __id__ 替换为 Flask __url_for__ 模块创建的  __uri__。然后，在我们响应客户请求的时候，就可以先用这个函数处理一下相应的数据：

{% highlight python %}
@app.route('/todo/api/v1.0/tasks', methods=['GET'])
def get_tasks():
    return jsonify({'tasks': map(make_public_task, tasks)})
{% endhighlight %}

这样，客户请求 Tasks 数据看到的内容将会是这样的：

{% highlight text %}
$ curl -i http://localhost:5000/todo/api/v1.0/tasks
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 406
Server: Werkzeug/0.8.3 Python/2.7.3
Date: Mon, 20 May 2013 18:16:28 GMT

{
  "tasks": [
    {
      "title": "Buy groceries",
      "done": false,
      "description": "Milk, Cheese, Pizza, Fruit, Tylenol",
      "uri": "http://localhost:5000/todo/api/v1.0/tasks/1"
    },
    {
      "title": "Learn Python",
      "done": false,
      "description": "Need to find a good Python tutorial on the web",
      "uri": "http://localhost:5000/todo/api/v1.0/tasks/2"
    }
  ]
}
{% endhighlight %}

给之前定义的所有响应客户请求的所有函数都应用这个逻辑就可以让客户总是得到 URLs。

------------

### 提高 RESTful Web Service 的安全性

做完了不是吗？的确，我们完成了这个 Web Service 的所有功能实现。但是有一个问题：我们的 Service 是完全公开的，任何人都可以访问，这通常不是个好事情。

我们创建了一个 TO-DO List 的管理服务并开放给所有人，如果一个不安分的程序员理解了这个服务的逻辑后有心恶搞，那完全可以实现一个新的客户端来捣乱我们的数据。很多入门的教程都忽略了安全性问题并止步于此，但我认为这是个严肃的问题，需要被严肃地对待。

最简单的提高安全性的方式是要求访问我们 Service 的客户端提供一个 username 和 password。一般的 Web 应用程序都有登录机制，然后服务器端会对每个登录成功的用户创建一个 Session 进行后续服务，这个 Session 的 id 会被保存在客户端的 Cookie 里面。不幸的是，这样做就会违反 REST 准则中的 __Stateless__ 准则，所以我们要求访问我们服务的客户端在每个请求中都包含自己的认证信息。

在进行 REST 的设计中我们尽可能地遵循 HTTP 协议的规范，现在既然我们实现认证的机制，那我们当然考虑 HTTP 上下文中的认证机制。HTTP 协议提供了两种形式的认证方式，[Basic][22] 和 [Digest][23]。

Flask 的一个插件可以帮助我们实现认证，这个开源插件就是原文作者实现的 [Flask-HTTPAuth][24]，尽管安装使用吧：

	$ flask/bin/pip install flask-httpauth

如果我们假定这个 Web Service 只允许用户名为 __miguel__ （Ross：原文作者的名字）以密码 __python__ 来认证，我们可以这样实现：

{% highlight python %}
from flask.ext.httpauth import HTTPBasicAuth
auth = HTTPBasicAuth()

@auth.get_password
def get_password(username):
    if username == 'miguel':
        return 'python'
    return None

@auth.error_handler
def unauthorized():
    return make_response(jsonify({'error': 'Unauthorized access'}), 401)
{% endhighlight %}

__get_password__ 函数是一个回调函数，这个插件会使用它来获取指定用户的密码。在更复杂的场景中，这个函数可以进行数据库的操作以及加密等，这里我们就简单实现了。

__error_handler__ 函数也是一个回调函数，当插件需要返回一个未认证的错误给客户端的时候会被调用。就像我们之前处理 404 错误提示一样，我们返回 JSON 格式的提示。

在认证机制实现后，我们接下来要做的就是确定哪些函数的执行需要认证保护，然后添加 __@auth.login_required__ 的装饰器就可以了，举个例子：

{% highlight python %}
@app.route('/todo/api/v1.0/tasks', methods=['GET'])
@auth.login_required
def get_tasks():
    return jsonify({'tasks': tasks})
{% endhighlight %}

如果我们不提供认证信息直接访问上面的 API 就会是这样的：

{% highlight text %}
$ curl -i http://localhost:5000/todo/api/v1.0/tasks
HTTP/1.0 401 UNAUTHORIZED
Content-Type: application/json
Content-Length: 36
WWW-Authenticate: Basic realm="Authentication Required"
Server: Werkzeug/0.8.3 Python/2.7.3
Date: Mon, 20 May 2013 06:41:14 GMT

{
  "error": "Unauthorized access"
}
{% endhighlight %}

如果要正常地得到数据就需要这样：

{% highlight text %}
$ curl -u miguel:python -i http://localhost:5000/todo/api/v1.0/tasks
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 316
Server: Werkzeug/0.8.3 Python/2.7.3
Date: Mon, 20 May 2013 06:46:45 GMT

{
  "tasks": [
    {
      "title": "Buy groceries",
      "done": false,
      "description": "Milk, Cheese, Pizza, Fruit, Tylenol",
      "uri": "http://localhost:5000/todo/api/v1.0/tasks/1"
    },
    {
      "title": "Learn Python",
      "done": false,
      "description": "Need to find a good Python tutorial on the web",
      "uri": "http://localhost:5000/todo/api/v1.0/tasks/2"
    }
  ]
}
{% endhighlight %}

Flask 的这个插件让我们可以自由设置这个服务中的那些函数需要认证信息，那些不需要。为了保证认证信息的安全，我们需要在一个 HTTP Secure 的服务器上部署这个服务 （就是支持 https），这样所有通信数据都会被加密传输，从而避免任何第三方能获取到我们的认证信息。

不幸的是，浏览器一般在得到 401 未认证的响应后非常讨厌地显示一个很丑的登录表单，这在一些后台请求的时候也会发生。所以我们需要实现一个前台登录页面来屏蔽浏览器自己会呈现的登录页面。一个避免浏览器呈现登录页面的小技巧是不要返回 401 错误吗，现在比较流行的是返回表示 “Forbidden” 的 403 错误码。虽然这二者很接近，但是还是违反了 HTTP 协议的标准。所以除非理由充足否则这样做不是一个好的实现。比如如果访问服务的不是一个浏览器那这样做就是一个糟糕的实现。但是如果服务器和客户端是一起实现的，那就简单了，这样做是可以的：

{% highlight python %}
@auth.error_handler
def unauthorized():
    return make_response(jsonify({'error': 'Unauthorized access'}), 403)
{% endhighlight %}

当然，这样修改后客户端应用就需要处理 403 错误了。

------------

### 更多可能的改进

还有很多种方式来改进我们上面实现的 Web Service。

首先，一个真正用于生产环境的服务可能有个数据库作为后盾，使用内存中的 List 存储数据这种方式有很多限制。

其次，让我们的服务支持多用户也是一个改进方向。如果我们的系统支持多用户，那么用户的认证信息可以被用来请求这个用户的 Tasks。在这种情况下，我们需要第二种 Resource，也就是 Users Resource。一个 对 Users 的 POST 请求表示为这个服务注册一个新用户；一个 GET 请求可以返回用户的信息；一个 PUT 请求可以修改一个用户的信息；当然，一个 DELETE 请求可以删除一个已有用户。

对请求 Tasks 数据的 GET 方法的响应可以分得更细。比如这个请求可以包含可选的分页参数，这样客户可以请求一部分 Tasks 的数据。或者为了让这个服务更有价值，我们可以允许用户对请求的数据基于一些条件进行过滤， 比如可以请求所有已经完成的 Tasks 的数据，以及只请求所有Tasks 数据中，Task 的 title 以 A 开头的所有 Tasks的数据。所有这些额外的信息都可以作为参数传递进来做处理。

------------

### 总结

本文中完整的 RESTful API 的代码可以在这里找到：[Gist][25]

我希望文本是一个简单而且友好的针对 RESTful APIs 的介绍。


## 译者的话

原文作者对文本的 Topic 进行了跟进，并提供了一个 [Javascript REST client][26]，并且，原文作者写了另外一篇文章 [使用 Flask-RESTful 插件对本文的 Demo 进行了重构][27]，推荐阅读。


----------------

### 声明：本文 Article 部分的英文版权属于[原作者][7]，本文转载请注明出处。



[1]: http://roy.gbiv.com/
[2]: http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm
[3]: http://www.infoq.com/cn/minibooks/web-based-apps-archit-design
[4]: http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
[5]: http://en.wikipedia.org/wiki/Representational_state_transfer
[6]: http://www.infoq.com/cn/articles/doctor-fielding-article-review
[7]: http://blog.miguelgrinberg.com/
[8]: http://blog.miguelgrinberg.com/post/designing-a-restful-api-with-python-and-flask
[9]: https://www.python.org/
[10]: http://flask.pocoo.org/
[11]: http://whatisrest.com/introduction_to_services/index
[12]: http://www.infoq.com/cn/articles/understanding-restful-style
[13]: http://stackoverflow.com/questions/20988051/what-is-cacheable-communications-protocol
[14]: http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol
[15]: https://en.wikipedia.org/wiki/Uniform_Resource_Identifier
[16]: http://en.wikipedia.org/wiki/Query_string
[17]: http://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world
[18]: http://www.cygwin.com/
[19]: https://pypi.python.org/pypi/virtualenv
[20]: http://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world
[21]: http://curl.haxx.se/
[22]: http://en.wikipedia.org/wiki/Basic_access_authentication
[23]: http://en.wikipedia.org/wiki/Digest_access_authentication
[24]: https://github.com/miguelgrinberg/flask-httpauth
[25]: https://gist.github.com/miguelgrinberg/5614326
[26]: http://blog.miguelgrinberg.com/post/writing-a-javascript-rest-client
[27]: http://blog.miguelgrinberg.com/post/designing-a-restful-api-using-flask-restful


