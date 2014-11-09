---
layout: post
title:  FreeBSD--Jail初探
categories:
- Unix
tags:
- Jail

---

##Introduction

随着计算机硬件的发展，很多的电脑（服务器和个人PC）的利用率普遍偏低，虚拟化的出现为充分利用机器提供了一种契机。FreeBSD在4时代引进的jail虽然是是作为一种安全机制开始其在FreeBSD中得生命，但随着计算机软硬件的发展，jail逐步演化成为一种虚拟化解决方案。但正如在[FreeBSD Jail](http://jeff-lee.name/cn/2014/07/FreeBSD-Jail/)中所指出的，Jail是一种```operating system level virtualization```，这与VMWare级别的虚拟化方案是不同的。Jail的虚拟化方案更显得轻盈，但由于没有虚拟出完整的硬件平台，其应用可能也比较受限。

##Preparation

如果需要在FreeBSD中使用Jail进行虚拟化，需要考虑下面一些基本的问题，需要多少虚拟化系统，需要部署的应用适不适合在虚拟化系统上运行，宿主机器在性能上能否有效支撑这些系统，每个虚拟化系统将会有多大，将虚拟化系统安装在哪些目录比较合适等基本问题。考虑好这些问题后，就可以创建虚拟化目录了。下面使用一个简单地test例子来演示jail的过程及其一些基本配置：
	
	mkdir -p /usr/jails/test  	##首先创建该test虚拟机的目录
	cd /usr/src   				## 切换到src目录，需要确保src目录下有构建系统的源码，如果没有需要从安装光盘中把这些源码文件安装进宿主机器。
	make buildworld				## 构建系统，需要较长时间。
	make installworld DESTDIR=/usr/jails/test	##把构建好的系统安装到指定的虚拟器位置
	make distribution DESTDIR=/usr/jails/test  ## 把宿主机器中得配置文件配置到虚拟机器上
	mount -t devfs devfs /usr/jails/test/dev ## 不是必须得，每个系统都需要一个device去读写。
	
到此，一个基本的Jail系统就完成了。

##Jail Commands
FreeBSD提供了一下三个命令来进行Jail的管理。

* jail ：创建，修改，移除已经存在的jail虚拟机
* jls：列出宿主机器中当前已有的jail虚拟机
* jexec：在指定的虚拟机中执行命令


##Basic Configuration
虽然创建了jail虚拟机，但是还需要一定的配置才能让虚拟机正常运行，需要在宿主机器的/etc/rc.conf中配置：
	
	jail_enable="YES"  ## 允许jail虚拟机在宿主系统启动时一并启动
	jail_list="test" ## 当有多个虚拟机时，使用空格分割
	
对于jail_list中得每个虚拟机需要配置：
	
	jail_test_rootdir="/usr/jails/test" ## 配置test虚拟机的根目录
	jail_test_hostname="www.test.org"	## 配置tset虚拟机的hostname
	jail_test_ip = "10.211.55.6" 		## 配置test虚拟机的ip
	jail_test_devfs_enable="YES"		## 配置test虚拟机可以挂接devfs
	ifconfig_em0_alias0="inet 10.211.55.6 netmask 255.255.255.0" ##配置虚拟机ip到宿主机器的网卡上，需要注意的时在实际应用时，把em0替换成具体的网卡，可以通过ifconfig查看。

对于需要启动sshd服务的虚拟机，需要在该虚拟机的/etc/rc.conf（一般为空文件）配置文件中增加sshd配置：
	
	sshd_enable="YES"

然后在命令行执行

	jexec 1 sh /etc/rc   		##这里假设该虚拟机的编号为1，可以通过jls查看虚拟机编号。
	
配置好sshd时，通过ssh仍是不能登陆进去。还需要给该虚拟机设置root用户的密码：
	
	#先增加一个普通用户
	jexec 1 pw user add lee
	jexec 1 passwd lee
	jexec 1 pw groupmod wheel -m lee
	#用普通用户登陆
	jexec 1 su lee
	$su root
	$passwd root

现在就可以使用ssh登陆该jail虚拟机了。


## Application
基本的jail虚拟机配置完成后，就可以安装应用了。可以通过把宿主机器的ports挂接在jail虚拟机下进行应用安装。在宿主机器下执行：

	mount_nullfs /usr/ports /usr/jails/test/usr/ports
	
如果想再jail启动时就挂在ports系统，那么可以把这一行加入到宿主机器的/etc/rc.conf文件中。


## Miscellaneous

当jail虚拟机的某些功能不能使用时，可以通过从宿主机器的系统copy响应功能的配置文件到jail虚拟机中，并做相应地修改，就可以了。

