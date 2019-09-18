---
layout: post
title: "Docker 使用中的一些最佳实践"
description: ""
category: [Docker, Linux]
---

#### 本文将记录：

- 在使用 Docker 的过程中整理出来的最佳实践。
- 其他 Docker 使用者发布的（可能会是翻译过来的，全部附带原始链接）经过验证的最佳实践。

#### 所以，本文将长期更新。

## Docker 安装相关

如果安装完成后使用 docker 时出现下面的错误：

	> Cannot connect to the Docker daemon. Is 'docker -d' running on this host?

参考[这里的内容][1]，我们需要执行下面的命令来安装 apparmor：

	sudo apt-get install apparmor


## Docker 使用相关

- 使用 Bash Aliases 简化 Docker 使用命令，灵感来自于[一篇很好的文章][2]（发布这篇文章的站点 docker.cn 也是很好的 docker 资源站），我做了扩充：

将下面的内容添加到 $HOME/.bash_aliases 中：

{% highlight bash %}
# Show all available docker related aliases.
alias dockeraliases='printf "\nBelow are all pre-defined docker maintainance aliases:\n\ndkka | dockerkillall\tKill all running containers\ndksa | dockerstopall\tStop all running containers\ndkrc | dockerremovec\tRemove all stopped containers\ndkri | dockerremovei\tRemove all untagged images\ndkra | dockerremovea\tRemove all stopped containers and untagged images\n\n"'
alias dkas='printf "\nBelow are all pre-defined docker maintainance aliases:\n\ndkka | dockerkillall\tKill all running containers\ndksa | dockerstopall\tStop all running containers\ndkrc | dockerremovec\tRemove all stopped containers\ndkri | dockerremovei\tRemove all untagged images\ndkra | dockerremovea\tRemove all stopped containers and untagged images\n\n"'
 
# Kill all running containers.
alias dockerkillall='printf "\n>>> Killing all running containers\n\n" && docker kill $(docker ps -q) && printf "\n"'
alias dkka='printf "\n>>> Killing all running containers\n\n" && docker kill $(docker ps -q) && printf "\n"'
 
# Stop all running containers.
alias dockerstopall='printf "\n>>> Stopping all running containers\n\n" && docker stop $(docker ps -q) && printf "\n"'
alias dksa='printf "\n>>> Stopping all running containers\n\n" && docker stop $(docker ps -q) && printf "\n"'
 
# Remove all stopped containers.
alias dockerremovec='printf "\n>>> Removing stopped containers\n\n" && docker rm $(docker ps -a -q) && printf "\n"'
alias dkrc='printf "\n>>> Removing stopped containers\n\n" && docker rm $(docker ps -a -q) && printf "\n"'
 
# Remove all untagged images.
alias dockerremovei='printf "\n>>> Removing untagged images\n\n" && docker rmi $(docker images -q -f dangling=true) && printf "\n"'
alias dkri='printf "\n>>> Removing untagged images\n\n" && docker rmi $(docker images -q -f dangling=true) && printf "\n"'
 
# Remove all stopped containers and untagged images.
alias dockerremovea='dockerremovec || true && dockerremovei'
alias dkra='dockerremovec || true && dockerremovei'
{% endhighlight %} 

然后，在命令行中执行 “source ~/.bash_aliases” 就可以使用这些别名了，比如：

{% highlight bash %}
$ dkas

Below are all pre-defined docker maintainance aliases:

dkka | dockerkillall    Kill all running containers
dksa | dockerstopall    Stop all running containers
dkrc | dockerremovec    Remove all stopped containers
dkri | dockerremovei    Remove all untagged images
dkra | dockerremovea    Remove all stopped containers and untagged images

{% endhighlight %} 
	

[1]: https://www.digitalocean.com/community/questions/start-docker-error-message
[2]: https://docker.cn/p/docker-15-tips/


