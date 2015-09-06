---
layout: post
title:  google被封，网页加载慢
categories:
- it
tags:
- fonts
- google
- googleapis

---

自从gongle被封后，网站加载网页特别的慢，尤其是国外的网站，在浏览器的左下角，可以看到“正在传输来自fonts.googleapis.com”的提示。究其原因是因为这些网站中可能使用了google字体fonts.googleapis.com

下面如果使用提供一种火狐浏览器端的设置方式，可以秒开那些加载有google提供api的站点。只要安装个Adblock Plus的火狐插件，然后自定义过滤规则，添加进fonts.googleapis.com这个站点即可。
![fonts.googleapis.com](/media/img/googleapis.png "adblock plus")
