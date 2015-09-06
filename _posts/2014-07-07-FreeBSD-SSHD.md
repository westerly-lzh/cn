---
layout: post
title:  FreeBSD--开启SSHD
categories:
- Unix
tags:
- SSHD

---

FreeBSD默认是不开启sshd的，即FreeBSD默认不能通过ssh进行远程登录。下面就是配置如何使得FreeBSD可以通过远程进行登录，包括普通用户的登录和root用户的登录。

![FreeBSD 默认SSHD不开启](/media/img/bsd-imgs/bsd003.png "FreeBSD 默认SSHD不开启")

下面是如何开启普通用户的远程登录权限。

首先需要修改/etc/inetd.conf，如果需要支持IPv4，那么去掉红线上ssh那一行的“#”，

![FreeBSD inetd.conf](/media/img/bsd-imgs/bsd004.png "FreeBSD inetd.conf")

如果需要sshd支持IPv6，那么就把途中红线下的那行“#”也去掉，然后编辑/etc/rc.conf，增加一行
	
![FreeBSD rc.conf](/media/img/bsd-imgs/bsd005.png "FreeBSD rc.conf")
	
最后，启动sshd服务：
	
	/etc/rc.d/sshd start   ###启动服务
	/etc/rc.d/sshd restart ###重启服务
	
这样就可以使用FreeBSD的ssh服务来远程登录了。

![FreeBSD sshd start](/media/img/bsd-imgs/bsd006.png "FreeBSD sshd start")

这样就可以使用普通用户登录了，如果希望使用root，那么可以在普通用户下使用su命令切换到root用户下。但是这样的配置是不允许直接使用root账户登录的。可以通过修改/etc/ssh/sshd_config文件来开启root用户直接登录，

	#PermitRootLogin no

取消/etc/ssh/sshd_config中上面一行中的“#”注释，并把no改为yes，保存后重启sshd服务即可。