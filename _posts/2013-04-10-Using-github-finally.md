---
layout: post
title: 最终还是用上了Github
categories:
- IT
tags:
- github

---

折腾了好久总算在github上建起了站点，不过我这个站点建的有点厚颜无耻，照搬的[yihui](http://yihui.name)的站点模板，而且在遇到不明白的地方时还忝者脸皮发邮件去问他。在这个过程中遇到了一些问题，现在就分享下。

首先是关于项目主页的问题，我看到[yihui](http://yihui.name)站点上有http://yihui.name/cn,http://yihui.name/en就比较奇怪了，在github的repository中，他的主站项目yihui.github.com(yihui.name指向的地址)和en，cn是同一级别的，按照正常的web网站的结构，yihui.name/cn和yihui.name/en是有问题的，关于这个问题我就请教了[yihui](http://yihui.name)本人，经他指点，在网上找到了[Jiang Xin](http://www.worldhello.net/gotgithub/index.html)的关于github的大作，其中就有在github上如何建站的介绍。

其次关于ubuntu12.04上安装python3++的问题。我的体验还是不要安装的好，当然我是由于安装好后，执行了下面两行操作

{% highlight python %}
sudo rm /usr/bin/python
sudo ln -s /usr/bin/python3.2 /usr/bin/python
{% endhighlight %}

不得不说在ubuntu中有很多的工具是依赖python2.7的，而我上面做的事情就是八python2.7的链接干掉然后链接到python3.2上，后边就是当我使用一些工具命令时，就开始出问题了。比如我
{% highlight python %}
sudo add-apt-repository ppa:xxx
{% endhighlight %}

再次就是关于markdown的问题，我的想法是，既然有一些工具可以实现markdown的编辑时实时预览的功能，那我就何必再安装个jekyll呢，可现实问题是按照网上关于markdown的语法，我使用retext居然不对其中的一些起作用，看来我只有有空再把jekyll搞上了。

在这个web2.0的时代，看来我是要好好的学写html5和js的东西了，一前也想着要好好学下，但是总是没有坚持下来，这样照搬[yihui](http://yihui.name)的站点，我心里实在过意不去，但如此的雷同，确实不好，以后再慢慢的修改下吧。

最后还是不得不抱怨下国内校园网的不给力，现在越来越感到因为网速问题，严重影响工作效率。



