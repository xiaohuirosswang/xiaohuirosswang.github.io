---
layout: post
title: "解决 Windows 平台上运行 gem install 的时候出现的 SSL Connect error"
description: ""
category: [Ruby]
---

## 问题

最近在 Windows 平台 使用 Gem 安装一个包的时候，遇到下面的错误：

	# travis-lint 是 ruby 实现的用来验证 .travis.yml 文件正确性的。
	gem install travis-lint

> OpenSSL::SSL::SSLError (SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed):
  app/controllers/users_controller.rb:37:in `update'

## 解决方案

在寻找解决方案的时候，在 StackOverflow 上面发现一个[非常好的回答][1]，解决了各个平台上的相同问题。
具体到 Windows 平台，其中一个答案提到了 [Github 上面的一个脚本][2]，下载后直接运行出错，经过调查发现脚本下载的证书文件放在了 C 盘，由于权限的问题未能创建成功，故而导致整个脚本运行失败。我修改了脚本，将其中用到的 Curl 的证书保存到了 ruby 的安装目录：

{% highlight ruby %}
require 'net/http'
 
# create a path to the file "C:\RailsInstaller\cacert.pem"
cacert_file = File.join(%w{d: Ruby193 cacert.pem})
 
Net::HTTP.start("curl.haxx.se") do |http|
  resp = http.get("/ca/cacert.pem")
  if resp.code == "200"
    open(cacert_file, "wb") { |file| file.write(resp.body) }
    puts "\n\nA bundle of certificate authorities has been installed to"
    puts "D:\\Ruby193\\cacert.pem\n"
    puts "* Please set SSL_CERT_FILE in your current command prompt session with:"
    puts "     set SSL_CERT_FILE=D:\\Ruby193\\cacert.pem"
    puts "* To make this a permanent setting, add it to Environment Variables"
    puts "  under Control Panel -> Advanced -> Environment Variables"
  else
    abort "\n\n>>>> A cacert.pem bundle could not be downloaded."
  end
end
{% endhighlight %}

## 吐槽

话说 ruby 对 Windows 支持真是不怎么样，之前安装 ruby 1.9.3 之后，安装专门用于他的 Devtools，init 就是执行不成功，Google 半天还是未能解决，只能使用其提供的 workaround 解决。


[1]: http://stackoverflow.com/questions/4528101/ssl-connect-returned-1-errno-0-state-sslv3-read-server-certificate-b-certificat
[2]: https://gist.github.com/fnichol/867550


