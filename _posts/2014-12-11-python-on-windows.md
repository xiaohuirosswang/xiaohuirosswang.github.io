---
layout: post
title: "Windows 平台上的 Python 开发"
description: ""
category: [Python]
---

## Why

作为一个程序员，我们经常会需要开发一些自己或者 Team 内部需要的工具，从而提高工作效率或者减少重复的人工工作。比如日志文件的分析和数据提取，系统的日常维护等。当然，这些工作中有些可以用脚本来做，有些可以用产品或服务的编程语言比如 C++, JAVA 或者 C# 来做，但是根据我个人的经验，在 Windows 系统上这样做会有些问题：

- 使用脚本对于稍微复杂一点的工具来说不好调试和维护。
- 使用 C++ 或者 JAVA 有些太重量级了，好像有些小题大做。
- 使用 C# 有 .Net 框架的限制，有时候我们的工具不一定是只给程序员用的，有些可能产品经理和负责商务的同事也会用到，而他们的电脑运行环境就有些复杂，不一定能正常运行。（考虑一下 XP 目前的市场占有率以及各种非官方 Windows 7/8 Ghost 镜像）。等微软正式推出 .Net Native 之后应该会好一些。

所以我们需要考虑的重点在方便部署，环境依赖低以及开发效率高。这样自然就会想到 Python 或者 Ruby 这些动态解释型语言。我喜欢 Python 更多主要是因为：

1. Python 的哲学中很重要的一点是 [There should be one-- and preferably only one --obvious way to do it.][9]。一般情况下做好做对一件事情只有一个最好的方式，没有过多的语法糖。多人协作代码可读性也高。
2. Virtualenv 让开发各种小工具的隔离性很好。
3. Pyinstaller 能让我们将 Python 代码编译封装为一个独立的 .exe 可执行文件，只要 32bit 和 64bit 没有问题，部署就等于拷贝。
4. Python 有很多优秀的 microframework，比如 Web.py 以及 Flask，非常轻量级，又可以根据需要方便地进行 scale，让我们很容易地将结果 Web 化。
5. Python 有非常多高质量的 packages 可供使用，减少了很多造轮子的成本。

说了这么多，概括一下就是：Python 很适合在 Windows 平台进行小工具的开发。顺便吐槽一下 Ruby，在国内安装 pip 后安装包还好，但是不换国内 Gem 源，挂上代理也不一定能稳定访问 Rubygem。

## What

前面解释了一下为什么要用 Python，这篇文章就来谈谈我个人在这样做的过程中的一些经验。主要想说的有下面这些：

- Windows 平台配置 Python 开发环境
- Virtualenv 的使用以及为什么我们还需要 virtualenvwrapper

## How

### Windows 平台配置 Python 开发环境

1. 从 [Python 官网][1] 下载需要的 Python 安装包。建议首选 2.7.x 的正式版本，因为现有很多开源项目以及大量的 Packages 都是 2.7 兼容的。相关学习资料也更容易获得。如果考虑可能分享给别人使用，最好选择 32bit 版本，这样之后使用 pyinstaller 生成的单独可执行文件才会是 32bit 的，才可以在 32bit 的 Windows 平台运行。或者看看 [官方建议][2]
2. [安装 Python pip][3]，pip 是 Python 的包管理器，可以用来安装、升级和移除第三方的 Packages。
3. [安装 PyWin32][4]，PyWin32 是在 Windows 平台编译 Python 需要用到的。
4. [安装 Pyinstaller][5]，Pyinstaller 可以让我们编译封装 Python 程序为独立的可执行文件。

安装完 Python 后，确保将 Python 安装目录以及 “__Python安装目录__\Scripts” 加入系统环境变量。如果在使用 “pip install __some package__” 无法找到相应的 Windows 平台的 Package，那可以看看 [这个地址][5]，里面有非常全的非官方 Windows 平台的 Python Package 移植。

### 使用 virtualenv

__virtualenv__ 是让我们隔离我们的 Python 开发环境并方便迁移的非常好的解决方案。

#### 安装完 pip 后，可以使用下面的方式安装 virtualenv：

	pip install virtualenv

#### 然后打开 CMD，进入自己准备放置 Python 项目的文件夹中运行：

	# 建立项目的 virtualenv 并命名为 virtenv，该操作会在当前目录基于默认 Python 版本（2.7.x）创建一个名为 virtenv 的文件夹，并自动安装 pip。
	virtualenv virtenv

#### 继续执行下面的命令：

	virtenv/scripts/activate

这样就进进入了该项目的 virutalenv，所有该项目的依赖项在使用 “pip install” 命令安装后不会安装到 Python 系统默认版本的库中。如果想退出该 virtualenv，只需要执行：

	deactivate

### 使用 [virtualenvwrapper][6]

有的时候我们可能使用 Git 或者其他 CVS 来管理我们的代码，我们对给每个项目建立 ignore 配置文件太麻烦，这些 virtualenv 还不好重用，每个项目文件夹下面都得有，是否有更好的办法呢？很幸运的是，virtualenvwrapper 的存在就是来解决这个问题的。

__virtualenvwrapper__ 会将所有 virtualenv 集中管理，并且用户可以在多个项目中使用相同的 virtualenv 而无需有重复拷贝。很遗憾从 pip 安装的 virtualenvwrapper 是无法在 Windows 平台工作的，不过 [Python 官方提供了一个办法][7]。

#### 安装完 virtualenvwrapper, 这样使用：

	# 列出所有现有的 virtualenv
	workon
	# 新建一个 virtualenv
	mkvirtualenv virtenv
	# 使用一个已有的 virtualenv
	workon virtenv

更多使用方式可以参考 [官方文档][8]

### 使用 Pyinstaller

将 Python 文件编译打包为独立可执行文件运行下面的命令：

	pyinstaller -F yourPythonFile -i yourIconFile	


[1]: https://www.python.org/downloads/
[2]: https://wiki.python.org/moin/Python2orPython3
[3]: https://pip.pypa.io/en/latest/installing.html
[4]: http://sourceforge.net/projects/pywin32/
[5]: http://www.lfd.uci.edu/~gohlke/pythonlibs/
[6]: https://virtualenvwrapper.readthedocs.org/en/latest/
[7]: https://pypi.python.org/pypi/virtualenvwrapper-win
[8]: https://virtualenvwrapper.readthedocs.org/en/latest/
[9]: https://www.python.org/dev/peps/pep-0020/


