---
layout: post
title: "编译部署 OpenWrt X86 VMWare 虚拟机"
description: ""
category: [Openwrt,Linux,Cross compile]
---

# HowTo: 编译X86架构的OpenWrt VMWare virtual disk


### OpenWrt介绍

[OpenWrt][1]是可用于嵌入式设备的一个Linux开源发行版，主要用作智能路由器的操作系统，开发人员可以非常方便地基于自己的业务需求对该系统进行深度定制，从而提供离线下载，代理设置，[Captive portal][7]等功能，在此基础上，将智能路由器打造成局域网智能中心，如果硬件配置足够，甚至可以在上面部署内部网站（比如PHP and uhttp web server），最近国内厂商基于占领用户客厅，打造家庭智能云而推出的各种智能路由器有很多就基于OpenWrt，比如[极路由][2]，[小米路由器][3]等。

在官方和广大开发者的贡献下，OpenWrt支持非常多的路由器型号，如果你手边有个路由器，[查一下看是否支持][4]。

### 为什么要编译OpenWrt x86的VMWare虚拟硬盘？
如果手边有OpenWrt支持的型号，就可以使用git或者svn同步OpenWrt的源码，设置好image所需的packages后，编译，然后通过路由器官方提供的固件升级途径进行刷机就可以了，如果你的路由器官方不支持第三方固件，就需要自己编译[uboot镜像][5]，打开路由器盒子，引出调试口的线，使用[tftp][6]进行刷机，风险比较大。在熟悉OpenWrt之前不想拿自己的路由器做小白鼠或者需要开发测试OpenWrt上的应用时，本地虚拟机是个不错的选择，而且VMWare虚拟机也能很方便地拓展硬盘，内存以及虚拟网卡，对业务拓展的适应性较好。

### 编译OpenWrt X86的VMWare虚拟硬盘格式文件

##### 1. 参考官方wiki进行[编译环境的搭建][8]

##### 2. 同步OpenWrt的代码
	git clone git://git.openwrt.org/openwrt.git

##### 3. 进入openwrt目录
	cd trunk/

##### 4. 更新并安装编译依赖的包
	scripts/feeds update -a
	scripts/feeds install –a

##### 5. 检查编译环境是否就绪
	make defconfig

##### 6. 配置openwrt的image，这里可以参考[这篇文章][9]。到这里可以看到，整个的流程和编译Linux内核非常像，只是进行menuconfig的时候需要进行更多的packages的选择和自定义。
	make menuconfig

##### 7. 开始编译，为了看到可能的错误详细信息，建议使用V=99参数，成功后就可以在看到文件 trunk/bin/x86/openwrt-x86-generic-combined-ext4.vmdk
	make

##### 8. 在VMWare中创建基于Linux2.6.x的x86虚拟机，然后选择从刚才编译的vmdk文件作为启动硬盘。

##### 9. 在启动前，进行下面的设置：从VMWare的工具栏中，点击“编辑” -> “虚拟网络编辑器”，将VMnet1修改为仅主机模式，子网IP修改为：
	192.168.1.0

##### 10. 给刚才创建的虚拟机添加一个虚拟网卡，默认网卡设置为刚才修改的VMnet1，新建网卡设置为VMnet0桥接模式。这样，OpenWrt的虚拟机可以获取和host同一网段的IP，也可以通过自己的DNS给host分配一个IP，用于测试，当其他虚拟机使用VMnet1时，其得到的IP也是OpenWrt分配的。

##### 11. 在host上注意修改VMnet1对应的虚拟网卡，设置其DNS服务器为192.168.1.1，我们之后会将OpenWrt的eth1设置为该IP。

##### 12. 启动虚拟机，进入系统后，修改root的密码，由于OpenWrt已经内置了SSH， 我们就可以通过SecureCRT之类的PC端软件或者Linux平台的SSH命令进行远程连接终端了。

##### 13. 修改/etc/config/network文件为下面的内容
	
	config interface 'loopback'
        option ifname 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

	config interface 'lan'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.1.1'
        option netmask '255.255.255.0'
        option ip6assign '60'

	config interface 'wan'
        option ifname 'eth1'
        option proto 'dhcp


#### Done！Have fun!

[1]: https://openwrt.org/
[2]: https://code.google.com/p/openwrt-hiwifi/
[3]: http://www.zhihu.com/question/22339109
[4]: http://wiki.openwrt.org/toh/start
[5]: http://wiki.openwrt.org/?do=search&id=uboot
[6]: http://wiki.openwrt.org/?do=search&id=tftp
[7]: http://wiki.openwrt.org/doc/howto/wireless.overview
[8]: http://wiki.openwrt.org/doc/howto/buildroot.exigence
[9]: http://blog.csdn.net/openme_openwrt/article/details/7998389

