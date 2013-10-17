---
layout: post
title: OS杂记[3]
categories:
- Ubuntu
tags:
- Ubuntu
- Checking Battery...

---
昨天我的Ubuntu卡在了启动画面上了。当时就慌了，还以为是KDE出什么问题了。现在总结一下事情的来龙去脉。

在若干天前，我在Ubuntu上安装了一个virtualbox，当时也没有立马在virtualbox里面创建虚拟机，就一直放在了那里。然后有一天，我通过kde自带的Muon software center进行软件升级，当时好像提示升级virtualbox，graphics card及其他的一些软件，然后我就很积极的选择了升级。这次升级肯定是有东西重新配置了我的内核，所以升级完成后就提示我需要重新启动Ubuntu，但是我没有管它，因为的我计算机是一直开着的，有一些定时任务需要执行，所以就一直这样等着了，然后到了昨天，我在Ubuntu里面登陆哪个龙井QQ时候的就突然卡死了，大家都知道龙井QQ是在wine上跑着的，有时候出些问题是必然的，所以我也没有太在意，等了好大一会儿，还是一直卡着，于是我就硬件重启了，主要整个ubuntu桌面都没有反应，也进不去控制台。然后悲剧的事情就这样发生了，Ubuntu在KDE的启动画面上卡住了，重启了好几次都是如此，我还以为是我的kde有问题了呢，因为当时的想法是kde先出现的界面卡死现象，所以自然而然的就想到可能是kde出了问题，然后再次重启后，迅速按CTL+ALT+F1,进入控制台，然后就是按照网上的各种关于kde卡的解决方案进行尝试，但都一一无效。

最后看到网上的一些介绍及自己电脑上的前前后后的现象，想到了在virtualbox更新时和graphics card更新时可能重写了内核的配置，所以导致了dm需要更新，于是按照网上的介绍。


	sudo apt-get install gdm
	//然后可以配置gdm，但是在上面执行安装后，应当会自动进入到configure界面
	sudo dpkg-reconfigure gdm
	//重新选择显示器，会显示gdm,lightdm，选择lightdm

接着重新启动。
