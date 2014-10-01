---
layout: post
title:  FreeBSD--内核定制，以IPFilter防火墙为例
categories:
- Unix
tags:
- Kernel
- IPFilter

---

本文中所使用的FreeBSD版本为FreeBSD 10.0-RELEASE

本文以简单介绍FreeBSD中的防火墙IPFIlter为起因，完成FreeBSD内核的定制化方法及其遇到的一些问题。

首先需要先了解下当前的系统状况

![FreeBSD 内核版本及构建时间](/media/img/bsd-imgs/bsd007.png "FreeBSD 内核版本及构建时间")

下面需要为定制内核做准备。

![FreeBSD 内核定制准备](/media/img/bsd-imgs/bsd008.png "FreeBSD 内核定制准备")

下图说明在使用IPFilter（IPF）防火墙之前，网络是好的。

![FreeBSD 使用IPF之前的网络](/media/img/bsd-imgs/bsd009.png "FreeBSD 使用IPF之前的网络")

下面将会编辑/root/kernels/KERNEL_IPF文件，修改该文件中ident为“KERNEL_IPF”，并增加IPF选项：

![FreeBSD IPF在内核中的选项](/media/img/bsd-imgs/bsd010.png "FreeBSD IPF在内核中的选项")

下面就可以构建并安装内核了。内核会被安装到/boot/kernel/目录下，/boot/kernel/目录下原来的内核会被移动到/boot/kernel.old/目录下.构建安装内核的命令很简单：

	make buildkernel KERNCONF=KERNEL_IPF 
	make installkernel KERNCONF=KERNEL_IPF

这样后/boot目录下就会kernel子目录和kernel.old子目录了。

最后重新启动系统，就可以看见现在已经在新内核下了：

![FreeBSD 新内核及IPF](/media/img/bsd-imgs/bsd011.png "FreeBSD 新内核及IPF")

下面简单的配置下IPF，使系统可以使用Loopback Interface。创建/etc/ips.rules文件，并添加如下内容：

	pass in quick on lo0 all
	pass out quick on lo0 all
	
并在终端输入如下命令：
	
	ipf -Fa -f /etc/ips.rules

这样在终端就可以ping通127.0.0.1了。


再回到定制化内核。内核定制需要关注几个文件:
	
	/usr/src/sys/amd64/conf/NOTES
	/usr/src/sys/conf/NOTE

这两个文件详细的说明了定制内核时，需要注意的内容。在定制完内核后，从启系统，在下面画面中选择 【3】Escape to loader prompt ：

![FreeBSD 启动画面](/media/img/bsd-imgs/bsd012.png "FreeBSD 启动画面")

然后就可以进入loder交互界面，在该界面下，可以选择启动个内核，可以选择如何启动。首先可以使用lsmod命令看一下当前引导的内核是哪一个：

![FreeBSD loader界面](/media/img/bsd-imgs/bsd013.png "FreeBSD loader界面")

然后可以使用load命令来加载内核，在最开始时，有把一个内核备份到/root/kernels/kernel_ipf/目录下，而且在/boot/kernel.old/目录下也有一个系统自动备份的内核。所以可以使用其中的任何一个：

	###下面的命令使用任何一个都可以：
	load /root/kernels/kernel_ipf/kernel
	load /boot/kernel.old/kernel
	
这样就会把内核加载起来。最后，通过boot命令就可以使用指定加载的内核完成系统启动了。

在了解系统启动过程时，需要关注/boot/defaults/loader.conf这个文件，这里面记录了系统启动的一些参数。如果需要修改参数系统启动参数，可以通过把需要修改的参数添加到/boot/loader.conf这个文件来完成。

