---
layout: post
title:  Windows2008 开启 SSH
categories:
- SSH
tags:
- Windows
- SSH
- SFTP

---

## 问题情境描述
我由一个远程的VMware的虚拟机，该虚拟机具有独立的IP地址，通过本地的vSphere Client进行连接。并且该虚拟机处于一个巨大的局域网内部，而且该虚拟机不能连接到外网。本地的vSphere Client可以ping通该虚拟机，但不是处于相同的局域网段。于是我需要一个服务来支持本地到该虚拟机的文件共享。


## 方案
我在该虚拟机上初始安装时拷贝了一个freeSSHd的软件来支持ssh服务。只需要在该服务器上安装该软件，适当的配置就可以使得该服务器支持ssh，sftp等服务。

## 为什么scp不行
但是我在虚拟机上安装好freeSSHd后，虽然可以ssh到该虚拟机上，但是scp命令却不能连接到该虚拟机。Stack Overflow给出的解释是：

>scp, as many other things, is two-side protocol. It requires scp to be present on both client and server. When you issue copy command, ssh connects to given server and spawns scp process, which your local scp then communicates with. In your case, there is no scp on server, so no communication possible.

参见[freeSSHD cannot be accessed by scp](http://stackoverflow.com/questions/22624881/freesshd-cannot-be-accessed-by-scp) .

## 开启SFTP服务

由于再次通过usb等方式拷贝到虚拟机软件已经不现实了（代价比较高），所以只能放弃scp的使用，转而使用sftp。sftp的开启在freeSSHd上非常简单。

**注意：** 在每次更改freeSSHd时，我的体验是仅仅通过退出并重freeSSHd服务是不可以的。所以，我只好每次重启虚拟机。

## 客户端使用WinScp

WinScp客户端软件内置了支持sftp，所以只要配置下，就可以连接到虚拟机的sftp服务器上了。

