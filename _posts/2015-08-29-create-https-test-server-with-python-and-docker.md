---
layout: post
title: "Mac OS 平台使用 Python 和 Docker 创建测试用 Https Server"
description: ""
category: [Flask, Docker]
---

__Flask__ 是我很喜欢的 Python Web Framework，最近需要测试 Https 通信，需要创建一个使用自签名证书的 Https Server，那么用 __Flask__ 可以这样通过下面的步骤非常方便地实现。

## 自签名证书

使用 `OpenSSL` 生成 `.key` 私钥文件和 `.csr` Certificate Signing Request 文件：

	openssl req -new -key test.key -out test.csr

生成 10 年期的自签名证书文件：

	openssl x509 -req -days 3650 -in test.csr -signkey test.key -out test.crt

更多可以参考 [HTTPS 证书链以及 Android 应用中的 HTTPS 实现问题](http://xhrwang.me/2015/06/06/https-and-android.html)。

## Flask 工程

Python 的开发环境搭建就不提了，命令行就行，很简单，安装 __Flask__ 也就一行命令：

	pip install flask

然后，创建一个 Python 源码文件，内容如下：

{% highlight python %}
from OpenSSL import SSL
from flask import Flask
from flask import render_template

app = Flask(__name__)
context = SSL.Context(SSL.SSLv23_METHOD)
context.use_privatekey_file('/Users/rwang/testssl/test.key')
context.use_certificate_file('/Users/rwang/testssl/test.crt')


@app.route("/")
def hello():
	return render_template("index.html")

if __name__ == "__main__":
    app.run(host='127.0.0.1',port=8443, 
        debug = True, ssl_context=context)
{% endhighlight %}

可以看到在 Https 测试站点配置了刚才生成的自签名证书，而且，使用了 Flask 的 `template` 来响应一个 `index.html` 文件中的内容。这需要在上面 Python 源码文件同级目录下创建 `templates` 目录，并创建 `index.html` 文件，里面的内容和测试目的有关，比如跳转 Http 站点的：

{% highlight html %}
<html>
  <head>
    <title>Redirect</title>
    <script>window.location.href='http://www.baidu.com';</script>
  </head>
  <body>
      <h1>Hello, Stranger!</h1>
  </body>
</html>
{% endhighlight %}

到这里，可以在命令行中执行以下上面的 Python 源码，在浏览器中访问 `https://127.0.0.1:8443` 来检查 Https 站点是否正常工作：

	python pyhttpsserver.py

如果正常跳转到百度首页就说明站点工作正常。

## Docker

现在，除了 Boot2Docker，Docker 官方提供了 [Mac OS 平台上安装 Docker](https://docs.docker.com/installation/mac/) 的文件，下载后按照提示安装好就行。

安装后，参考上面的官方文档创建一个 VirtualBox并启动就可以进行 Docker 的日常管理了。这里在存放上面 `pyhttpsserver.py` 文件和 `templates` 目录的文件夹中，创建一个 `Dockerfile` 文件，内容为：

{% highlight text %}
FROM python:2.7.10
RUN pip install Flask
ADD . /code
WORKDIR /code
CMD python pyhttpsserver.py
{% endhighlight %}
 
要使用上面的 `Dockerfile` build 出一个 Docker 镜像，需要先 pull 下来 `python：2.7.10` 基础镜像：

	docker pull python:2.7.10

然后在 `shell` 中 `cd` 到包含 `Dockerfile` 的目录下，运行下面的命令 build 一个包含这个 Https Server 的镜像：

	docker build -t httpsserver .

注意最后面的 `.` 不能少。

有了 `httpsserver` image，就可以在后台运行这个 Https Server 的 `Container` 了：

	docker run -d httpsserver

好了，我们有了一个随时可用的 Https 测试服务器了，如果想用做其他用途，修改 Flask 工程重新编译不同的 Docker image 就好了，嫌麻烦也可以将包含 Flask 工程的目录通过 Volumn 在运行 Docker 容器时挂载到容器中再执行，具体可以参考 [理解 Docker 中的 Volumes](http://xhrwang.me/2014/12/18/understanding-volumes-in-docker.html)。


