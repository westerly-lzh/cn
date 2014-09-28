---
layout: post
title:  FreeBSD--从普通用户切换到root
categories:
- Unix
tags:
- FreeBSD

---
刚刚从Ubuntu转到FreeBSD，各种操作的不习惯，其中遇到的第一个是FreeBSD下居然不能用su来从普通用户切换到root用户下。

![bsd-default-su](./bsd-imgs/bsd000.png "默认的BSD不支持su到root用户")

当然不是FreeBSD不支持此操作了，而是BSD更严谨的不是默认支持此操作。所以需要小小的设置一下。

![bsd-su-modify](./bsd-imgs/bsd001.png "修改BSD的用户组设置")

只需要在把需要切换的账户（这里是jeff）添加到root得组下即可。

现在支持从普通用户通过su向超级用户切换了

![bsd-su-support](./bsd-imgs/bsd002.png "BSD修改组后支持普通用户su向超级用户切换")

这样的设置可以有选择的准许某些用户可以执行root操作，而其他的则不可以，拥有更多地权限控制。